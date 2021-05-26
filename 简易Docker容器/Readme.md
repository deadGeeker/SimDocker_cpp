# Linux NameSpace技术
---
+ 在C++中，每一个namespace就相当于一个单独隔离的容器，容器中有定义好的变量或是函数，只要 namespace 的名称不同，就能够让 namespace 中的代码名称相同，从而解决了代码名称冲突的问题。
+ Linux Namespace 则是 Linux 内核提供的一种技术，它为应用程序提供了一种资源隔离的方案，和 C++ 中的 namespace 有异曲同工之妙。
+ 在 Docker 技术中，我们时常听说 LXC、操作系统级虚拟化这些名词，而 LXC 就是利用了 Namespace 这种技术实现了不同容器之前的资源隔离。利用 Namespace 技术，不同的容器内进程属于不同的 Namespace，彼此不相干扰。总的来说，Namespace 技术，提供了虚拟化的一种轻量级形式，使我们可以从不同方面来运行系统全局属性。
+ 在Linux中，和Namespace相关的系统调用最重要的就是clone()。clone() 的作用是在创建进程时，将线程限制在某个Namespace中。

---

# 对系统调用封装
---
+  Linux 系统调用都是由 C 语言写成的，要编写的是 C++ 相关的代码，为了让整套代码不是出于 C 和 C++ 混搭的风格，先对这些必要的 API 进行一层 C++ 形式的封装，同时也能更加深入的了解这些 API 的使用。
+ clone()：
  + + clone 和 fork 两个系统调用的功能非常类似，他们都是 linux 提供的创建进程的系统调用。
  + + 函数原型：int clone(int (*fn)(void *), void *child_stack, int flags, void *arg);
  + + fork 在创建一个进程的时候，子进程会完全复制父进程的资源。而 clone 则比 fork 更加强大，因为 clone 可以有选择性的将父进程的资源复制给子进程，而没有复制的数据结构则通过指针的复制让子进程共享(arg)，具体要复制的资源，则可以通过 flags 进行指定，并返回子进程的 PID。
  + + 进程主要由四个要素组成：程序、专用堆栈空间、进程控制块、进程专有的Namespace
  + + 前两点要素在 clone 中对应的参数很明显，分别为 fn 和 child_stack。而对于进程控制块来说，由操作系统自动创建，因此，Namespace 就落在了 flags 的身上。
+ execv()：
  + + 函数原型：int execv(const char *path, char *const argv[]);
  + + execv 可以通过传入一个 path 来执行 path 中的执行文件，这个系统调用可以让子进程执行 /bin/bash 从而让整个容器保持运行。

+ sethostname()
  + + 函数原型：int sethostname(const char *name, size_t len);
  + + 这个系统调用能够设置我们的主机名

+ chdir()
  + + 函数原型：int chdir(const char *path);
  + + 任何一个程序，都会在某个特定的目录下运行。需要访问资源时，就可以通过相对路径而不是绝对路径来访问相关资源。而 chdir 恰好提供给了一个便利之处，那就是可以改变我们程序的运行目录，从而达到某些不可描述的目的。

+ chroot()
  + + 函数原型：int chroot(const char *path);
  + + 这个系统调用能够用于设置根目录

+ mount()
  + + 函数原型：int mount(const char *source, const char *target,
                 const char *filesystemtype, unsigned long mountflags,
                 const void *data);
  + + 这个系统调用用于挂载文件系统