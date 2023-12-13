---
title:  "benchmark基准测试"
date:   2023-12-13 19:28:00 +0800
categories: golang
---
# 目的
如何使用golang内置的基准测试实现以下功能？
1. 测试函数的执行耗时情况
2. 测试函数的执行内存分配情况

# 1、简介
benchmark基准测试在测试文件中是以Benchmark开头的函数
```golang
    func BenchmarkXxx(t *testing.B) {
        for n:=1;n<b.N;n++ {
            //待测试函数
        }
    }

    //示例
    func BenchmarkSum(b *testing.B) {
    	for n := 0; n < b.N; n++ {
		    Sum()
	    }   
    }

    func Sum() {
	    sum := 1
	    for sum < 1000 {
		    sum++
	    }
    }
```
b.N是在默认1s内函数执行的次数，待测试函数总耗时小于1s时，b.N会增加执行次数，直到执行耗时超过1s.

# 2、常用option
## 2.1、使用正则匹配需要执行的基准测试
```bash
    -bench='Sum$'
    # 执行以Sum结尾的基准测试函数
    go test -bench='Sum$' ./...
```
```bash
BenchmarkSum
BenchmarkSum-12          4926554               237.1 ns/op             0 B/op          0 allocs/op
PASS
```
## 2.2、指定cpu的数量，输出在指定cpu数量下的基准测试结果
```bash
    -cpu=1,2,4

    go test -v -bench=. -cpu=1,2,4 ./...
```
```bash
BenchmarkSum
BenchmarkSum             5000994               237.3 ns/op
BenchmarkSum-2           4977946               238.3 ns/op
BenchmarkSum-4           5008383               237.0 ns/op
PASS
```
## 2.3、指定总执行耗时，超过后终止基准测试
### 2.3.1、指定总耗时
```bash
    # 指定总执行耗时为5s
    -benchtime=5s

    go test -v -bench=. -benchtime=5s ./testandbench/...
```
```bash
BenchmarkSum
BenchmarkSum-12         24894925               237.9 ns/op
PASS
```

### 2.3.2、指定次数
```bash
    # 总共执行50次
    -benchtime=50x

    go test -v -bench=. -benchtime=50x ./testandbench/...
```
```bash
BenchmarkSum
BenchmarkSum-12               50               242.6 ns/op
PASS
```

## 2.4、查看内存分配情况
```bash
    -benchmem

    go test -v -bench=. -benchmem ./testandbench/...
```
```bash
BenchmarkSum
BenchmarkSum-12          4985266(总执行次数)               238.5 ns/op(单次执行耗时)             0 B/op(每次执行申请的内存大小)          0 allocs/op(单次执行申请内存的次数)
```


# 3、如何排除程序初始化对基准测试的影响
## 3.1、使用b.ResetTimer()重置计时器
```golang
func BenchmarkSum(b *testing.B) {
    time.Sleep(3*time.Second)
	for n := 0; n < b.N; n++ {
		Sum()
	}
}

func Sum() {
	sum := 1
	for sum < 1000 {
        sum++
	}
}

```
```bash
BenchmarkSum
BenchmarkSum-12                1        3000369339 ns/op
PASS
```

使用b.ResetTimer()
```golang
func BenchmarkSum(b *testing.B) {
	time.Sleep(3 * time.Second)
	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		Sum()
	}
}
```
```bash
BenchmarkSum
BenchmarkSum-12          4972824               237.8 ns/op
```
## 3.2、结合b.StopTimer()、b.StartTimer()来跳过耗时的初始化操作
```golang
func BenchmarkSum(b *testing.B) {
	for n := 0; n < b.N; n++ {
		b.StopTimer()
		time.Sleep(3 * time.Second)
		b.StartTimer()
		Sum()
	}
}
```
```bash
BenchmarkSum
BenchmarkSum-12          4977744               245.7 ns/op
PASS
```