---
title:  "go_channel底层实现"
date:   2024-03-26 17:21:26 +0800
categories: golang
---

# 结构
```golang
type hchan struct {
	qcount   uint           // total data in the queue(列队中的数量)
	dataqsiz uint           // size of the circular queue(循环列队的大小)
	buf      unsafe.Pointer // points to an array of dataqsiz /elements(指向循环列队存储数组的指针)
	elemsize uint16 //存储元素的类型大小
	closed   uint32//通道是否关闭
	elemtype *_type // element type (元素的类型)
	sendx    uint   // send index 
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters //等待发送的协程列队
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

# 参考文档
1. [通道](https://gfw.go101.org/article/channel.html)