# <a name="linuxRuntime" />Linux Runtime


## <a name="runtimeLinuxFileDescriptors" />File descriptors

默认情况下，运行时只会为应用一直打开 `stdin`, `stdout` 和 `stderr` 文件描述符。
运行时 MAY 将其他的文件描述符传递给应用来支持[套接字激活][socket-activated-containers] 等功能。
一些文件描述符可以被重定向到 `/dev/null`，尽管它们是打开的。


## <a name="runtimeLinuxDevSymbolicLinks" /> Dev symbolic links

当创建容器时（[生命周期](runtime_zh.md#lifecycle)的第二步），如果处理 [`mounts`](config_zh.md#mounts) 后源文件存在，则运行时 MUST 创建以下符号链接：

Source | Destination
--- | ---
/proc/self/fd   | /dev/fd
/proc/self/fd/0 | /dev/stdin
/proc/self/fd/1 | /dev/stdout
/proc/self/fd/2 | /dev/stderr

[socket-activated-containers]: http://0pointer.de/blog/projects/socket-activated-containers.html
