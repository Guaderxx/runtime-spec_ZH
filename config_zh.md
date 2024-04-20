# <a name="configuration" />Configuration

该配置文件包含对容器实施 [标准操作](runtime_zh.md#operations) 所需的元数据。
这包括要运行的进程、要注入的环境变量、要使用的沙箱功能等。

本文档中定义了规范模式，但 [`schema/config-schema.json`](schema/config-schema.json) 中有 JSON 模式， [`specs-go/config.go`](specs-go/config.go) 中有 Go 绑定。
特定于 [平台](spec_zh.md#platforms) 的配置架构在下面链接的 [特定于平台的文档](#platform-specific-configuration) 中定义。
对于仅为某些 [平台](spec_zh.md#platforms) 定义的属性，Go 属性具有列出这些协议的 `platform` 标签（例如： `platform:"linux,solaris"` ）。

下面是配置格式中定义的每个字段的详细描述，并指定了有效值。
特定于平台的字段被这样标识。
对于所有特定于平台的配置值，下面在 [特定于平台的配置](#platform-specific-configuration) 部分中定义的范围适用。


## <a name="configSpecificationVersion" />Specification version

* **`ociVersion`** (string, REQUIRED) MUST 是 [SemVer v2.0.0][semver-v2.0.0] 格式并指定该捆绑包所遵循的开放容器计划运行时规范的版本。
    OCI运行时规范遵循语义版本控制，并保留主要版本内的向前和向后兼容性。
    例如，如果配置符合本规范的 1.1 版，则它与支持本规范的任何 1.1 或更高版本的所有运行时兼容，但与支持 1.0 但不支持 1.1 的运行时不兼容。


### Example

```json
"ociVersion": "0.1.0"
```


## <a name="configRoot" />Root

**`root`** (object, OPTIONAL) 指定了容器的根文件系统。
在Windows上，对于 Windows Server 容器，这个字段是 REQUIRED。
对于 [Hyper-V 容器](config-windows.md#hyperv) ，这个字段 MUST NOT 被设置

对于所有其他平台，这个字段都是 REQUIRED。

* **`path`** (string, REQUIRED) 指定容器的根文件系统的路径。
  * 在Windows，`path` MUST 是[卷 GUID 路径][naming-a-volume]。
  * 在 POISX 平台， `path` 可以是绝对路径，也可以是相对于捆绑包的相对路径
      例如，捆绑包在 `/to/bundle` ，根文件系统在 `/to/bundle/rootfs` ， `path` 值可以是 `/to/bundle/rootfs` 或是 `rootfs` 。
      这个值按照传统 SHOULD 为 `rootfs`
    
    在字段声明的路径上 MUST 存在一个文件夹。

* **`readonly`** (bool, OPTIONAL) 如果是 `true` 那么容器内的根文件系统 MUST 是只读的，默认是 `false` 。
  * 在Windows，这个字段 MUST 被省略或是 `false`


### Example (POSIX platforms)

```json
"root": {
    "path": "rootfs",
    "readonly": true
}
```


### Example (Windows)

```json
"root": {
    "path": "\\\\?\\Volume{ec84d99e-3f02-11e7-ac6c-00155d7682cf}\\"
}
```


## <a name="configMounts" />Mounts

**`mounts`** (array of objects, OPTIONAL) 指定 [`root`](#root) 之外的附加挂载。
运行时 MUST 按列出的顺序挂载条目。
对于 Linux，参数如 [mount(2)][mount.2] 系统调用手册页中所述。
对于 Solaris，挂载条目对应于 [zonecfg(1M)][zonecfg.1m] 手册页中的 "fs" 资源。

* **`destination`** (string, REQUIRED) 挂载点的目标位置：容器内的路径。
  * Linux：这个值 SHOULD 为绝对路径。
    为了与旧工具和配置兼容，它可以是一个相对路径，在这种情况下，它 MUST 被解释为相对于 "/" 的路径。
    相对路径已被 **弃用** 。
  * Windows：这个值 MUST 是绝对路径。
    一个挂载目标 MUST NOT 嵌套在另一个挂载目标中（例如，`c:\foo` 和 `c:\foo\bar`）。
  * Solaris：这个值 MUST 是绝对路径。
    对应于 [zonecfg(1M)](zonecfg.1m) 中 fs 资源的 "dir"。
  * 所有其他平台：这个值 MUST 是绝对路径。
* **`source`** (string, OPTIONAL) 设备名称，也可用作挂载文件夹的绑定挂载或哑挂载的文件或文件夹名
  绑定挂载的路径值可以是绝对路径值，也可以是相对于捆绑包的路径值。
  如果选项中包含 `bind` 或 `rbind`，则该挂载为绑定挂载。
  * Windows：容器主机文件系统上的本地文件夹。不支持 UNC 路径和映射驱动器。
  * Solaris：对应于 [zonecfg(1M)](zonecfg.1m) 中 fs 资源的 "special"。
* **`options`** (array of strings, OPTIONAL) 要使用的文件系统的挂载选项。
  * Linux：见下面的 [Linux 挂载选项](#configLinuxMountOptions) 。
  * Solaris：对应于 [zonecfg(1M)](zonecfg.1m) 中 fs 资源的 "options"。
  * Windows：运行时 MUST 支持 `ro`，当给定 `ro` 时将文件系统挂载为只读。


### <a name="configLinuxMountOptions" />Linux mount options

运行时 MUST/SHOULD/MAY 实现了下面的 Linux 选项：

Option name | Requirement   | Description
--- | ---   | ---
`async`          | MUST        | [^1]
 `atime`          | MUST        | [^1]
 `bind`           | MUST        | Bind mount [^2]
 `defaults`       | MUST        | [^1]
 `dev`            | MUST        | [^1]
 `diratime`       | MUST        | [^1]
 `dirsync`        | MUST        | [^1]
 `exec`           | MUST        | [^1]
 `iversion`       | MUST        | [^1]
 `lazytime`       | MUST        | [^1]
 `loud`           | MUST        | [^1]
 `mand`           | MAY         | [^1] (Deprecated in kernel 5.15, util-linux 2.38)
 `noatime`        | MUST        | [^1]
 `nodev`          | MUST        | [^1]
 `nodiratime`     | MUST        | [^1]
 `noexec`         | MUST        | [^1]
 `noiversion`     | MUST        | [^1]
 `nolazytime`     | MUST        | [^1]
 `nomand`         | MAY         | [^1]
 `norelatime`     | MUST        | [^1]
 `nostrictatime`  | MUST        | [^1]
 `nosuid`         | MUST        | [^1]
 `nosymfollow`    | SHOULD      | [^1] (Introduced in kernel 5.10, util-linux 2.38)
 `private`        | MUST        | Bind mount propagation [^2]
 `ratime`         | SHOULD      | Recursive `atime` [^3]
 `rbind`          | MUST        | Recursive bind mount [^2]
 `rdev`           | SHOULD      | Recursive `dev` [^3]
 `rdiratime`      | SHOULD      | Recursive `diratime` [^3]
 `relatime`       | MUST        | [^1]
 `remount`        | MUST        | [^1]
 `rexec`          | SHOULD      | Recursive `dev` [^3]
 `rnoatime`       | SHOULD      | Recursive `noatime` [^3]
 `rnodiratime`    | SHOULD      | Recursive `nodiratime` [^3]
 `rnoexec`        | SHOULD      | Recursive `noexec` [^3]
 `rnorelatime`    | SHOULD      | Recursive `norelatime` [^3]
 `rnostrictatime` | SHOULD      | Recursive `nostrictatime` [^3]
 `rnosuid`        | SHOULD      | Recursive `nosuid` [^3]
 `rnosymfollow`   | SHOULD      | Recursive `nosymfollow` [^3]
 `ro`             | MUST        | [^1]
 `rprivate`       | MUST        | Bind mount propagation [^2]
 `rrelatime`    | SHOULD      | Recursive `relatime` [^3]
 `rro`            | SHOULD      | Recursive `ro` [^3]
 `rrw`            | SHOULD      | Recursive `rw` [^3]
 `rshared`        | MUST        | Bind mount propagation [^2]
 `rslave`         | MUST        | Bind mount propagation [^2]
 `rstrictatime`   | SHOULD      | Recursive `strictatime` [^3]
 `rsuid`          | SHOULD      | Recursive `suid` [^3]
 `rsymfollow`     | SHOULD      | Recursive `symfollow` [^3]
 `runbindable`    | MUST        | Bind mount propagation [^2]
 `rw`             | MUST        | [^1]
 `shared`         | MUST        | [^1]
 `silent`         | MUST        | [^1]
 `slave`          | MUST        | Bind mount propagation [^2]
 `strictatime`    | MUST        | [^1]
 `suid`           | MUST        | [^1]
 `symfollow`      | SHOULD      | Opposite of `nosymfollow`
 `sync`           | MUST        | [^1]
 `tmpcopyup`      | MAY         | copy up the contents to a tmpfs
 `unbindable`     | MUST        | Bind mount propagation [^2]
 `idmap`          | SHOULD      | 指示挂载 MUST 应用 idmapping。 此选项 SHOULD NOT 传递给底层 [`mount(2)`][mount.2] 调用。 如果为挂载指定了 `uidMappings` 或 `gidMappings` ，则运行时 MUST 将这些值用于挂载的映射。 如果未指定，运行时 MAY 使用容器的用户命名空间映射，否则 [MUST 返回错误](runtime_zh.md#errors)。如果没有 `uidMappings` 和 `gidMappings` 被指定且容器未使用用户命名空间， [MUST 返回错误](runtime_zh.md#errors)。 这 SHOULD 用[`mount_setattr(MOUNT_ATTR_IDMAP)`][mount_setattr.2] 实现, 自 Linux 5.12 起可用.
 `ridmap`         | SHOULD      | 表示挂载必须应用 idmapping，并且映射是递归应用的 [^3]. 该选项 SHOULD NOT 传递给底层 [`mount(2)`][mount.2] 调用. 如果挂载指定了 `uidMappings` 或 `gidMappings` , 运行时 MUST 使用那些值作为挂载的映射。如果它们未指定, 运行时 MAY 使用容器的用户命名空间映射, 否则 [MUST 返回错误](runtime_zh.md#errors) 。如果没有指定 `uidMappings` 和 `gidMappings` 且容器没有使用用户命名空间, [MUST 返回错误](runtime_zh.md#errors)。 这 SHOULD 使用 [`mount_setattr(MOUNT_ATTR_IDMAP)`][mount_setattr.2] 实现, 自 Linux 5.12 起可用。

[^1]: 对应于 [`mount(8)` (filesystem-independent)][mount.8-filesystem-independent].
[^2]: 对应于 [bind mounts and shared subtrees][mount-bind].
[^3]: These `AT_RECURSIVE` options need kernel 5.12 or later. See [`mount_setattr(2)`][mount_setattr.2]

"MUST" 选项对应于 [`mount(8)`][mount.8]。

运行时还 MAY 实现上表中未列出的自定义选项字符串。
如果自定义选项字符串已被 [`mount(8)`][mount.8] 识别，则运行时 SHOULD 遵循 [`mount(8)`][mount.8] 的行为。

运行时 SHOULD 将未知选项视为 [特定于文件系统的][mount.8-filesystem-specific]选项
并将它们作为逗号分隔的字符串传递给 [`mount(2)`][mount.2] 的第五个（`const void *data`）参数。


### Example (Windows)

```json
"mounts": [
    {
        "destination": "C:\\folder-inside-container",
        "source": "C:\\folder-on-host",
        "options": ["ro"]
    }
]
```


### <a name="configPOSIXMounts" />POSIX-platform Mounts

对于POSIX平台，`mounts` 结构有以下字段：

* **`type`** (string, OPTIONAL) 要挂载的文件系统类型。
  * Linux： */proc/filesystems* 中列出的内核支持的文件系统类型（例如，"minix"、"ext2"、"ext3"、"jfs"、"xfs"、"reiserfs"、"msdos"、"proc"、"nfs"、"iso9660"）。对于绑定挂载（当 `options` 包括 `bind` 或`rbind`时），类型是虚拟的，通常为 "none"（未在 */proc/filesystems* 中列出）。
  * Solaris：对应于 [zonecfg(1M)][zonecfg.1m] 中 fs 资源的 "type"。
* **`uidMappings`** (array of type LinuxIDMapping, OPTIONAL) 将 UIDs 从源文件系统转换到目标安装点的映射。
  SHOULD 使用 [`mount_setattr(MOUNT_ATR_IDMAP)`][mount_setattr.2]（自 Linux 5.12 起可用）来实现这一功能。
  如果指定了这一方法，`mounts` 结构的 `options` 字段应包含 `idmap` 或 `ridmap`，以指定映射是否应递归应用于 `rbind` 挂载，并确保旧版运行时不会默默忽略该字段。
  格式与 [用户命名空间映射](config-linux_zh.md#user-namespace-mappings) 相同。
  如果指定，它 MUST 与 `gidMappings` 一起指定。
* **`gidMappings`** (array of type LinuxIDMapping, OPTIONAL) 将 GIDs 从源文件系统转换到目标安装点的映射。
  SHOULD 使用 [`mount_setattr(MOUNT_ATR_IDMAP)`][mount_setattr.2]（自 Linux 5.12 起可用）来实现这一功能。
  如果指定了这一方法，`mounts` 结构的 `options` 字段应包含 `idmap` 或 `ridmap`，以指定映射是否应递归应用于 `rbind` 挂载，并确保旧版运行时不会默默忽略这一字段。
  更多详情，请参阅 `uidMappings`。
  如果指定，MUST 与 `uidMappings` 一起指定。


### Example (Linux)

```json
"mounts": [
    {
        "destination": "/tmp",
        "type": "tmpfs",
        "source": "tmpfs",
        "options": ["nosuid", "strictatime", "mode=755", "size=65536k"]
    },
    {
        "destination": "/data",
        "type": "none",
        "source": "/volumes/testing",
        "options": ["rbind","rw"]
    }
]
```


### Example (Solaris)

```json
"mounts": [
    {
        "destination": "/opt/local",
        "type": "lofs",
        "source": "/usr/local",
        "options": ["ro","nodevices"]
    },
    {
        "destination": "/opt/sfw",
        "type": "lofs",
        "source": "/opt/sfw"
    }
]
```


## <a name="configProcess" />Process

**`process`** (object, OPTIONAL) 指定容器进程。
调用 [`start`](runtime_zh.md#start) 时 REQUIRED 此属性。

* **`terminal`** (bool, OPTIONAL) 指定终端是否附加到进程，默认为 false。
  例如，如果在 Linux 上设置为 true，则会为进程分配一个伪终端对，并且在进程的 [标准流][stdin.3] 上复制伪终端 pty。
* **`consoleSize`** (object, OPTIONAL) 指定终端的控制台大小（以字符为单位）。
  如果终端为 `false` 或未设置，运行时 MUST 忽略 `consoleSize`。
  * **`height`** (uint, REQUIRED)
  * **`width`** (uint, REQUIRED)
* **`cwd`** (string, REQUIRED) 是将要为可执行文件设置的工作文件夹。
  该值 MUST 是绝对路径。
* **`env`** (array of strings, OPTIONAL) 与 [IEEE Std 1003.1-2008's `environ`][ieee-1003.1-2008-xbd-c8.1] 具有相同的语义。
* **`args`** (array of strings, OPTIONAL) 与 [IEEE Std 1003.1-2008 `execvp`'s *argv*][ieee-1003.1-2008-functions-exec] 有类似的语义。
  该规范扩展了 IEEE 标准，因为至少有一个条目是REQUIRED（非 Windows），并且该条目与 `execvp` 的 *file* 具有相同的语义。在 Windows 系统中，该字段为可选项，如果省略该字段，则 `commandLine` 为 REQUIRED。
* **`commandLine`** (string, OPTIONAL) 指定要在 Windows 上执行的完整命令行。
  这是在 Windows 上提供命令行的首选方式。如果省略，运行时将在对 Windows 进行系统调用之前退回到从 `args` 中转义和连接字段。


### <a name="configPOSIXProcess" />POSIX process

对于支持 POSIX rlimits 的系统（例如 Linux 和 Solaris），`process` 对象支持以下特定于进程的属性：

* **`rlimits`** (array of objects, OPTIONAL) 允许为进程设置资源限制。
    每个条目结构如下：

  * **`type`** (string, REQUIRED) 要限制的平台资源。
    * Linux：合法值在 [`getrlimit(2)`][getrlimit.2] 手册页中定义，例如 `RLIMIT_MSGQUEUE`。
    * Solaris：合法值在 [`getrlimit(3)`][getrlimit.3] 手册中定义，例如 `RLIMIT_CORE`

      对于任何无法映射到相关内核接口的值，运行时必须[产生错误](runtime_zh.md#errors)。
      对于 `rlimits` 中的每个条目，`type` 上的 [`getrlimit(3)`][getrlimit.3] MUST 成功。
      对于以下属性，`rlim` 指的是 [`getrlimit(3)`][getrlimit.3] 调用返回的状态。

    * **`soft`** (uint64, REQUIRED) 对相应资源强制执行的限制值。 
        `rlim.rlim_cur` MUST 与配置的值匹配。
  * **`hard`** (uint64, REQUIRED) 非特权进程可以设置的软限制的上限。 
    `rlim.rlim_max` MUST 与配置的值匹配。
    只有特权进程（例如具有 `CAP_SYS_RESOURCE` 功能的进程）才能提高硬限制。

    如果 `rlimits` 包含相同 `type` 的重复条目，运行时 MUST [产生错误](runtime_zh.md#errors)。


### <a name="configLinuxProcess" />Linux Process

对于基于 Linux 的系统，`process` 对象支持以下特定于进程的属性。

* **`apparmorProfile`** (string, OPTIONAL) 指定进程的 AppArmor 配置文件的名称。
    有关 AppArmor 的更多信息，请参阅 [AppArmor 文档][apparmor]。
* **`capabilities`** (object, OPTIONAL) 是一个包含数组的对象，该数组指定进程的功能集。
    有效值在 [capability(7)][capabilities.7] 手册页中定义，例如 `CAP_CHOWN` 。
    任何无法映射到相关内核接口或无法授予的值都 MUST 由运行时[记录为警告](runtime_zh.md#warnings) 。如果容器配置请求的能力无法被授予，例如运行时是在具有有限能力集的受限环境中运行，则运行时 SHOULD NOT 失败。
    `capabilities` 包含以下属性：

  * **`effective`** (array of strings, OPTIONAL) `effective` 字段是为进程保留的一系列有效能力。
  * **`bounding`** (array of strings, OPTIONAL) `bounding` 字段是为进程保留的边界功能的数组。
  * **`inheritable`** (array of strings, OPTIONAL) `inheritable` 字段是为进程保留的可继承功能的数组。
  * **`permitted`** (array of strings, OPTIONAL) `permitted` 字段是为进程保留的一系列允许的功能。
  * **`ambient`** (array of strings, OPTIONAL) `ambient` 字段是为该过程保留的一系列环境功能。
* **`noNewPrivileges`** (bool, OPTIONAL) 将 `noNewPrivileges` 设置为 `true` 可防止进程获得额外的权限。
    例如，内核文档中的 [`no_new_privs`][no-new-privs] 文章提供了有关如何在 Linux 上使用 `prctl` 系统调用来实现此目的的信息。
* **`oomScoreAdj`** *(int, OPTIONAL)* 调整 [proc 伪文件系统][proc_2] 中进程 `pid` 的 `[pid]/oom_score_adj` 中的 oom-killer 分数。
    如果设置了 `oomScoreAdj`，则运行时 MUST 将 `oom_score_adj` 设置为给定值。
    如果未设置 `oomScoreAdj`，则运行时 MUST NOT 更改 `oom_score_adj` 的值。

    这是一个针对每个进程的设置，其中，[`disableOOMKiller`](config-linux_zh.md#memory) 的作用域为内存 cgroup。
    有关这两个设置如何协同工作的更多信息，请参阅 [内存 cgroup 文档部分 10. OOM 控制][cgroup-v1-memory_2] 。
* **`scheduler`** (object, OPTIONAL) 是描述进程的调度程序属性的对象。`scheduler` 包含以下属性：

  * **`policy`** (string, REQUIRED) 代表调度策略。有效的值列表是：

    * `SCHED_OTHER`
    * `SCHED_FIFO`
    * `SCHED_RR`
    * `SCHED_BATCH`
    * `SCHED_ISO`
    * `SCHED_IDLE`
    * `SCHED_DEADLINE`

  * **`nice`** (int32, OPTIONAL) 是进程的 nice 值，影响其优先级。较低的 nice 值对应较高的优先级。如果未设置，运行时必须使用值 0。
  * **`priority`** (int32, OPTIONAL) 表示进程的静态优先级，由 `SCHED_FIFO` 和 `SCHED_RR` 等实时策略使用。如果未设置，运行时必须使用值 0。
  * **`flags`** (array of strings, OPTIONAL) 是一个字符串数组，代表调度标志。有效值列表如下：

    * `SCHED_FLAG_RESET_ON_FORK`
    * `SCHED_FLAG_RECLAIM`
    * `SCHED_FLAG_DL_OVERRUN`
    * `SCHED_FLAG_KEEP_POLICY`
    * `SCHED_FLAG_KEEP_PARAMS`
    * `SCHED_FLAG_UTIL_CLAMP_MIN`
    * `SCHED_FLAG_UTIL_CLAMP_MAX`

  * **`runtime`** (uint64, OPTIONAL) 表示在给定时间段内允许进程运行的时间量（以纳秒为单位），由截止时间调度程序使用。如果未设置，运行时必须使用值 0。
  * **`deadline`** (uint64, OPTIONAL) 表示进程完成其执行的绝对期限，由期限调度程序使用。如果未设置，运行时必须使用值 0。
  * **`period`** (uint64, OPTIONAL) 表示用于确定进程运行时间的时间段长度（以纳秒为单位），由截止时间调度程序使用。如果未设置，运行时必须使用值 0。
* **`selinuxLabel`** (string, OPTIONAL) 指定进程的 SELinux 标签。有关 SELinux 的更多信息，请参阅 [SELinux 文档][selinux] 。
* **`ioPriority`** (object, OPTIONAL) 配置进程组内容器进程的 I/O 优先级设置。
    I/O 优先级设置将自动应用于整个进程组，影响容器内的所有进程。
    可以使用以下属性：

  * **`class`** (string, REQUIRED) 指定 I/O 调度类。可能的值为 `IOPRIO_CLASS_RT`、`IOPRIO_CLASS_BE` 和 `IOPRIO_CLASS_IDLE`。
  * **`priority`** (int, REQUIRED) 指定类内的优先级。该值应该是一个介于 0（最高）到 7（最低）之间的整数。


### <a name="configUser" />User

进程的用户是特定于平台的结构，允许对进程以哪个用户身份运行进行特定控制。


#### <a name="configPOSIXUser" />POSIX-platform User

对于 POSIX 平台，`user` 结构具有以下字段：

* **`uid`** (int, REQUIRED) 指定 [容器命名空间](glossary_zh.md#container-namespace) 中的用户 ID。
* **`gid`** (int, REQUIRED) 指定 [容器命名空间](glossary_zh.md#container-namespace) 中的组 ID。
* **`umask`** (int, OPTIONAL) 指定用户的 [umask][umask.2] 。如果未指定，则不应更改调用进程的 umask。
* **`additionalGids`** (array of ints, OPTIONAL) 指定 [容器命名空间](glossary_zh.md#container-namespace)中要添加到进程的其他组 ID。

*注意：uid 和 gid 的符号名称，例如分别是 uname 和 gname，留给上层来派生（即 `/etc/passwd` 解析、NSS 等）*


### Example (Linux)

```json
"process": {
    "terminal": true,
    "consoleSize": {
        "height": 25,
        "width": 80
    },
    "user": {
        "uid": 1,
        "gid": 1,
        "umask": 63,
        "additionalGids": [5, 6]
    },
    "env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "TERM=xterm"
    ],
    "cwd": "/root",
    "args": [
        "sh"
    ],
    "apparmorProfile": "acme_secure_profile",
    "selinuxLabel": "system_u:system_r:svirt_lxc_net_t:s0:c124,c675",
    "ioPriority": {
        "class": "IOPRIO_CLASS_IDLE",
        "priority": 4
    },
    "noNewPrivileges": true,
    "capabilities": {
        "bounding": [
            "CAP_AUDIT_WRITE",
            "CAP_KILL",
            "CAP_NET_BIND_SERVICE"
        ],
       "permitted": [
            "CAP_AUDIT_WRITE",
            "CAP_KILL",
            "CAP_NET_BIND_SERVICE"
        ],
       "inheritable": [
            "CAP_AUDIT_WRITE",
            "CAP_KILL",
            "CAP_NET_BIND_SERVICE"
        ],
        "effective": [
            "CAP_AUDIT_WRITE",
            "CAP_KILL"
        ],
        "ambient": [
            "CAP_NET_BIND_SERVICE"
        ]
    },
    "rlimits": [
        {
            "type": "RLIMIT_NOFILE",
            "hard": 1024,
            "soft": 1024
        }
    ]
}
```


### Example (Solaris)

```json
"process": {
    "terminal": true,
    "consoleSize": {
        "height": 25,
        "width": 80
    },
    "user": {
        "uid": 1,
        "gid": 1,
        "umask": 7,
        "additionalGids": [2, 8]
    },
    "env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "TERM=xterm"
    ],
    "cwd": "/root",
    "args": [
        "/usr/bin/bash"
    ]
}
```


#### <a name="configWindowsUser" />Windows User

对于基于 Windows 的系统，用户结构包含以下字段：

* **`username`** (string, OPTIONAL) 指定进程的用户名。


### Example (Windows)

```json
"process": {
    "terminal": true,
    "user": {
        "username": "containeradministrator"
    },
    "env": [
        "VARIABLE=1"
    ],
    "cwd": "c:\\foo",
    "args": [
        "someapp.exe",
    ]
}
```


## <a name="configHostname" />Hostname

* **`hostname`** (string, OPTIONAL) 指定容器内运行的进程所看到的容器主机名。
    例如，在 Linux 上，这将更改 [容器](glossary_zh.md#container-namespace) [UTS 命名空间][uts-namespace.7] 中的主机名。
    根据您的 [命名空间配置](config-linux_zh.md#namespaces)，容器 UTS 命名空间可能是 [运行时](glossary_zh.md#runtime-namespace) [UTS 命名空间][uts-namespace.7]。


### Example

```json
"hostname": "mrsdalloway"
```


## <a name="configDomainname" />Domainname

* **`domainname`** (string, OPTIONAL) 指定容器内运行的进程所看到的容器域名。
    例如，在 Linux 上，这将更改 [容器](glossary_zh.md#container-namespace) [UTS 命名空间][uts-namespace.7] 中的域名。
    根据您的 [命名空间配置](config-linux_zh.md#namespaces) ，容器 UTS 命名空间可能是 [运行时](glossary_zh.md#runtime-namespace) [UTS 命名空间][uts-namespace.7] 。


### Example

```json
"domainname": "foobarbaz.test"
```


## <a name="configPlatformSpecificConfiguration" />Platform-specific configuration

* **`linux`** (object, OPTIONAL) [Linux 特定的配置](config-linux_zh.md) 。
  如果此规范的目标平台是 `Linux`，则 MAY 设置此值。
* **`windows`** (object, OPTIONAL) [Windows 特定的配置](config-windows.md) 。
    如果此规范的目标平台是 `windows`，则 MUST 设置此值。
* **`solaris`** (object, OPTIONAL) [Solaris 特定的配置](config-solaris.md)。
    如果此规范的目标平台是 `solaris`，则 MAY 设置此值。
* **`vm`** (object, OPTIONAL) [虚拟机特定的配置](config-vm.md)。
    如果此规范的目标平台和体系结构支持硬件虚拟化，则 MAY 设置此值。
* **`zos`** (object, OPTIONAL) [z/OS 特定的配置](config-zos.md).
    如果此规范的目标平台是 `zos`，则 MAY 设置此值。


### Example (Linux)

```json
{
    "linux": {
        "namespaces": [
            {
                "type": "pid"
            }
        ]
    }
}
```


## <a name="configHooks" />POSIX-platform Hooks

对于 POSIX 平台，配置结构支持用于配置与容器 [生命周期](runtime_zh.md#lifecycle) 相关的自定义操作的钩子。

* **`hooks`** (object, OPTIONAL) MAY 包含以下任何属性：
  * **`prestart`** (array of objects, OPTIONAL, **DEPRECATED**) 是一个 [`prestart` 钩子](#prestart) 的数组。
    * 数组中的条目包含以下属性：
      * **`path`** (string, REQUIRED) 与 [IEEE Std 1003.1-2008 `execv` 的 *path*][ieee-1003.1-2008-functions-exec] 具有相似的语义。
        该规范扩展了 IEEE 标准，该 `path` MUST 是绝对的。
      * **`args`** (array of strings, OPTIONAL) 和 [IEEE Std 1003.1-2008 `execv` 的 *argv*][ieee-1003.1-2008-functions-exec] 具有相似的语义。
      * **`env`** (array of strings, OPTIONAL) 和 [IEEE Std 1003.1-2008 的 `environ`][ieee-1003.1-2008-xbd-c8.1] 具有相似的语义。
      * **`timeout`** (int, OPTIONAL) 是中止钩子前的秒数。
        如果设置， `timeout` MUST 大于零。
    * `path` 的值 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
    * `prestart` 钩子 MUST 在[运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。
  * **`createRuntime`** (array of objects, OPTIONAL) 是 [`createRuntime` 钩子](#createRuntime-hooks) 的数组。
    * 数组中的条目包含以下属性（与已废弃的 `prestart` 钩子中的条目相同）：
      * **`path`** (string, REQUIRED) 与 [IEEE Std 1003.1-2008 `execv` 的 *path*][ieee-1003.1-2008-functions-exec] 具有相似的语义。
        该规范扩展了 IEEE 标准，该 `path` MUST 是绝对的。
      * **`args`** (array of strings, OPTIONAL) 和 [IEEE Std 1003.1-2008 `execv` 的 *argv*][ieee-1003.1-2008-functions-exec] 具有相似的语义。
      * **`env`** (array of strings, OPTIONAL) 和 [IEEE Std 1003.1-2008 的 `environ`][ieee-1003.1-2008-xbd-c8.1] 具有相似的语义。
      * **`timeout`** (int, OPTIONAL) 是中止钩子前的秒数。
        如果设置， `timeout` MUST 大于零。
    * `path` 的值 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
    * `createRuntime` 钩子 MUST 在[运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。
  * **`createContainer`** (array of objects, OPTIONAL) 是一个 [`createContainer` 钩子](#createContainer-hooks) 数组。
    * 数组中的条目与 `createRuntime` 条目具有相同的架构。
    * `path` 的值必须在[运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
    * `createContainer` 钩子必须在 [容器命名空间](glossary_zh.md#container-namespace) 中执行。
  * **`startContainer`** (array of objects, OPTIONAL) 是一个 [`startContainer` 钩子](#startContainer-hooks) 数组。
    * 数组中的条目与 `createRuntime` 条目具有相同的架构。
    * `path` 的值必须在[容器命名空间](glossary_zh.md#container-namespace)中解析。
    * `startContainer` 钩子必须在 [容器命名空间](glossary_zh.md#container-namespace) 中执行。
  * **`poststart`** (array of objects, OPTIONAL) 是一个 [`poststart` 钩子](#poststart) 数组。
    * 数组中的条目与 `createRuntime` 条目具有相同的架构。
    * `path` 的值必须在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
    * `poststart` 钩子必须在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。
  * **`poststop`** (array of objects, OPTIONAL) 是一个 [`poststop` 钩子](#poststop) 数组。
    * 数组中的条目与 `createRuntime` 条目具有相同的架构。
    * `path` 的值 MUST 在[运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
    * `poststart` 钩子 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。

钩子允许用户指定在各种生命周期事件之前或之后运行的程序。
钩子必须按列出的顺序调用。
容器的 [状态](runtime_zh.md#state) MUST 通过标准输入传递给钩子，以便它们可以执行适合容器当前状态的工作。


### <a name="configHooksPrestart" />Prestart

必须在创建运行时环境之后（根据 `config.json` 中的配置）但在执行 `pivot_root` 或任何等效操作之前调用 *prestart* 钩子作为[`create`](runtime_zh.md#create)操作的一部分。
例如，在 Linux 上，它们在创建容器名称空间后被调用，因此它们提供了自定义容器的机会（例如，可以在此钩子中指定网络名称空间）。
`prestart` 钩子必须在 `createRuntime` 钩子之前调用。

注意：`prestart` 钩子已被弃用，取而代之的是 `createRuntime`、`createContainer` 和 `startContainer` 钩子，这些钩子允许在创建和启动阶段进行更精细的钩子控制。

`prestart` 钩子的路径 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
`prestart` 钩子 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。


### <a name="configHooksCreateRuntime" />CreateRuntime Hooks

`createRuntime` 钩子必须作为 [`create`](runtime_zh.md#create) 操作的一部分在创建运行时环境之后（根据 `config.json` 中的配置）但在执行 `pivot_root` 或任何等效操作之前调用。

`createRuntime` 钩子的路径 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
`createRuntime` 钩子 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。

例如，在 Linux 上，它们在创建容器名称空间后被调用，因此它们提供了自定义容器的机会（例如，可以在此钩子中指定网络名称空间）。

`createRuntime` 钩子的定义目前尚未指定，钩子作者应该只期望运行时已创建挂载命名空间并执行挂载操作。其他操作（例如 `cgroup` 和 `SELinux/AppArmor` 标签）可能尚未由运行时执行。


### <a name="configHooksCreateContainer" />CreateContainer Hooks

`createContainer` 钩子必须作为[`create`](runtime_zh.md#create) 操作的一部分在创建运行时环境之后（根据 `config.json` 中的配置）但在执行 `pivot_root` 或任何等效操作之前调用。
`createContainer` 钩子 MUST 在 `createRuntime` 钩子之后调用。

`createContainer` 钩子的路径 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
`createContainer` 钩子 MUST 在 [容器命名空间](glossary_zh.md#container-namespace) 中执行。

例如，在 `Linux` 上，这会在执行 `pivot_root` 操作之前但在创建和设置挂载命名空间之后发生。

`createContainer` 钩子的定义目前尚未明确，钩子作者应该只期望在运行时安装挂载命名空间和不同的挂载。其他操作（例如 `cgroup` 和 `SELinux/AppArmor` 标签）可能尚未由运行时执行。


### <a name="configHooksStartContainer" />StartContainer Hooks

作为 [`start`](runtime_zh.md#start) 操作的一部分，MUST 在 [执行用户指定的进程之前](runtime_zh.md#lifecycle) 调用 `startContainer` 钩子。
此钩子可用于在容器中执行某些操作，例如在容器进程生成之前在 `linux` 上运行 `ldconfig` 二进制文件。

`startContainer` 钩子的路径 MUST 在 [容器命名空间](glossary_zh.md#container-namespace) 中解析。
`startContainer` 钩子 MUST 在 [容器命名空间](glossary_zh.md#container-namespace) 中执行。


### <a name="configHooksPoststart" />Poststart

`poststart` 钩子 MUST 在 [执行用户指定的进程之后](runtime_zh.md#lifecycle) 但[`start`](runtime_zh.md#start) 操作返回之前调用。
例如，这个钩子可以通知用户容器进程已生成。

`poststart` 钩子的路径 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
`poststart` 钩子 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。


### <a name="configHooksPoststop" />Poststop

`poststop` 钩子 MUST 在 [容器被删除之后](runtime_zh.md#lifecycle)、[`delete`](runtime_zh.md#delete)操作返回之前调用。
清理或调试功能就是此类钩子的示例。

`poststop` 钩子的路径 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中解析。
`poststop` 钩子 MUST 在 [运行时命名空间](glossary_zh.md#runtime-namespace) 中执行。


### Summary

有关钩子及其调用时间的摘要，请参阅下表：

 名称 | 命名空间 | 何时 
 --- | --- | --- 
 `prestart` （废弃） | runtime | 在 start 操作调用后，但在用户指定的程序执行前 
 `createRuntime` | runtime   | 在 create 操作期间，在创建运行时环境之后且在 pivot root 或任何等效操作之前。
 `createContainer` | container | 在 create 操作期间，在创建运行时环境之后且在 pivot root 或任何等效操作之前。
 `startContainer`  | container | 在调用 start 操作之后，但在执行用户指定的程序命令前。
 `poststart`   | runtime   | 在执行用户指定的进程之后但在 start 操作返回前。
 `poststop`    | runtime   | 在删除容器之后，但在 delete 操作返回之前。


### Example

```json
"hooks": {
    "prestart": [
        {
            "path": "/usr/bin/fix-mounts",
            "args": ["fix-mounts", "arg1", "arg2"],
            "env": ["key1=value1"]
        },
        {
            "path": "/usr/bin/setup-network"
        }
    ],
    "createRuntime": [
        {
            "path": "/usr/bin/fix-mounts",
            "args": ["fix-mounts", "arg1", "arg2"],
            "env": ["key1=value1"]
        },
        {
            "path": "/usr/bin/setup-network"
        }
    ],
    "createContainer": [
        {
            "path": "/usr/bin/mount-hook",
            "args": ["-mount", "arg1", "arg2"],
            "env": ["key1=value1"]
        }
    ],
    "startContainer": [
        {
            "path": "/usr/bin/refresh-ldcache"
        }
    ],
    "poststart": [
        {
            "path": "/usr/bin/notify-start",
            "timeout": 5
        }
    ],
    "poststop": [
        {
            "path": "/usr/sbin/cleanup.sh",
            "args": ["cleanup.sh", "-f"]
        }
    ]
}
```


## <a name="configAnnotations" />Annotations

**`annotations`** (object, OPTIONAL) 包含容器的任意元数据。
该信息 MAY 是结构化的或非结构化的。
注解 MUST 是键值对。
如果没有注解，则此属性 MAY 不存在或为空对象。

键 MUST 是字符串。
键 MUST NOT 是空字符串。
键 SHOULD 使用反向域表示法命名 - 例如 `com.example.myKey`。

键的 `org.opencontainers` 命名空间保留供本规范使用，使用此命名空间中的键的注解 MUST 如本节中所述。
MAY 使用 `org.opencontainers` 命名空间中的以下键：

 Key | Definition 
--- | ---
 `org.opencontainers.image.os` | 表示容器镜像构建后运行的操作系统。注解值 MUST 是 [OCI 镜像规范][oci-image-config-properties] 中定义的 `os` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion] 使用。
 `org.opencontainers.image.os.version` | 表示容器镜像所针对的操作系统版本。注解值 MUST 是 [OCI 镜像规范][oci-image-config-properties] 中定义的 `os.version` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion] 使用。
 `org.opencontainers.image.os.features` | 表示容器镜像所需的强制性操作系统功能。注解值 MUST 具有 [OCI 镜像规范][oci-image-config-properties] 中定义的 `os.features` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion]使用。
 `org.opencontainers.image.architecture` | 表示容器镜像中的二进制文件构建用于运行的体系结构。注解值 MUST 具有 [OCI 镜像规范][oci-image-config-properties] 中定义的 `architecture` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion] 使用。
 `org.opencontainers.image.variant`  | 表示容器镜像中的二进制文件是为在其上运行而构建的体系结构的变体。注解值 MUST 具有 [OCI 镜像规范][oci-image-config-properties]中定义的 `variant` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion]使用。
 `org.opencontainers.image.author` | 表示容器镜像的作者。注解值 MUST 具有 [OCI 镜像规范][oci-image-config-properties]中定义的 `author` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion] 使用。
 `org.opencontainers.image.created` | 表示创建容器镜像的日期和时间。注解值 MUST 具有 [OCIimage 规范][oci-image-config-properties] 中定义的 `created` 属性的有效值。该注解 SHOULD 仅根据 [OCI 图像规范的运行时转换规范][oci-image-conversion] 使用。
 `org.opencontainers.image.stopSignal` | 表示容器运行时 SHOULD 发送的 [杀死容器](runtime_zh.md#kill) 的信号。注解值 MUST 具有 [OCI 镜像规范][oci-image-config-properties] 中定义的 `config.StopSignal` 属性的有效值。该注解 SHOULD 仅根据 [OCI 镜像规范的运行时转换规范][oci-image-conversion] 使用。

上表中未指定的 `org.opencontainers` 命名空间中的所有其他键均被保留，并且 MUST NOT 在后续规范中使用。
运行时 MUST 像处理任何其他 [未知属性](#extensibility) 一样处理未知的注解键。

值 MUST 是字符串。
值 MAY 是空字符串。

```json
"annotations": {
    "com.example.gpu-cores": "2"
}
```


## <a name="configExtensibility" />Extensibility

运行时 MAY [记录](runtime_zh.md#warnings) 未知属性，但 MUST 忽略它们。
这包括在遇到未知属性时不会 [生成错误](runtime_zh.md#errors) 。


## Valid values

当遇到无效或不支持的值时，运行时 MUST 生成错误。
除非明确要求支持有效值，否则运行时 MAY 选择它支持的有效值的子集。


## Configuration Schema Example

这是一个完整的 `config.json` 示例供参考。

```json
{
    "ociVersion": "1.0.1",
    "process": {
        "terminal": true,
        "user": {
            "uid": 1,
            "gid": 1,
            "additionalGids": [
                5,
                6
            ]
        },
        "args": [
            "sh"
        ],
        "env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "TERM=xterm"
        ],
        "cwd": "/",
        "capabilities": {
            "bounding": [
                "CAP_AUDIT_WRITE",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE"
            ],
            "permitted": [
                "CAP_AUDIT_WRITE",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE"
            ],
            "inheritable": [
                "CAP_AUDIT_WRITE",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE"
            ],
            "effective": [
                "CAP_AUDIT_WRITE",
                "CAP_KILL"
            ],
            "ambient": [
                "CAP_NET_BIND_SERVICE"
            ]
        },
        "rlimits": [
            {
                "type": "RLIMIT_CORE",
                "hard": 1024,
                "soft": 1024,
            },
            {
                "type": "RLIMIT_NOFILE",
                "hard": 1024,
                "soft": 1024
            }
        ],
        "apparmorProfile": "acme_secure_profile",
        "oomScoreAdj": 100,
        "selinuxLabel": "system_u:system_r:svirt_lxc_net_t:s0:c124,c675",
        "ioPriority": {
            "class": "IOPRIO_CLASS_IDLE",
            "priority": 4
        },
        "noNewPrivileges": true
    },
    "root": {
        "path": "rootfs",
        "readonly": true
    },
    "hostname": "slartibartfast",
    "mounts": [
        {
            "destination": "/proc",
            "type": "proc",
            "source": "proc"
        },
        {
            "destination": "/dev",
            "type": "tmpfs",
            "source": "tmpfs",
            "options": [
                "nosuid",
                "strictatime",
                "mode=755",
                "size=65536k"
            ]
        },
        {
            "destination": "/dev/pts",
            "type": "devpts",
            "source": "devpts",
            "options": [
                "nosuid",
                "noexec",
                "newinstance",
                "ptmxmode=0666",
                "mode=0620",
                "gid=5"
            ]
        },
        {
            "destination": "/dev/shm",
            "type": "tmpfs",
            "source": "shm",
            "options": [
                "nosuid",
                "noexec",
                "nodev",
                "mode=1777",
                "size=65536k"
            ]
        },
        {
            "destination": "/dev/mqueue",
            "type": "mqueue",
            "source": "mqueue",
            "options": [
                "mosuid",
                "noexec",
                "nodev"
            ]
        },
        {
            "destination": "/sys",
            "type": "sysfs",
            "source": "sysfs",
            "options": [
                "nosuid",
                "noexec",
                "nodev"
            ]
        },
        {
            "destination": "/sys/fs/cgroup",
            "type": "cgroup",
            "source": "cgroup",
            "options": [
                "nosuid",
                "noexec",
                "nodev",
                "relatime",
                "ro"
            ]
        }
    ],
    "hooks": {
        "prestart": [
            {
                "path": "/usr/bin/fix-mounts",
                "args": [
                    "fix-mounts",
                    "arg1",
                    "arg2"
                ],
                "env": [
                    "key1=value1"
                ]
            },
            {
                "path": "/usr/bin/setup-network"
            }
        ],
        "poststart": [
            {
                "path": "/usr/bin/notify-start",
                "timeout": 5
            }
        ],
        "poststop": [
            {
                "path": "/usr/sbin/cleanup.sh",
                "args": [
                    "cleanup.sh",
                    "-f"
                ]
            }
        ]
    },
    "linux": {
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
                "gid" 0
            }
        ],
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
        ],
        "sysctl": {
            "net.ipv4.ip_forward": "1",
            "net.core.somaxcomm": "256"
        },
        "cgroupsPath": "/myRuntime/myContainer",
        "resources": {
            "network": {
                "classID": 1048577,
                "priorities": [
                    {
                        "name": "eth0",
                        "priority": 500
                    },
                    {
                        "name": "eth1",
                        "priority": 1000
                    }
                ]
            },
            "pids": {
                "limit": 32771
            },
            "hugepageLimits": [
                {
                    "pageSize": "2MB",
                    "limit": 9223372036854772000
                },
                {
                    "pageSize": "64KB",
                    "limit": 1000000
                }
            ],
            "memory": {
                "limit": 536870912,
                "reservation": 536870912,
                "swap": 536870912,
                "kernel": -1,
                "kernelTCP": -1,
                "swappiness": 0,
                "disableOOMKiller": false
            },
            "cpu": {
                "shares": 1024,
                "quota": 1000000,
                "period": 500000,
                "realtimeRuntime": 950000,
                "realtimePeriod": 1000000,
                "cpus": "2-3",
                "idle": 1,
                "mems": "0-7"
            },
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
            ],
            "blockIO": {
                "weight": 10,
                "leadWeight": 10,
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
        },
        "rootfsPropagation": "slave",
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
        },
        "timeOffsets": {
            "monotonic": {
                "secs": 172800,
                "nanosecs": 0
            },
            "boottime": {
                "secs": 604800,
                "nanosecs": 0
            }
        },
        "namespaces": [
            {
                "type": "pid"
            },
            {
                "type": "network"
            },
            {
                "type": "ipc"
            },
            {
                "type": "uts"
            },
            {
                "type": "mount"
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
        ],
        "maskedPaths": [
            "/proc/kcore",
            "/proc/latency_stats",
            "/proc/timer_stats",
            "/proc/sched_debug"
        ],
        "readonlyPaths": [
            "/proc/asound",
            "/proc/bus",
            "/proc/fs",
            "/proc/irq",
            "/proc/sys",
            "/proc/sysrq-trigger"
        ],
        "mountLabel": "system_u:object_r:svirt_sandbox_file_t:s0:c715,c811"
    },
    "annotations": {
        "com.example.key1": "value1",
        "com.example.key2": "value2"
    }
}
```

[apparmor]: https://wiki.ubuntu.com/AppArmor
[cgroup-v1-memory_2]: https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt
[selinux]:http://selinuxproject.org/page/Main_Page
[no-new-privs]: https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt
[proc_2]: https://www.kernel.org/doc/Documentation/filesystems/proc.txt
[umask.2]: http://pubs.opengroup.org/onlinepubs/009695399/functions/umask.html
[semver-v2.0.0]: http://semver.org/spec/v2.0.0.html
[ieee-1003.1-2008-xbd-c8.1]: http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html#tag_08_01
[ieee-1003.1-2008-functions-exec]: http://pubs.opengroup.org/onlinepubs/9699919799/functions/exec.html
[naming-a-volume]: https://aka.ms/nb3hqb
[oci-image-config-properties]: https://github.com/opencontainers/image-spec/blob/v1.1.0-rc2/config.md#properties
[oci-image-conversion]: https://github.com/opencontainers/image-spec/blob/v1.1.0-rc2/conversion.md

[capabilities.7]: http://man7.org/linux/man-pages/man7/capabilities.7.html
[mount.2]: http://man7.org/linux/man-pages/man2/mount.2.html
[mount.8]: http://man7.org/linux/man-pages/man8/mount.8.html
[mount.8-filesystem-independent]: http://man7.org/linux/man-pages/man8/mount.8.html#FILESYSTEM-INDEPENDENT_MOUNT_OPTIONS
[mount.8-filesystem-specific]: http://man7.org/linux/man-pages/man8/mount.8.html#FILESYSTEM-SPECIFIC_MOUNT_OPTIONS
[mount_setattr.2]: http://man7.org/linux/man-pages/man2/mount_setattr.2.html
[mount-bind]: https://docs.kernel.org/filesystems/sharedsubtree.html
[getrlimit.2]: http://man7.org/linux/man-pages/man2/getrlimit.2.html
[getrlimit.3]: http://pubs.opengroup.org/onlinepubs/9699919799/functions/getrlimit.html
[stdin.3]: http://man7.org/linux/man-pages/man3/stdin.3.html
[uts-namespace.7]: http://man7.org/linux/man-pages/man7/namespaces.7.html
[zonecfg.1m]: http://docs.oracle.com/cd/E86824_01/html/E54764/zonecfg-1m.html
