---
 layout: post
 title: Golang动态参数 args ...interface{}
 tags: [ golang, interface, 动态参数, go]
 comments: true
 image:
  background: triangular.png
---

大家在使用 golang 的动态参数特性时候注意以下几点：

```golang
func test(args ...interface){  
  
}  
func args(args ...interface{}) {  
  
  test(args)  
  test(args...)  
}
```

##  
1. args函数在接受的时候支持 args(a,b,c,d）
2. test(args) 这里的args 虽然设置的动态参数，但是传递进去相当于 args []interface  数组
3. 如果想让参数以动态参数传递 采用格式 args...进行传递



