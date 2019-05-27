## 目录
* [概述](#概述)
* [几种可选方法介绍](#几种可选方法介绍)
* [use](#use)
* [如何查看当前nginx的IO方法](#如何查看当前nginx的IO方法)

# 概述
nginx支持多种连接处理IO方法，特定IO方法的可用性取决于所使用的平台。如果平台支持多种IO方法，nginx会自动选择最有效的IO方法。但是，如果需要的话，也可以通过指令use指定具体的IO方法

如果不指定use，则选择系统和nginx可以支持的最优IO模型

configure编译选项中可以选择--without-poll_module、--with-poll_module、--without-select_module、--with-select_module使其强制使用或者禁止使用某IO模型

如果使用--without-select_module，则不可以使用use select指令了，报错提示：invalid event type "select"
  
如果使用--with-select_module，则可以使用use epoll指令，最后nginx使用的是epoll的IO模型

# 几种可选方法介绍
**select**

  标准方法。如果平台没有更有效的方法的话，则这个方法会自动被选择和构建

**poll**

  标准方法。如果平台没有更有效的方法的话，则这个方法会自动被选择和构建
  
**kqueue**
  
  优先选择的有效方法（在FreeBSD 4.1+，OpenBSD 2.9+，NetBSD 2.0和macOS平台上）
  
**epoll**

  优先选择的有效方法（在Linux 2.6+平台上）
  
 **/dev/poll**

  优先选择的有效方法（在Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+，和Tru64 UNIX 5.1A+平台上）

 **eventport**
 
  建议使用/dev/poll替代，该IO模型用于Solaris 10+ 
  
 # use
```
  Syntax:	use method;
  Default:	—
  Context:	events
```
```
  events{
    use select;
    #use epoll;
  }
```
通常不会指定，因为nginx会默认选择最优的方案

# 如何查看当前nginx的IO方法

```
  server{
    ...
    error_log /tmp/nginx_error.log debug;  #注意，debug级别需要configure --with-debug
    ...
  }
```
可以指定error_log为debug级别，然后会有debug信息
```
2018/10/23 23:27:40 [debug] 90859#0: epoll add event: fd:7 op:1 ev:00002001 
```
