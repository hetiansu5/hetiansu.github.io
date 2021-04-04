---
layout: post
title: Go之WaitGroup底层实现
date: 2021-04-05
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: WaitGroup Go
---

# WaitGroup
WaitGroup用于等待一组线程的结束，父线程调用Add来增加等待的线程数，被等待的线程在结束后调用Done来将等待线程数减1，父线程通过调用Wait阻塞等待所有结束（计数器清零）后进行唤醒。

## 源码位置

WaitGroup的源码在SDK包的路径为`src/sync/waitgroup.go`。

## 数据结构
``` golang
type WaitGroup struct {
	noCopy noCopy
	state1 [3]uint32
}
```

1.noCopy noCopy

noCopy这个主要用来限制不能进行copy，这里是为了避免copy后的waitGroup并发使用后，可能会与原waitGroup出现异常而panic。

2.state1 [3]unit32

数组的三个元素（非顺序）：

- couter 通过Add()设置的子goroutine的数量，即被等待线程计数
- waiter 通过Wait()陷入阻塞的等待者计数
- semap  信号量，用于唤醒阻塞waiter

这里需要注意一下couter、waiter、semap并不是顺序存储的，64bit操作系统的原子操作需要保证64bit的内存对齐，在设计上我们需要保证couter和waiter的操作原子性。如果数组的首元素地址能被8整除，则counter和waiter刚好可以在同一块原子操作的64bit内存上，所以取数组前两个元素分别表示couter和waiter；如果不能被8整除（根据内存对齐的原理，地址必然是4的倍数），则取数组后两个。

``` golang
// 根据内存对齐方式的不同，返回statep(couter占用高32bit和waiter占用低32bit)和semap的地址
func (wg *WaitGroup) state() (statep *uint64, semap *uint32) {
	if uintptr(unsafe.Pointer(&wg.state1))%8 == 0 {
		return (*uint64)(unsafe.Pointer(&wg.state1)), &wg.state1[2]
	} else {
		return (*uint64)(unsafe.Pointer(&wg.state1[1])), &wg.state1[0]
	}
}
```

![state1内存对齐](/assets/img/alignment.png)

## 公共方法

```golang
func (wg *WaitGroup) Add(delta int) //增加waitGroup子goruntine计数值
func (wg *WaitGroup) Done() //当子goruntine完成后，将计数器-1
func (wg *WaitGroup) Wait() //调用此方法的goruntine，阻塞等待计数值为0
```

以下方法去除了race竞争检查的源代码。

### Add
操作counter计数值加减。

- 当counter增加时，直接return
- 当counter减少时, 判断条件：counter > 0 || waiter == 0
	- true时，直接return
	- false（等待线程都完成且有等待者）时，statep复位为0，通过semap信号量唤醒所有等待者

```golang
func (wg *WaitGroup) Add(delta int) {
	//从数组中拿到stetep（counter+waiter的组合）和semap信号量的内存地址
	statep, semap := wg.state()
	//stetep原子加操作，高位32bit是counter,实际counter+1
	state := atomic.AddUint64(statep, uint64(delta)<<32)
	//state的高位32bit，表示couter的计数值
	v := int32(state >> 32)
	//state的低位32bit，表示waiter的等待者数量
	w := uint32(state)
	// couter不能小于0
	if v < 0 {
		panic("sync: negative WaitGroup counter")
	}
	// 需要避免错误操作：Add和Wait并发操作，否则会panic
	if w != 0 && delta > 0 && v == int32(delta) {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// 如果还有等待线程未完成或者并没有等待者，直接return
	if v > 0 || w == 0 {
		return
	}
	// 需要避免错误操作：Add和Wait并发操作，否则会panic
	if *statep != state {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// 将statep复位为0（ counter和waiter都置为0）
	*statep = 0
	// 有多少个等待者就往semap循环发信号量（其实就是semap+1），Wait等待有一个调用	// runtime_Semacquire(semap)就是在等待这个信号量
	for ; w != 0; w-- {
		runtime_Semrelease(semap, false, 0)
	}
}
```

### Done
被等待线程完成后调用Done，将counter计数-1，表示线程结束

``` golang
func (wg *WaitGroup) Done() {
	wg.Add(-1)
}
```


### Wait
主线程循环对waiter原子操作+1直到成功后，然后阻塞等待semap信号量而被唤醒，最后return

```golang
func (wg *WaitGroup) Wait() {
	// 从数组中拿到stetep（counter+waiter的组合）和semap信号量的内存地址
	statep, semap := wg.state()
	for {
		//从内存总线中加载最新的statep值
		state := atomic.LoadUint64(statep)
		//state的高位32bit，表示couter的计数值
		v := int32(state >> 32)
		//state的低位32bit，表示waiter的等待者数量
		w := uint32(state)
		//如果couter为0，表示当前已经没有在运行的等待线程了
		if v == 0 {
			return
		}
		// CAS操作statep+1，低位属于waiter,即waiter+1
		if atomic.CompareAndSwapUint64(statep, state, state+1) {
			// CAS操作成功后，阻塞等待semap信号为非零，竞争到会将semap-1，并唤醒线程
			runtime_Semacquire(semap)
			if *statep != 0 {
				panic("sync: WaitGroup is reused before previous Wait has returned")
			}
			return
		}
		// CAS操作失败了，重新进入循环
	}
}
```
