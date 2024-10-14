# <a name="linuxContainerConfiguration" />Linux Container Configuration

本文档描述了 [容器配置](config_zh.md) 的 [Linux 特定部分](config_zh.md#platform-specific-configuration) 的架构。 
Linux 容器规范使用各种内核功能（例如命名空间、cgroup、capabilities、LSM 和 filesystem jails）来满足规范。


## <a name="configLinuxDefaultFilesystems" />Default Filesystems

Linux ABI 包括系统调用和几个特殊的文件路径。
期望在 Linux 环境下运行的应用程序很可能希望这些文件路径设置正确。

每个容器的文件系统中都 SHOULD 使以下文件系统可用：

Path    | Type  
--- | ---
/proc    | [proc][]
/sys     | [sysfs][]
/dev/pts | [devpts][]
/dev/shm | [tmpfs][] 


## <a name="configLinuxNamespaces" />Namespaces

命名空间将全局系统资源包装在抽象中，使命名空间内的进程看起来拥有自己的全局资源的隔离实例。
对全局资源的更改对于命名空间成员的其他进程可见，但对于其他进程不可见。
有关更多信息，请参见 [namespaces(7)][namespaces.7_2] 手册页。

命名空间被指定为 `namespaces` 根字段内的条目数组。
可以指定以下参数来设置命名空间：

* **`type`** *(string, REQUIRED)* 命名空间类型。以下类型的命名空间 SHOULD 支持：
  * **`pid`** 容器内的进程只能看到同一容器内或同一 `pid` 命名空间内的其他进程。
  * **`network`** 容器将有自己的网络堆栈。
  * **`mount`** 容器将有一个独立的挂载表。
  * **`ipc`** 容器内的进程只能通过系统级 IPC 与同一容器内的其他进程进行通信。
  * **`uts`** 容器将能够拥有自己的主机名和域名。
  * **`user`** 容器将能够将主机上的用户和组 ID 重新映射到容器内的本地用户和组。
  * **`cgroup`** 容器将具有 cgroup 层次结构的独立视图。
  * **`time`** 容器将能够拥有自己的时钟。
* **`path`** *(string, OPTIONAL)* 命名空间文件。
    该值 MUST 是 [运行时挂载命名空间](glossary_zh.md#runtime-namespace)中的绝对路径。
    运行时 MUST 将容器进程放置在与该 `path` 关联的命名空间中。
    如果 `path` 未与类型 `type` 的命名空间关联，则运行时 MUST [生成错误](runtime_zh.md#errors)。

    如果未指定 `path`，则运行时必须创建类型为 `type` 的新 [容器命名空间](glossary_zh.md#container-namespace)。

如果 `namespaces` 数组中未指定命名空间类型，则容器 MUST 继承该类型的 [运行时命名空间](glossary_zh.md#runtime-namespace)。
如果 `namespaces` 字段包含具有相同 `type` 的重复名称空间，则运行时 MUST [生成错误](runtime_zh.md#errors)。


### Example

```json
"namespaces": [
    {
        "type": "pid",
        "path": "/proc/1234/ns/pid"
    },
    {
        "type": "network",
        "path": "/var/run/netns/neta"
    },
    {
        "type": "mount"
    },
    {
        "type": "ipc"
    },
    {
        "type": "uts"
    },
    {
        "type": "user"
    },
    {
        "type": "cgroup"
    },
    {
        "type": "time"
    }
]
```


## <a name="configLinuxUserNamespaceMappings" />User namespace mappings

**`uidMappings`** (array of objects, OPTIONAL) 描述从主机到容器的用户命名空间 uid 映射。
**`gidMappings`** (array of objects, OPTIONAL) 描述从主机到容器的用户命名空间 gid 映射。

每个条目都有以下结构：

* **`containerID`** *(uint32, REQUIRED)* - 是容器中的起始 uid/gid。
* **`hostID`** *(uint32, REQUIRED)* - 是主机上要映射到 *containerID* 的起始uid/gid。
* **`size`** *(uint32, REQUIRED)* -  是要映射的 id 数量。

运行时 SHOULD NOT 修改引用的文件系统的所有权来实现映射。
请注意，映射条目的数量 MAY 受到 [内核][user-namespaces] 的限制。


### Example

```json
"uidMappings": [
    {
        "containerID": 0,
        "hostID": 1000,
        "size": 32000
    }
],
"gidMappings": [
    {
        "containerID": 0,
        "hostID": 1000,
        "size": 32000
    }
]
```


## <a name="configLinuxTimeOffset" />Offset for Time Namespace

**`timeOffsets`** (object, OPTIONAL) 设置时间命名空间的偏移量。
有关详细信息，请参阅 [time_namespaces][time_namespaces.7]。

时钟名称是条目关键字。
条目值是具有以下属性的对象：

* **`secs`** *(int64, OPTIONAL)* - 是容器中时钟的偏移量（以秒为单位）。
* **`nanosecs`** *(uint32, OPTIONAL)*  是容器中时钟的偏移量（以纳秒为单位）。


## <a name="configLinuxDevices" />Devices

**`devices`** (array of objects, OPTINOAL) 例出容器中 MUST 可用的设备。
运行时 MAY 按照自己喜欢的方式提供它们（使用 [`mknod`][mknod.2] ，通过从运行时挂载命名空间绑定挂载，使用 `symlinks`  等）。

每个条目有以下结构：

* **`tpye`** *(string, REQUIRED)* - 设备类型：`c`、`b`、`u` 或 `p`。
  更多信息请参见 [`mknod(1)`][mknod.1] 。
* **`path`** *(string, REQUIRED)* - 容器内设备的完整路径。
  如果 [file][] 已存在于与请求的设备不匹配的 `path` 中，则运行时 MUST 生成错误。
  该路径 MAY 位于容器文件系统中的任何位置，特别是 `/dev` 之外。
* **`major, minor`** *(int64, REQUIRED 除非 `type` 是 `p`)* - 设备的[主、次编号][devices] 。
* **`fileMode`** *(uint32, OPTIONAL)* - 设备的文件模式。
  你还可以 [使用 cgroups](#configLinuxDeviceAllowedlist) 控制对设备的访问。
* **`uid`** *(uint32, OPTIONAL)* - 在 [容器命名空间](glossary_zh.md#container-namespace) 中设备所有者的 id。
* **`gid`** *(uint32, OPTIONAL)* 在 [容器命名空间](glossary_zh.md#container-namespace) 中设备所在组的 id。

同一 `type`、`major` 和 `minor` 设备 SHOULD NOT 用于多个设备。

容器 MAY NOT 访问未在 **`devices`** 数组中显式引用或未列为 [默认设备](#configLinuxDefaultDevices) 一部分的任何设备节点。
理由：基于虚拟机的运行时需要能够调整节点设备，访问未调整的设备节点可能会出现未定义的行为。


### Example

```json
"devices": [
    {
        "path": "/dev/fuse",
        "type": "c",
        "major": 10,
        "minor": 229,
        "fileMode": 438,
        "uid": 0,
        "gid": 0
    },
    {
        "path": "/dev/sda",
        "type": "b",
        "major": 8,
        "minor": 0,
        "fileMode": 432,
        "uid": 0,
        "gid": 0
    }
]
```


### <a name="configLinuxDefaultDevices" />Default Devices

除了使用此设置配置的任何设备之外，运行时还 MUST 提供：

* [`/dev/null`][null.4]
* [`/dev/zero`][zero.4]
* [`/dev/full`][full.4]
* [`/dev/random`][random.4]
* [`/dev/urandom`][random.4]
* [`/dev/tty`][tty.4]
* `/dev/console` 如果通过将伪终端 `pty` 绑定安装到 `/dev/console` 在配置中启用 [终端](config_zh.md#process) ，则设置。
* [`/dev/ptmx`][pts.4]. [容器的 `/dev/pts/ptmx` 的绑定挂载或符号链接][devpts]。


## <a name="configLinuxControlGroups" />Control groups

也称为 cgroups，它们用于限制容器的资源使用并处理设备访问。
cgroupss 提供控制（通过控制器），以限制容器的 cpu、memory、IO、pids、network 和 RDMA 资源。
更多信息，请参阅 [内核 cgroups 文档][cgroup-v1] 。

运行时 MAY 在特定 [容器操作](runtime_zh.md#operation) （例如 [create](runtime_zh.md#create) 、 [start](runtime_zh.md#start) 或 [exec](runtime_zh.md#exec) ）期间检查容器 cgroup 是否适合目的，
并且如果此类检查失败，则 MUST [生成错误](runtime_zh.md#errors) 。
例如，冻结的 cgroup 或（对于 [create](runtime_zh.md#create) 操作）非空的 cgroup。
这样做的原因是，接受此类配置可能会导致用户无法预料或理解的容器操作结果，例如对一个容器的操作会无意中影响到其他容器。


### <a name="configLinuxCgroupsPath" />Cgroups Path

**`cgroupsPath`** (string, OPTIONAL) cgroups 的路径。
它可用于控制容器的 cgroups 层次结构或在现有容器中运行新进程。

`cgroupsPath` 的值 MUST 是绝对路径或相对路径。

* 在绝对路径（以 `/` 开头）的情况下，运行时 MUST 采用相对于 cgroups 挂载点的路径。
* 在相对路径（不以 `/` 开头）的情况下，运行时 MAY 解释相对于 cgroups 层次结构中运行时确定的位置的路径。

如果指定了该值，则在给定相同 `cgroupsPath` 值的情况下，运行时 MUST 一致地附加到 cgroups 层次结构中的同一位置。
如果未指定该值，运行时 MAY 定义默认 cgroups 路径。
运行时 MAY 认为某些 `cgroupsPath` 值无效，如果是这种情况，则 MUST 生成错误。

规范的实现可以选择以任何方式命名 cgroups。
该规范不包括 cgroups 的命名架构。
由于 [cgroupv2 文档][cgroup-v2] 中讨论的原因，规范不支持每个控制器路径。
如果 cgroups 不存在，则会创建它们。

您可以通过 Linux 配置的 `resources` 字段配置容器的 cgroups。
除非必须更新限制，否则不要指定 `resources` 。
例如，要在现有容器中运行新进程而不更新限制，不需要指定 `resources` 。

运行时 MAY 将容器进程附加到超出满足 `resources` 设置所需的其他 cgroup 控制器。


### Cgroup ownership

运行时 MAY 根据以下规则将容器的 cgroup 的所有者更改为（或导致更改为）映射到容器 [命名空间](glossary_zh.md#container-namespace) 中 `process.user.uid` 的值的主机 uid；也就是将执行容器进程的用户。

当 cgroups v1 正在使用时，运行时 SHOULD NOT 更改容器 cgroups 的所有权。 
cgroups v1 中的 cgroup 委派不安全。

运行时 SHOULD NOT 更改容器 cgroup 的所有权，除非它还会为容器创建新的 cgroup 命名空间。
通常，当 `linux.namespaces` 数组包含 `type` 等于 `"cgroup"` 且 `path` 未设置的对象时，会发生这种情况。

当且仅当 cgroup 文件系统以 读/写 方式挂载时，运行时 SHOULD 更改 cgroup 所有权；
也就是说，当配置的 `mounts` 数组包含一个对象时，其中：

* `source` 字段等于 `"cgroup"`
* `destination` 字段等于 `"/sys/fs/cgroup"`
* `options` 字段不包含值 `"ro"` 

如果配置未指定此类挂载，则运行时 SHOULD NOT 更改 cgroup 所有权。

更改 cgroup 所有权的运行时 SHOULD 仅更改容器的 cgroup 文件夹以及该文件夹中 `/sys/kernel/cgroup/delegate` 中列出的文件的所有权。
有关此文件的详细信息，请参阅 `cgroups(7)`。
请注意，并非 `/sys/kernel/cgroup/delegate` 中列出的所有文件都必须存在于每个 cgroup 中。
在这种情况下，运行时 MUST NOT 失败，并且 SHOULD 更改 cgroup 中存在的列出文件的所有权。

如果 `/sys/kernel/cgroup/delegate` 文件不存在，运行时 MUST 回退到使用以下文件列表：

```
cgroup.procs
cgroup.subtree_control
cgroup.threads
```

运行时 SHOULD NOT 更改任何其他文件的所有权。
更改其他文件可能会允许容器提高其自身的资源限制或执行其他不需要的行为。


### Example

```json
"cgroupPath": "/myRuntime/myContainer",
"resources": {
    "memory": {
        "limit": 100000,
        "reservation": 200000
    },
    "devices": [
        {
            "allow": false,
            "access": "rwm"
        }
    ]
}
```


### <a name="configLinuxDeviceAllowedlist" />Allowed Device list

**`devices`** (array of objects, OPTIONAL) 配置 [允许的设备列表][cgroup-v1-devices] 。
运行时 MUST 按列出的顺序应用条目。

每个条目结构如下:

* **`allow`** *(boolean, REQUIRED)* - 是否允许或拒绝进入。
* **`type`** *(string, OPTIONAL)* - 设备类型：`a`（all）、`c`（char）或 `b`（block）。
  未设置的值表示 "all" ，映射到 `a`。
* **`major, minor`** *(int64, OPTIONAL)* - 设备的 [主要、次要编号][devices] 。
  未设置的值表示 "all" ，映射到 [文件系统 API 中的 `*`][cgroup-v1-devices] 。
* **`access`** *(string, OPTIONAL)* - 设备的 cgroup 权限。
  由 `r`（读）、`w`（写）和 `m`（mknod）组成。


#### Example

```json
"devices": [
    {
        "allow": false,
        "access": "rwm"
    },
    {
        "allow": true,
        "type": "c",
        "major": 10,
        "minor": 229,
        "access": "rw"
    },
    {
        "allow": true,
        "type": "b",
        "major": 8,
        "minor": 0,
        "access": "r"
    }
]
```


### <a name="configLinuxMemory" />Memory

**`memory`** (object, OPTIONAL) 代表cgroup 子系统 `memory` ，用于设置容器的内存使用限制。
有关更多信息，请参阅有关 [内存][cgroup-v1-memory] 的内核 cgroups 文档。

内存值通过字节指定限制，或 `-1` 表示无限内存。

* **`limit`** *(int64, OPTIONAL)* - 设置内存使用限制
* **`reservation`** *(int64, OPTIONAL)* - 设置内存使用的软限制
* **`swap`** *(int64, OPTIONAL)* - 设置内存 + 交换空间使用限制
* **`kernel`** *(int64, OPTIONAL, NOT RECOMMENDED)* - 设置内核内存的硬限制
* **`kernelTCP`** *(int64, OPTIONAL, NOT RECOMMENDED)* - 设置内核 TCP 缓冲区内存的硬限制

以下属性不指定内存限制，但由 `memory` 控制器负责：

* **`swappiness`** *(uint64, OPTIONAL)* - 设置 `vmscan` 的 `swappiness` 参数（参见 `sysctl` 的 `vm.swappiness`）。
  值从 0 到 100。越高意味着 `swappy` 越多。
* **`disableOOMKiller`** *(bool, OPTIONAL)* - 启用或禁用 OOM killer。
  如果启用（`false`），试图消耗超过允许内存的任务会立即被 OOM killer 杀死。
  每个使用内存子系统的 cgroup 都默认启用 OOM killer。
  要禁用它，请指定值为 `true`。
* **`useHierarchy`** *(bool, OPTIONAL)* - 启用或禁用分层内存记帐（hierarchical memory accounting）。
  如果启用（`true`），子 cgroups 将共享此 cgroup 的内存限制。
* **`checkBeforeUpdate`** *(bool, OPTIONAL)* - 在设置新限制之前启用容器内存使用情况检查。
  如果启用（`true`），运行时 MAY 检查新的内存限制是否低于当前使用情况，并且 MUST 拒绝新的限制。
  实际上，当使用 `cgroup v1` 时，内核会拒绝低于当前使用情况的限制，而当使用 `cgroup v2` 时，则会调用 OOM killer。
  此设置可用于 `cgroup v2` 来模仿 `cgroup v1` 行为。


#### Example

```json
"memory": {
    "limit": 536870912,
    "reservation": 536870912,
    "swap": 536870912,
    "kernel": -1,
    "kernelTCP": -1,
    "swappiness": 0,
    "disableOOMKiller": false
}
```


### <a name="configLinuxCPU" />CPU

**`cpu`** (object, OPTIONAL) 代表 cgroup 子系统 `cpu` 和 `cpusets`。
有关更多信息，请参阅有关 [cpusets][cgroup-v1-cpusets] 的内核 cgroups 文档。

可以指定以下参数来设置控制器：

* **`shares`** *(uint64, OPTIONAL)* - 指定 cgroup 中任务可用的 CPU 时间的相对份额
* **`quota`** *(int64, OPTIONAL)* - 以微秒为单位指定 cgroup 中所有任务在一个时间段内可以运行的总时间（由下面的 **`period`** 定义）。  
  如果指定了任何（有效）正值，它 MUST 不小于 `burst`（运行时 MAY 会产生错误）。
* **`burst`** *(uint64, OPTIONAL)* - 指定 cgroup 中的所有任务在一个周期内（由下面的 `period` 定义）可以额外突发运行的最大累计时间（以微秒为单位）。
  如果指定，该值 MUST 不大于任何正 `quota`（运行时 MAY 会产生错误）。
* **`period`** *(uint64, OPTIONAL)* - 指定应重新分配 cgroup 对 CPU 资源的访问权限的时间段（以微秒为单位）（仅限 CFS 调度程序）
* **`realtimeRuntime`** *(int64, OPTIONAL)* - 指定 cgroup 中的任务可以访问 CPU 资源的最长连续时间段（以微秒为单位）
* **`realtimePeriod`** *(uint64, OPTIONAL)* - 与 **`period`** 相同，但仅适用于实时调度程序
* **`cpus`** *(string, OPTIONAL)* - 容器将在其上运行的 CPU 列表。这是一个以逗号分隔的列表，其中的破折号表示范围。例如， `"0-3,7"` 表示 CPU 0、1、2、3 和 7。
* **`mems`** *(string, OPTIONAL)* - 容器将在其上运行的内存节点列表。这是一个以逗号分隔的列表，其中的破折号表示范围。例如， `"0-3,7"` 表示内存节点 0、1、2、3 和 7。
* **`idle`** *(int64, OPTIONAL)* - cgroup 配置了最小权重，0：默认行为，1：SCHED_IDLE。


#### Example

```json
"cpu": {
    "shares": 1024,
    "quota": 1000000,
    "burst": 1000000,
    "period": 500000,
    "realtimeRuntime": 950000,
    "realtimePeriod": 1000000,
    "cpus": "2-3",
    "mems": "0-7",
    "idle": 0
}
```


### <a name="configLinuxBlockIO" />Block IO

**`blockIO`** (object, OPTIONAL) 表示实现块 IO 控制器的 cgroup 子系统 `blkio`。
有关更多信息，请参阅有关 cgroup v1 的 [blkio][cgroup-v1-blkio] 的内核 cgroups 文档或 cgroup v2 的 [io][cgroup-v2-io] 。

请注意，由于内核实现限制，cgroup v1 中的 I/O 限制设置仅适用于直接 I/O，而 cgroup v2 中不存在此限制。

可以指定以下参数来设置控制器：

* **`weight`** *(uint16, OPTIONAL)* - 指定每个 cgroup 的权重。这是所有设备上组的默认权重，除非被每个设备规则覆盖。
* **`leafWeight`** *(uint16, OPTIONAL)* - `weight` 的等价物，用于决定给定 cgroup 中的任务在与 cgroup 的子 cgroup 竞争时有多少权重。
* **`weightDevice`** *(array of objects, OPTIONAL)* - 每个设备带宽权重的数组。
  每个条目都有以下结构：
  * **`major, minor`** *(int64, REQUIRED)* - 设备的主要、次要编号。
    有关更多信息，请参见 [mknod(1)][mknod.1] 手册页。
  * **`weight`** *(uint16, OPTIONAL)* - 设备的带宽权重。
  * **`leafWeight`** *(uint16, OPTIONAL)* - 与 cgroup 的子 cgroup 竞争时设备的带宽权重，仅限 CFQ 调度程序

  你 MUST 在给定条目中至少指定 `weight` 或 `leafWeight` 之一，并且 MAY 同时指定两者。

* **`throttleReadBpsDevice`**, **`throttleWriteBpsDevice`** *(array of objects, OPTIONAL)* - 每个设备带宽速率限制的数组。
  每个条目都有以下结构：
  * **`major, minor`** *(int64, REQUIRED)* - 设备的主要、次要编号。
  有关更多信息，请参见 [mknod(1)][mknod.1] 手册页。
  * **`rate`** *(uint64, REQUIRED)* - 设备的带宽速率限制（以字节/秒为单位）

* **`throttleReadIOPSDevice`**, **`throttleWriteIOPSDevice`** *(array of objects, OPTIONAL)* - 每个设备 IO 速率限制的数组。
  每个条目都有以下结构：
  * **`major, minor`** *(int64, REQUIRED)* - 设备的主要、次要编号。
  有关更多信息，请参见 [mknod(1)][mknod.1] 手册页。
  * **`rate`** *(uint64, REQUIRED)* - 设备的 IO 速率限制


#### Example

```json
"blockIO": {
    "weight": 10,
    "leafWeight": 10,
    "weightDevice": [
        {
            "major": 8,
            "minor": 0,
            "weight": 500,
            "leafWeight": 300
        },
        {
            "major": 8,
            "minor": 16,
            "weight": 500
        }
    ],
    "throttleReadBpsDevice": [
        {
            "major": 8,
            "minor": 0,
            "rate": 600
        }
    ],
    "throttleWriteIOPSDevice": [
        {
            "major": 8,
            "minor": 16,
            "rate": 300
        }
    ]
}
```


### <a name="configLinuxHugePageLimits" />Huge page limits

**`hugepageLimits`** (array of objects, OPTIONAL) 代表 `hugetlb` 控制器，它允许限制 HugeTLB 保留（如果支持）或使用（页错误）。
如果内核支持，默认情况下 `hugepageLimits` 会为 HugeTLB 控制器预留核算定义 hugepage 大小和限制，从而限制每个控制组的 HugeTLB 预留，并在预留时和没有预留的 HugeTLB 内存出现故障时强制执行控制器限制。
此外，如果内核不支持，则应回退到页错误计数，这允许用户限制每个控制组的 HugeTLB 使用（页错误）并在页面错误期间强制执行限制。

请注意，预留限制优于页错误限制，因为预留限制是在预留时间（在 mmap 或 shget 上）强制执行的，并且如果事先预留了内存，则永远不会导致应用程序获取 SIGBUS 信号。
这样就可以更容易地使用非 HugeTLB 内存等替代品。在页面故障会计的情况下，很难避免进程获得 SIGBUS，因为系统管理员需要精确了解系统中所有任务的 HugeTLB 使用情况，并确保有足够的页面来满足所有请求。
使用页面错误统计来避免在过度使用的系统上获取 SIGBUS 的任务实际上是不可能的。

有关详细信息，请参阅有关 [HugeTLB][cgroup-v1-hugetlb] 的内核 cgroups 文档。

每个条目都有以下结构：

* **`pageSize`** *(string, REQUIRED)* - 大页大小。
  该值的格式为 `<size><unit-prefix>B`（64KB、2MB、1GB），并且必须与 `/sys/fs/cgroup/hugetlb/hugetlb.<hugepagesize>.rsvd.limit_in_bytes` 中相应控制文件匹配。（如果支持 `hugetlb_cgroup` 预留）或 `/sys/fs/cgroup/hugetlb/hugetlb.<hugepagesize>.limit_in_bytes`（如果不支持）。
  `<unit-prefix>` 的值应以 1024 为基数进行解析（"1KB" = 1024，"1MB" = 1048576 等）。
* **`limit`** *(uint64, REQUIRED)* - HugeTLB 预留（如果支持）或使用的 *hugepagesize* 字节数限制。


#### Example

```json
"hugepageLimits": [
    {
        "pageSize": "2MB",
        "limit": 209715200
    },
    {
        "pageSize": "64KB",
        "limit": 1000000
    }
]
```


### <a name="configLinuxNetwork" />Network

**`network`** (object, OPTIONAL) 表示 cgroup 子系统 `net_cls` 和 `net_prio`。
有关更多信息，请参阅有关 [net\_cls cgroup][cgroup-v1-net-cls] 和 [net\_prio cgroup][cgroup-v1-net-prio] 的内核 cgroups 文档。

可以指定以下参数来设置控制器：

* **`classID`** *(uint32, OPTIONAL)* - 是 cgroup 的网络数据包将被标记的网络类别标识符
* **`priorities`** *(array of objects, OPTIONAL)* - 指定分配给源自组中进程并在各种接口上传出系统的流量的优先级对象列表。
  可以按优先级指定以下参数：
  * **`name`** *(string, REQUIRED)* - [运行时网络命名空间](glossary_zh.md#runtime-namespace) 中的接口名称
  * **`priority`** *(uint32, REQUIRED)* - 应用于接口的优先级
 

#### Example

```json
"network": {
    "classID": 1048577,
    "priorities": [
        {
            "name": "eth0",
            "priority": 500
        },
        {
            "name": "eth1"
        }
    ]
}
```


### <a name="configLinuxPIDS" />PIDs

**`pids`** (object, OPTIONAL) 代表 cgroup 子系统 `pids` 。
有关详细信息，请参阅有关 [pids][cgroup-v1-pids] 的内核 cgroups 文档。

可以指定以下参数来设置控制器：

* **`limit`** *(int64, REQUIRED)* - 指定cgroup 中任务的最大数量


#### Example

```json
"pids": {
    "limit": 32771
}
```


### <a name="configLinuxRDMA" />RDMA

**`rdma`** (object, OPTIONAL) 表示 cgroup 子系统 `rdma` 。
有关更多信息，请参阅有关 [rdma][cgroup-v1-rdma] 的内核 cgroups 文档。

要限制的设备的名称是条目键。
条目值是具有以下属性的对象：

* **`hcaHandles`** *(uint32, OPTIONAL)* - 指定 cgroup 中 `hca_handles` 的最大数量
* **`hcaObjects`** *(uint32, OPTIONAL)* - 指定 cgroup 中 `hca_objects` 的最大数量

您 MUST 在给定条目中至少指定 `hcaHandles` 或 `hcaObject` 之一，并且 MAY 同时指定两者。


#### Example

```json
"rdma": {
    "mlx5_1": {
        "hcaHandles": 3,
        "hcaObjects": 10000
    },
    "mlx4_0": {
        "hcaObjects": 1000
    },
    "rxe3": {
        "hcaObjects": 10000
    }
}
```


## <a name="configLinuxUnified" />Unified

**`unified`** (object, OPTIONAL) 允许为容器设置和修改 cgroup v2 参数。

对象中的每个键都引用 cgroup 统一层次结构中的一个文件。

OCI 运行时 MUST 确保为 cgroup 启用所需的 cgroup 控制器。

运行时未知的配置仍然 MUST 写入相关文件。

当配置引用不存在或无法启用的 cgroup 控制器时，运行时 MUST 生成错误。


### Example

```json
"unified": {
    "io.max": "259:0 rbps=2097152 wiops=120\n253:0 rbps=2097152 wiops=120",
    "hugetlb.1GB.max": "1073741824"
}
```

如果在 cgroup v2 层次结构上启用了控制器，但为 cgroup v1 等效控制器提供了配置，则运行时可以尝试转换。

如果无法进行转换，则运行时 MUST 生成错误。


## <a name="configLinuxIntelRdt" />IntelRdt

**`intelRdt`** (object, OPTIONAL) 代表 [英特尔资源管理器技术][intel-rdt-cat-kernel-interface]。
如果设置了 `intelRdt`，则运行时 MUST 将容器进程 ID 写入已挂载的 `resctrl` 伪文件系统中正确子文件夹中的 `tasks` 文件中。
该子文件夹名称由 `closID` 参数指定。
如果 [运行时挂载命名空间](glossary_zh.md#runtime-namespace) 中没有可用的挂载 `resctrl` 伪文件系统，则运行时MUST [生成错误](runtime_zh.md#errors)。

如果未设置 `intelRdt`，则运行时 MUST NOT 操作任何 `resctrl` 伪文件系统。

可以为容器指定以下参数：

* **`closID`** *(string, OPTIONAL)* - 指定 RDT 服务类别 (CLOS) 的标识。
* **`l3CacheSchema`** *(string, OPTIONAL)* - 指定 L3 缓存 ID 和容量位掩码 (CBM) 的架构。
  该值 SHOULD 以 `L3:` 开头并且 SHOULD NOT 包含换行符。
* **`memBwSchema`** *(string, OPTIONAL)* - 指定每个 L3 缓存 ID 的内存带宽模式。
  该值 MUST 以 `MB:` 开头，并且 MUST NOT 包含换行符。

MUST 应用以下参数规则：

* 如果同时设置了 `l3CacheSchema` 和 `memBwSchema`，则运行时 MUST 将组合值写入 `closID` 中讨论的子文件夹中的 `schemata` 文件。

* 如果 `l3CacheSchema` 包含以 `MB:` 开头的行，则写入 `schemata` 文件的值 MUST 是来自 `l3CacheSchema` 的非 `MB:` 行和来自 `memBWSchema` 的行。

* 如果设置了 `l3CacheSchema` 或 `memBwSchema`，则运行时必须将值写入 `closID` 中讨论的该子文件夹中的 `schemata` 文件。

* 如果 `l3CacheSchema` 和 `memBwSchema` 均未设置，则运行时 MUST NOT 写入任何 `resctrl` 伪文件系统中的 `schemata` 文件。

* 如果未设置 `closID`，运行时 MUST 从 [`start`](runtime_zh.md#start) 就使用容器 ID 并创建 `<container-id>` 文件夹。

* 如果设置了 `closID`，同时设置了 `l3CacheSchema` 和/或 `memBwSchema`
  * 如果已挂载的 `resctrl` 伪文件系统中的 `closID` 文件夹不存在，则运行时 MUST 创建它。
  * 如果已挂载的 `resctrl` 伪文件系统中存在 `closID` 文件夹，则运行时 MUST 将 `l3CacheSchema` 和/或 `memBwSchema` 值与 `schemata` 文件进行比较，如果不匹配则 [生成错误](runtime_.md#errors) 。

* 如果设置了 `closID`，并且 `l3CacheSchema` 和 `memBwSchema` 均未设置，则运行时 MUST 检查已挂载的 `resctrl` 中是否存在相应的预配置文件夹 `closID`。
  如果存在这样的预配置文件夹 `closID`，则运行时 MUST 将容器分配给该 `closID`，并在文件夹不存在时 [生成错误](runtime_zh.md#errors) 。

* **`enableCMT`** *(boolean, OPTIONAL)* - 指定是否应启用 Intel RDT CMT：
  * CMT（缓存监控技术）支持监控容器的末级缓存（LLC）占用情况。

* **`enableMBM`** *(boolean, OPTIONAL)* - 指定是否启用英特尔 RDT MBM：
  * MBM（内存带宽监控）支持监控容器的总内存带宽和本地内存带宽。


### Example 

考虑具有两个 L3 缓存的双插槽计算机，其中默认 CBM 为 `0x7ff`，最大 CBM 长度为 11 位，最小内存带宽为 10%，内存带宽粒度为 10%。

容器内的任务只能访问 socket 0 上的 L3 缓存的 "upper" 7/11 和 socket 1 上的 "lower" 5/11 L3 缓存，并且可以在 socket 0 使用最大内存带宽 20%，socket 1 上 70%。

```json
"linux": {
    "intelRdt": {
        "closID": "guaranteed_group",
        "l3CacheSchema": "L3:0=7f0;1=1f",
        "memBwSchema": "MB:0=20;1=70"
    }
}
```


## <a name="configLinuxSysctl" />Sysctl

**`sysctl`** (object, OPTIONAL) 允许在运行时修改容器的内核参数。
有关更多信息，请参见 [sysctl(8)][sysctl.8] 手册页。


### Example

```json
"sysctl": {
    "net.ipv4.ip_forward": "1",
    "net.core.somaxcomm": "256"
}
```


## <a name="configLinuxSeccomp" />Seccomp

Seccomp 在 Linux 内核中提供了应用程序沙箱机制。
Seccomp 配置允许配置对匹配的系统调用采取的操作，此外还允许对作为参数传递给系统调用的值进行匹配。
有关 Seccomp 的更多信息，请参阅 [Seccomp][seccomp] 内核文档。
操作、体系结构和运算符是与 [libseccomp][] 的 `seccomp.h` 中的定义匹配的字符串，并被转换为相应的值。

**`seccomp`** (object, OPTIONAL)

可以指定以下参数来设置 seccomp：

* **`defaultAction`** *(string, REQUIRED)* - seccomp 的默认操作。允许的值与 `syscalls[].action` 相同。
* **`defaultErrnoRet`** *(uint, OPTIONAL)* - 要使用的 errno 返回码。
  某些操作（例如 `SCMP_ACT_ERRNO` 和 `SCMP_ACT_TRACE`）允许指定要返回的 errno 代码。
  当操作不支持 `errno` 时，运行时 MUST 打印错误并失败。
  如果未指定，则其默认值为 `EPERM`。
* **`architectures`** *(array of strings, OPTIONAL)* - 用于系统调用的体系结构。
  下面显示了自 `libseccomp` v2.5.0 起的有效常量列表。

  * `SCMP_ARCH_X86`
  * `SCMP_ARCH_X86_64`
  * `SCMP_ARCH_X32`
  * `SCMP_ARCH_ARM`
  * `SCMP_ARCH_AARCH64`
  * `SCMP_ARCH_MIPS`
  * `SCMP_ARCH_MIPS64`
  * `SCMP_ARCH_MIPS64N32`
  * `SCMP_ARCH_MIPSEL`
  * `SCMP_ARCH_MIPSEL64`
  * `SCMP_ARCH_MIPSEL64N32`
  * `SCMP_ARCH_PPC`
  * `SCMP_ARCH_PPC64`
  * `SCMP_ARCH_PPC64LE`
  * `SCMP_ARCH_S390`
  * `SCMP_ARCH_S390X`
  * `SCMP_ARCH_PARISC`
  * `SCMP_ARCH_PARISC64`
  * `SCMP_ARCH_RISCV64`

* **`flags`** *(array of strings, OPTIONAL)* - 与 seccomp(2) 一起使用的标志列表。

  下面显示了有效的常量列表。

  * `SECCOMP_FILTER_FLAG_TSYNC`
  * `SECCOMP_FILTER_FLAG_LOG`
  * `SECCOMP_FILTER_FLAG_SPEC_ALLOW`
  * `SECCOMP_FILTER_FLAG_WAIT_KILLABLE_RECV`

* **`listenerPath`** *(string, OPTIONAL)* - 指定 UNIX 域套接字的路径，当使用 `SCMP_ACT_NOTIFY` 操作时，运行时将通过该路径发送 [容器进程状态](#containerprocessstate) 数据结构。
  该套接字 MUST 使用 `AF_UNIX` 域和 `SOCK_STREAM` 类型。
  运行时 MUST 为每个连接发送一个 [容器进程状态](#containerprocessstate) 。
  连接 MUST NOT 重复使用，并且必须在发送 seccomp 状态后关闭。
  如果发送到此套接字失败，运行时 MUST [生成错误](runtime_zh.md#errors) 。
  如果未使用 `SCMP_ACT_NOTIFY` 操作，则忽略此值。

  运行时使用 `SCM_RIGHTS` 发送以下文件描述符，并在 [容器进程状态](#containerprocessstate) 的 `fds` 数组中设置它们的名称：

  * **`seccompFd`** (string, REQUIRED) 是 seccomp 系统调用返回的 seccomp 文件描述符。

* **`listenerMetadata`** *(string, OPTIONAL)* - 指定要传递给 seccomp 代理的不透明数据。
  该字符串将作为 [容器进程状态](#containerprocessstate)中的 `metadata` 字段发送。
  如果未设置 `listenerPath`，则 MUST NOT 设置此字段。

* **`syscalls`** *(array of objects, OPTIONAL)* - 匹配 seccomp 中的系统调用。
  虽然此属性是 OPTIONAL 的，但如果没有系统调用条目，`defaultAction` 的某些值将无用。
  例如，如果 `defaultAction` 为 `SCMP_ACT_KILL` 并且 `syscalls` 为空或未设置，则内核将在第一个系统调用时终止容器进程。
  每个条目都有以下结构：

  * **`names`** *(array of strings, REQUIRED)* - 系统调用的名称。
    `names` 必须包含至少一项。
  * **`action`** *(string, REQUIRED)* - seccomp 规则的操作。
    下面显示了自 `libseccomp` v2.5.0 起的有效常量列表。

    * `SCMP_ACT_KILL`
    * `SCMP_ACT_KILL_PROCESS`
    * `SCMP_ACT_KILL_THREAD`
    * `SCMP_ACT_TRAP`
    * `SCMP_ACT_ERRNO`
    * `SCMP_ACT_TRACE`
    * `SCMP_ACT_ALLOW`
    * `SCMP_ACT_LOG`
    * `SCMP_ACT_NOTIFY`

  * **`errnoRet`** *(uint, OPTIONAL)* - 要使用的 `errno` 返回码。
    某些操作（例如 `SCMP_ACT_ERRNO` 和 `SCMP_ACT_TRACE`）允许指定要返回的 `errno` 代码。
    当操作不支持 `errno` 时，运行时 MUST 打印错误并失败。
    如果未指定，其默认值为 `EPERM`。

  * **`args`** *(array of objects, OPTIONAL)* - seccomp 中的特定系统调用。
    每个条目都有以下结构：
        
    * **`index`** *(uint, REQUIRED)* - seccomp 中系统调用参数的索引。
    * **`value`** *(uint64, REQUIRED)* - seccomp 中系统调用参数的值。
    * **`valueTwo`** *(uint64, OPTIONAL)* - seccomp 中系统调用参数的值。
    * **`op`** *(string, REQUIRED)* - seccomp 中系统调用参数的运算符。
            下面显示了自 `libseccomp` v2.3.2 起的有效常量列表。

      * `SCMP_CMP_NE`
      * `SCMP_CMP_LT`
      * `SCMP_CMP_LE`
      * `SCMP_CMP_EQ`
      * `SCMP_CMP_GE`
      * `SCMP_CMP_GT`
      * `SCMP_CMP_MASKED_EQ`


### Example

```json
"seccomp": {
    "defaultAction": "SCMP_ACT_ALLOW",
    "architectures": [
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
    ],
    "syscalls": [
        {
            "names": [
                "getcwd",
                "chmod"
            ],
            "action": "SCMP_ACT_ERRNO"
        }
    ]
}
```


### <a name="containerprocessstate" />The Container Process State

容器进程状态是通过 UNIX 套接字传递的数据结构。
容器运行时 MUST 通过 UNIX 套接字发送容器进程状态，因为以 JSON 序列化的常规有效负载必须使用 `SCM_RIGHTS` 发送。
容器运行时 MAY 使用多个 `sendmsg(2)` 调用来发送上述数据。
如果使用多个 `sendmsg(2)`，则文件描述符 MUST 仅在第一次调用中发送。

容器进程状态包括以下属性：

* **`ociVersion`** (string, REQUIRED) 是容器进程状态所遵循的 OCI 运行时规范 的版本。
* **`fds`** (array, OPTIONAL) 是一个字符串数组，其中包含传递的文件描述符的名称。
  该数组中名称的索引对应于 `SCM_RIGHTS` 数组中文件描述符的索引。
* **`pid`** (int, REQUIRED) 是运行时看到的容器进程 ID。
* **`metadata`** (string, OPTIONAL) 不透明的元数据。
* **`state`** ([state](runtime_zh.md#state), REQUIRED) 是容器的状态。

在 `SCM_RIGHTS` 数组中发送单个 `seccompFd` 文件描述符的示例：

```json
{
    "ociVersion": "1.0.2",
    "fds": [
        "seccompFd"
    ],
    "pid": 4422,
    "metadata": "MKNOD=/dev/null,/dev/net/tun;BPF_MAP_TYPES=hash,array",
    "state": {
        "ociVersion": "1.0.2",
        "id": "oci-container1",
        "status": "creating",
        "pid": 4422,
        "bundle": "/containers/redis",
        "annotations": {
            "myKey": "myValue"
        }
    }
}
```


## <a name="configLinuxRootfsMountPropagation" />Rootfs Mount Propagation

**`rootfsPropagation`** (string, OPTIONAL) 设置 rootfs 的挂载传播。
它的值可以是 `shared`、`slave`、`private` 或`unbindable` 。
值得注意的是，对等组定义为一组相互传播事件的 VFS 挂载。
嵌套容器定义为在现有容器内启动的容器。


* **`shared`** rootfs 挂载属于新的对等组。
  这意味着进一步的挂载（例如: 嵌套容器）也将属于该对等组并将事件传播到 rootfs。
  请注意，这并不意味着它与主机共享。
* **`slave`** rootfs 挂载从主机接收传播事件（例如，如果某些内容挂载在主机上，它也会出现在容器中），但反之则不然。
* **`private`** rootfs 挂载不会从主机接收挂载传播事件，嵌套容器中的进一步挂载将与主机和 rootfs 隔离（即使嵌套容器 `rootfsPropagation` 选项是共享的）。
* **`unbindable`** rootfs 挂载是私有挂载，无法绑定挂载。

内核文档中的 [共享子树][sharedsubtree] 文章提供了有关挂载传播的更多信息。


### Example

```json
"rootfsPropagation": "slave"
```


## <a name="configLinuxMaskedPaths" />Masked Paths

**`maskedPaths`** (array of strings, OPTIONAL) 将掩盖容器内提供的路径，以便无法读取它们。
这些值 MUST 是 [容器命名空间](glossary_zh.md#container_namespace) 中的绝对路径。


### Example

```json
"maskedPaths": [
    "/proc/kcore"
]
```


## <a name="configLinuxReadonlyPaths" />Readonly Paths

**`readonlyPaths`** (array of strings, OPTIONAL) 将在容器内将提供的路径设置为只读。
这些值 MUST 是 [容器命名空间](glossary_zh.md#container-namespace) 中的绝对路径。


### Example

```json
"readonlyPaths": [
    "/proc/sys"
]
```


## <a name="configLinuxMountLabel" />Mount Label

**`mountLabel`** (string, OPTIONAL) 将为容器中的挂载设置 `Selinux` 上下文。


### Example

```json
"mountLabel": "system_u:object_r:svirt_sanbox_file_t:s0:c715,c811"
```


## <a name="configLinuxPersonality" />Personality

**`personality`** (object, OPTIONAL) 设置 Linux 执行 `personality` 。
有关更多信息，请参阅 [`personality`][personality.2] 系统调用文档。
由于大多数选项已过时且很少使用，并且某些选项会降低安全性，因此当前支持的集只是可用选项的一小部分。

* **`domain`** *(string, REQUIRED)* - 执行域。
  有效的常量列表如下所示。 
  `LINUX32` 会设置 `uname` 系统调用来显示32位 CPU 类型，例如 `i686`。

  * `LINUX`
  * `LINUX32`

* **`flags`** *(array of strings, OPTIONAL)* - 要应用的附加标志。目前不支持任何标志值。

[cgroup-v1]: https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
[cgroup-v1-blkio]: https://www.kernel.org/doc/Documentation/cgroup-v1/blkio-controller.txt
[cgroup-v1-cpusets]: https://www.kernel.org/doc/Documentation/cgroup-v1/cpusets.txt
[cgroup-v1-devices]: https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt
[cgroup-v1-hugetlb]: https://www.kernel.org/doc/Documentation/cgroup-v1/hugetlb.txt
[cgroup-v1-memory]: https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt
[cgroup-v1-net-cls]: https://www.kernel.org/doc/Documentation/cgroup-v1/net_cls.txt
[cgroup-v1-net-prio]: https://www.kernel.org/doc/Documentation/cgroup-v1/net_prio.txt
[cgroup-v1-pids]: https://www.kernel.org/doc/Documentation/cgroup-v1/pids.txt
[cgroup-v1-rdma]: https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt
[cgroup-v2]: https://www.kernel.org/doc/Documentation/cgroup-v2.txt
[cgroup-v2-io]: https://docs.kernel.org/admin-guide/cgroup-v2.html#io
[devices]: https://www.kernel.org/doc/Documentation/admin-guide/devices.txt
[devpts]: https://www.kernel.org/doc/Documentation/filesystems/devpts.txt
[libseccomp]: https://github.com/seccomp/libseccomp
[proc]: https://www.kernel.org/doc/Documentation/filesystems/proc.txt
[seccomp]: https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt
[sharedsubtree]: https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt
[sysfs]: https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt
[tmpfs]: https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt

[full.4]: http://man7.org/linux/man-pages/man4/full.4.html
[mknod.1]: http://man7.org/linux/man-pages/man1/mknod.1.html
[mknod.2]: http://man7.org/linux/man-pages/man2/mknod.2.html
[namespaces.7_2]: http://man7.org/linux/man-pages/man7/namespaces.7.html
[null.4]: http://man7.org/linux/man-pages/man4/null.4.html
[personality.2]: http://man7.org/linux/man-pages/man2/personality.2.html
[pts.4]: http://man7.org/linux/man-pages/man4/pts.4.html
[random.4]: http://man7.org/linux/man-pages/man4/random.4.html
[sysctl.8]: http://man7.org/linux/man-pages/man8/sysctl.8.html
[tty.4]: http://man7.org/linux/man-pages/man4/tty.4.html
[zero.4]: http://man7.org/linux/man-pages/man4/zero.4.html
[user-namespaces]: http://man7.org/linux/man-pages/man7/user_namespaces.7.html
[intel-rdt-cat-kernel-interface]: https://www.kernel.org/doc/Documentation/x86/intel_rdt_ui.txt
[time_namespaces.7]: https://man7.org/linux/man-pages/man7/time_namespaces.7.html
