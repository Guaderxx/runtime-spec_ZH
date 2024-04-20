# <a name="features" />Features Structure

[运行时](glossary_zh.md#runtime) MAY 向 [运行时调用者](glossary_zh.md#runtime-caller) 提供有关其已实现功能的 JSON 结构。
这种 JSON 结构称为["特征结构"](glossary_zh.md#features-structure)。

功能结构与主机操作系统中功能的实际可用性无关。
因此，功能结构的内容应该根据运行时的编译时间而不是执行时间来确定。

功能结构中除 `ociVersionMin` 和 `ociVersionMax` 之外的所有属性都可以不存在或具有 `null` 值。
`null` 值 MUST NOT 与 `0`、`false`、`""`、`[]` 和 `{}` 等空值混淆。


## <a name="featuresSpecificationVersion" />Specification version

* **`ociVersionMin`** (string, REQUIRED) OCI 运行时规范 的最低公认版本。
  运行时 MUST 接受该值作为 [`config.json` 的 `ociVersion` 属性](config_zh.md#specification-version) 。
* **`ociVersionMax`** (string, REQUIRED) OCI 运行时规范 的最高公认版本。
  运行时 MUST 接受该值作为 [`config.json` 的 `ociVersion` 属性](config_zh.md#specification-version) 。
  该值 MUST NOT 小于 `ociVersionMin` 属性的值。
  功能结构 MUST NOT 包含此版本的 OCI 运行时规范中未定义的属性。


### Example

```json
{
    "ociVersionMin": "1.0.0",
    "ociVersionMax": "1.1.0"
}
```


## <a name="featuresHooks" />Hooks

* **`hooks`** (array of strings, OPTIONAL) [钩子](config_zh.md#posix-platform-hooks) 的可识别名称。
  运行时 MUST 支持此数组中的元素作为 [`config.json` 的 `hooks` 属性](config_zh.md#posix-platform-hooks) 。


### Example

```json
"hooks": [
  "prestart",
  "createRuntime",
  "createContainer",
  "startContainer",
  "poststart",
  "poststop"
]
```


## <a name="featuresMountOptions" />Mount Options

* **`mountOptions`** (array of strings, OPTIONAL) 挂载选项的可识别名称，包括主机操作系统可能不支持的选项。
  运行时 MUST 将此数组中的元素识别为 [`config.json` 中 `mounts` 对象的 `options`](config_zh.md#mounts) 。
  * Linux:  该数组 SHOULD NOT 包含作为 `const void *data` 传递给 [`mount(2)`][mount.2] 系统调用的特定于文件系统的挂载选项。


### Example

```json
"mountOptions": [
    "acl",
    "async",
    "atime",
    "bind",
    "defaults",
    "dev",
    "diratime",
    "dirsync",
    "exec",
    "iversion",
    "lazytime",
    "loud",
    "mand",
    "noacl",
    "noatime",
    "nodev",
    "nodiratime",
    "noexec",
    "noiversion",
    "nolazytime",
    "nomand",
    "norelatime",
    "nostrictatime",
    "nosuid",
    "nosymfollow",
    "private",
    "ratime",
    "rbind",
    "rdev",
    "rdiratime",
    "relatime",
    "remount",
    "rexec",
    "rnoatime",
    "rnodev",
    "rnodiratime",
    "rnoexec",
    "rnorelatime",
    "rnostrictatime",
    "rnosuid",
    "rnosymfollow",
    "ro",
    "rprivate",
    "rrelatime",
    "rro",
    "rrw",
    "rshared",
    "rslave",
    "rstrictatime",
    "rsuid",
    "rsymfollow",
    "runbindable",
    "rw",
    "shared",
    "silent",
    "slave",
    "strictatime",
    "suid",
    "symfollow",
    "sync",
    "tmpcopyup",
    "unbindable"
]
```


## <a name="featuresPlatformSpecificFeatures" />Platform-specific features

* **`linux`** (object, OPTIONAL) [Linux特有的功能](features-linux_zh.md)。
  如果运行时支持 `linux` 平台，则 MAY 设置此值。


## <a name="featuresAnnotations" />Annotations

**`annotations`** (object, OPTIONAL) 包含运行时的任意元数据。
该信息 MAY 是结构化的或非结构化的。
注解 MUST 是键值映射，遵循与 [`config.json` 的注解属性](config_zh.md#annotations)的键和值相同的约定。
但是，注解不需要包含 [`config.json` 的注解属性](config_zh.md#annotations)的可能值。
当前版本的规范没有提供枚举 [`config.json` 注解属性](config_zh.md#annotations)的可能值的方法。


### Example

```json
"annotations": {
  "org.opencontainers.runc.checkpoint.enabled": "true",
  "org.opencontainers.runc.version": "1.1.0"
}
```


## <a name="featuresPotentiallyUnsafeConfigAnnotations" />Unsafe annotations in `config.json`

**`potentiallyUnsafeConfigAnnotations`** (array of strings, OPTIONAL) 包含 [`config.json` 的注解属性](config.md#annotations)，这些值可能会更改运行时的行为。

以 "." 结尾的值被解释为注解的前缀。


### Example 

```json
"potentiallyUnsafeConfigAnnotations": [
    "com.example.foo.bar",
    "org.systemd.property."
]
```

上面的示例匹配 `com.example.foo.bar`、`org.systemd.property.ExecStartPre` 等。
该示例不匹配 `com.example.foo.bar.baz`。


# Example

这是一个供参考的完整示例。

```json
{
    "ociVersionMin": "1.0.0",
    "oviVersionMax": "1.1.0-rc.2",
    "hooks": [
        "prestart",
        "createRuntime",
        "createContainer",
        "startContainer",
        "poststart",
        "poststop"
    ],
    "mountOptions": [
        "async",
        "atime",
        "bind",
        "defaults",
        "dev",
        "diratime",
        "dirsync",
        "exec",
        "iversion",
        "lazytime",
        "loud",
        "mand",
        "noatime",
        "nodev",
        "nodiratime",
        "noexec",
        "noiversion",
        "nolazytime",
        "nomand",
        "norelatime",
        "nostrictatime",
        "nosuid",
        "nosymfollow",
        "private",
        "ratime",
        "rbind",
        "rdev",
        "rdiratime",
        "relatime",
        "remount",
        "rexec",
        "rnoatime",
        "rnodev",
        "rnodiratime",
        "rnoexec",
        "rnorelatime",
        "rnostrictatime",
        "rnosuid",
        "rnosymfollow",
        "ro",
        "rprivate",
        "rrelatime",
        "rro",
        "rrw",
        "rshared",
        "rslave",
        "rstrictatime",
        "rsuid",
        "rsymfollow",
        "runbindable",
        "rw",
        "shared",
        "silent",
        "slave",
        "strictatime",
        "suid",
        "symfollow",
        "sync",
        "tmpcopyup",
        "unbindable"
    ],
    "linux": {
        "namespaces": [
            "cgroup",
            "ipc",
            "mount",
            "network",
            "pid",
            "user",
            "uts"
        ],
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
        ],
        "cgroup": {
            "v1": true,
            "v2": true,
            "systemd": true,
            "systemdUser": true,
            "rdma": true
        },
        "seccomp": {
            "enabled": true,
            "actions": [
                "SCMP_ACT_ALLOW",
                "SCMP_ACT_ERRNO",
                "SCMP_ACT_KILL",
                "SCMP_ACT_KILL_PROCESS",
                "SCMP_ACT_KILL_THREAD",
                "SCMP_ACT_LOG",
                "SCMP_ACT_NOTIFY",
                "SCMP_ACT_TRACE",
                "SCMP_ACT_TRAP"
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
                "SCMP_ARCH_RISCV64",
                "SCMP_ARCH_S390",
                "SCMP_ARCH_S390X",
                "SCMP_ARCH_X32",
                "SCMP_ARCH_X86",
                "SCMP_ARCH_X86_64"
            ],
            "knownFlags": [
                "SECCOMP_FILTER_FLAG_TSYNC",
                "SECCOMP_FILTER_FLAG_SPEC_ALLOW",
                "SECCOMP_FILTER_FLAG_LOG"
            ],
            "supportedFlags": [
                "SECCOMP_FILTER_FLAG_TSYNC",
                "SECCOMP_FILTER_FLAG_SPEC_ALLOW",
                "SECCOMP_FILTER_FLAG_LOG"
            ]
        },
        "apparmor": {
            "enabled": true
        },
        "selinux": {
            "enabled": true
        },
        "interRdt": {
            "enabled": true
        }
    },
    "annotations": {
        "io.github.seccomp.libseccomp.version": "2.5.4",
        "org.opencontainers.runc.checkpoint.enabled": "true",
        "org.opencontainers.runc.commit": "v1.1.0-534-g26851168",
        "org.opencontainers.runc.version": "1.1.0+dev"
    }
}
```

[mount.2]: https://man7.org/linux/man-pages/man2/mount.2.html
