---
title:  "go_channel底层实现"
date:   2024-03-26 17:21:26 +0800
categories: golang
---

# 关键字
```txt
唤醒、原子指令CAS、
```

# 结构
```golang
type hchan struct {
	qcount   uint           // total data in the queue(列队中元素的数量)
	dataqsiz uint           // size of the circular queue(循环列队的长度)
	buf      unsafe.Pointer // points to an array of dataqsiz /elements(指向循环列队存储数组的指针)
	elemsize uint16 //存储元素的类型大小
	closed   uint32//通道是否关闭
	elemtype *_type // element type (元素的类型)
	sendx    uint   // send index 
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters //等待发送的协程列队(底层双向链表,节点结构为sudog)
	sendq    waitq  // list of send waiters //等待接收的协程列队

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex //互斥锁
}
```

```golang
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	// success indicates whether communication over channel c
	// succeeded. It is true if the goroutine was awoken because a
	// value was delivered over channel c, and false if awoken
	// because c was closed.
	success bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

# 基本原则
1. channel中的数据遵循先入先出的原则(先到先接收&先到先发送)
2. channel底层会通过加锁控制并发写、读

# 常见场景及操作
通道内部维护了三个列队
1. 接收数据的协程列队
2. 发送数据的协程列队
3. 数据缓冲列队
![](/assets/img/channel.png)


对于非零值通道
1. 接收操作在数据缓冲列队和发送数据的协程列队不为空的情况下正常；
2. 发送操作在数据缓冲列队未满的情况下正常。
3. 关闭操作正常。

## 发送时调度时机
1. 发送数据，recvq存在goroutine，设置处理器为runnext属性，下次调度器会执行recvq的接收goroutinue.
2. 向channel发送数据时阻塞，发送数据的goroutine会将自己放入sendq，并调用runtime.goparkunlock触发调度让出处理器的使用权(因为已经加入sendq并阻塞，干不了啥)

注:接收操作中的调度时机也类似。

# 参考文档
1. [通道](https://gfw.go101.org/article/channel.html)
2. [Channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/)