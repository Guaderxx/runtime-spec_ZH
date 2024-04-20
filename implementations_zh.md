# <a name="implementations" />Implementations

以下部分链接到相关项目，其中一些由 OCI 维护，另一些由外部组织维护。
如果您知道这里没有列出的任何相关项目，请提交拉取请求，添加指向该项目的链接。


## <a name="implementationsRuntimeContainer" />Runtime (Container)

* [alibaba/inclavare-containers][rune] - 用于机密计算的 Enclave OCI 运行时
* [containers/crun][crun] - C 实现的运行时
* [containers/youki][youki] - Rust 实现的运行时
* [opencontainers/runc][runc] - OCI 运行时的参考实现
* [projectatomic/bwrap-oci][bwrap-oci] - 将 OCI 规范文件转换为 [bubblewrap][bubblewrap] 命令行


## <a name="implementationsRuntimeVirtualMachine" />Runtime (Virtual Machine)

* [clearcontainers/runtime][cc-runtime] - Hypervisor-based OCI runtime utilising [virtcontainers][virtcontainers] by Intel®.
* [google/gvisor][gvisor] - gVisor is a user-space kernel, contains runsc to run sandboxed containers.
* [hyperhq/runv][runv] - Hypervisor-based runtime for OCI
* [kata-containers/runtime][kata-runtime] - Hypervisor-based OCI runtime combining technology from [clearcontainers/runtime][cc-runtime] and [hyperhq/runv][runv].


## <a name="implementationsTestingTools" />Testing & Tools

* [huawei-openlab/oct][oct] - 用于 OCI 配置和运行时的开放式容器测试框架
* [kunalkushwaha/octool][octool] - 配置检查器和验证器
* [opencontainers/runtime-tools][runtime-tools] - 配置生成器和运行时/包测试框架



[bubblewrap]: https://github.com/projectatomic/bubblewrap
[bwrap-oci]: https://github.com/projectatomic/bwrap-oci
[cc-runtime]: https://github.com/clearcontainers/runtime
[crun]: https://github.com/containers/crun
[gvisor]: https://github.com/google/gvisor
[kata-runtime]: https://github.com/kata-containers/runtime
[oct]: https://github.com/huawei-openlab/oct
[octool]: https://github.com/kunalkushwaha/octool
[runc]: https://github.com/opencontainers/runc
[rune]: https://github.com/alibaba/inclavare-containers
[runtime-tools]: https://github.com/opencontainers/runtime-tools
[runv]: https://github.com/hyperhq/runv
[virtcontainers]: https://github.com/containers/virtcontainers
[youki]: https://github.com/containers/youki
