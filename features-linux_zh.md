# <a name="linuxFeatures" />Linux Features Structure

本文档描述了 [功能结构](features_zh.md) 中 [特定于 Linux 的部分](features_zh.md#platform-specific-features) 。


## <a name="linuxFeaturesNamespaces" />Namespaces

* **`namespaces`** (array of string, OPTIONAL) 命名空间的可识别名称，包括主机操作系统可能不支持的命名空间。
  运行时 MUST 将此数组中的元素识别为 [`config.json` 中 `linux.namespaces` 对象的类型](config-linux_zh.md#namespaces) 。


### Example

```json
"namespaces": [
    "cgroup",
    "ipc",
    "mount",
    "network",
    "pid",
    "user",
    "uts"
]
```


## <a name="linuxFeaturesCapabilities" />Capabilities

* **`capabilities`** (array of strings, OPTIONAL) 已识别的功能名称，包括主机操作系统可能不支持的功能。
  运行时 MUST 识别 [`config.json` 的 `process.capabilities` 对象](config_zh.md#linux-process) 中此数组中的元素。


### Example

```json
"capabilities": [
  "CAP_CHOWN",
  "CAP_DAC_OVERRIDE",
  "CAP_DAC_READ_SEARCH",
  "CAP_FOWNER",
  "CAP_FSETID",
  "CAP_KILL",
  "CAP_SETGID",
  "CAP_SETUID",
  "CAP_SETPCAP",
  "CAP_LINUX_IMMUTABLE",
  "CAP_NET_BIND_SERVICE",
  "CAP_NET_BROADCAST",
  "CAP_NET_ADMIN",
  "CAP_NET_RAW",
  "CAP_IPC_LOCK",
  "CAP_IPC_OWNER",
  "CAP_SYS_MODULE",
  "CAP_SYS_RAWIO",
  "CAP_SYS_CHROOT",
  "CAP_SYS_PTRACE",
  "CAP_SYS_PACCT",
  "CAP_SYS_ADMIN",
  "CAP_SYS_BOOT",
  "CAP_SYS_NICE",
  "CAP_SYS_RESOURCE",
  "CAP_SYS_TIME",
  "CAP_SYS_TTY_CONFIG",
  "CAP_MKNOD",
  "CAP_LEASE",
  "CAP_AUDIT_WRITE",
  "CAP_AUDIT_CONTROL",
  "CAP_SETFCAP",
  "CAP_MAC_OVERRIDE",
  "CAP_MAC_ADMIN",
  "CAP_SYSLOG",
  "CAP_WAKE_ALARM",
  "CAP_BLOCK_SUSPEND",
  "CAP_AUDIT_READ",
  "CAP_PERFMON",
  "CAP_BPF",
  "CAP_CHECKPOINT_RESTORE"
]
```


## <a name="linuxFeaturesCgroup" />Cgroup

**`cgroup`** (object, OPTIONAL) 表示cgroup 管理器的运行时的实现状态。
与主机操作系统的 cgroup 版本无关。

* **`v1`** (bool, OPTIONAL) 表示运行时是否支持cgroup v1。
* **`v2`** (bool, OPTIONAL) 表示运行时是否支持cgroup v2。
* **`systemd`** (bool, OPTIONAL) 表示运行时是否支持系统范围的 systemd cgroup 管理器。
* **`systemdUser`** (bool, OPTIONAL) 表示运行时是否支持用户范围的 systemd cgroup 管理器。
* **`rdma`** (bool, OPTIONAL) 表示运行时是否支持 RDMA cgroup 控制器。


### Example

```json
"cgroup": {
    "v1": true,
    "v2": true,
    "systemd": true,
    "systemdUser": true,
    "rdma": false
}
```


## <a name="linuxFeaturesSeccomp" />Seccomp

**`seccomp`** (object, OPTIONAL) 表示 seccomp 的运行时的实现状态。
与主机操作系统的内核版本无关。

* **`enabled`** (bool, OPTIONAL) 表示运行时是否支持 seccomp。
* **`actions`** (array of strings, OPTIONAL) seccomp 操作的可识别名称。
  运行时 MUST 识别 [`config.json` 中 `linux.seccomp` 对象的 `syscalls[].action` 属性](config-linux_zh.md#seccomp) 中此数组中的元素。
* **`operators`** (array of strings, OPTIONAL) seccomp 运算符的可识别名称。
  运行时 MUST 识别 [`config.json` 中 `linux.seccomp` 对象的 `syscalls[].args[].op` 属性](config-linux_zh.md#seccomp) 中此数组中的元素。
* **`archs`** (array of strings, OPTIONAL) seccomp 架构的可识别名称。
  运行时 MUST 识别 [`config.json` 中 `linux.seccomp` 对象的 `architectures` 属性](config-linux_zh.md#seccomp) 中此数组中的元素。
* **`knownFlags`** (array of strings, OPTIONAL) seccomp 标志的可识别名称。
  运行时 MUST 识别 [`config.json` 中 `linux.seccomp` 对象的 `flags` 属性](config-linux_zh.md#seccomp) 中此数组中的元素。
* **`supportedFlags`** (array of strings, OPTIONAL) seccomp 标志的可识别和受支持的名称。
  由于当前内核和/或 `libseccomp` 不支持某些标志，因此该列表可能是 `knownFlags` 的子集。
  运行时 MUST 识别并支持 [`config.json` 中 `linux.seccomp` 对象的 `flags` 属性](config-linux_zh.md#seccomp) 中此数组中的元素。


### Example

```json
"seccomp": {
    "enabled": true,
    "actions": [
        "SCMP_ACT_ALLOW",
        "SCMP_ACT_ERRNO",
        "SCMP_ACT_KILL",
        "SCMP_ACT_LOG",
        "SCMP_ACT_NOTIFY",
        "SCMP_ACT_TRACE",
        "SCMP_ACT_TRAP",
    ],
    "operators": [
        "SCMP_CMP_EQ",
        "SCMP_CMP_GE",
        "SCMP_CMP_GT",
        "SCMP_CMP_LE",
        "SCMP_CMP_LT",
        "SCMP_CMP_MASKED_EQ",
        "SCMP_CMP_NE"
    ],
    "archs": [
        "SCMP_ARCH_AARCH64",
        "SCMP_ARCH_ARM",
        "SCMP_ARCH_MIPS",
        "SCMP_ARCH_MIPS64",
        "SCMP_ARCH_MIPS64N32",
        "SCMP_ARCH_MIPSEL",
        "SCMP_ARCH_MIPSEL64",
        "SCMP_ARCH_MIPSEL64N32",
        "SCMP_ARCH_PPC",
        "SCMP_ARCH_PPC64",
        "SCMP_ARCH_PPC64LE",
        "SCMP_ARCH_S390",
        "SCMP_ARCH_S390X",
        "SCMP_ARCH_X32",
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X86_64"
    ],
    "knownFlags": [
        "SECCOMP_FILTER_FLAG_LOG"
    ],
    "supportedFlags": [
        "SECCOMP_FILTER_FLAG_LOG"
    ]
}
```


## <a name="linuxFeaturesApparmor" />AppArmor

**`apparmor`** (object, OPTIONAL) 代表AppArmor 的运行时的实现状态。
与主机操作系统上 AppArmor 的可用性无关。

* **`enabled`** (bool, OPTIONAL) 表示运行时是否支持AppArmor。


### Example

```json
"apparmor": {
    "enabled": true
}
```


## <a name="linuxFeaturesApparmor" />SELinux

**`selinux`** (object, OPTIONAL) 代表SELinux 的运行时的实现状态。
与主机操作系统上 SELinux 的可用性无关。

* **`enabled`** (bool, OPTIONAL) 代表运行时是否支持SELinux。


### Example

```json
"selinux": {
    "enabled": true
}
```


## <a name="linuxFeaturesIntelRdt" />Intel RDT

**`intelRdt`** (object, OPTIONAL) 表示英特尔 RDT 的运行时实现状态。
与主机操作系统上英特尔 RDT 的可用性无关。

* **`enabled`** (bool, OPTIONAL) 表示运行时是否支持 Intel RDT。


### Example

```json
"intelRdt": {
    "enabled": true
}
```


## <a name="linuxFeaturesMountExtensions" />MountExtensions

**`mountExtensions`** (object, OPTIONAL) 表示运行时是否支持某些挂载功能，无论这些功能在主机操作系统上是否可用。

* **`idmap`** (object, OPTIONAL) 表示运行时是否支持使用挂载的 `uidMappings` 和 `gidMappings` 属性进行 `idmap` 挂载。
  * **`enabled`** (bool, OPTIONAL) 表示运行时是否解析并尝试使用安装的 `uidMappings` 和 `gidMappings` 属性（如果提供）。
    请注意，运行时可能会部分实现 id 映射挂载支持（例如仅允许具有与容器的用户命名空间匹配的映射的挂载，或者仅允许 id 映射的绑定挂载）。
    在这种情况下，运行时仍然必须将此值设置为 `true`，以指示运行时识别 `uidMappings` 和 `gidMappings` 属性。


### Example

```json
"mountExtensioins": {
    "idmap": {
        "enabled": true
    }
}
```
