# <a name="glossary" />Glossary

## <a name="glossaryBundle" />Bundle

提前编写、分发并用于为创建 [容器](#container) 并在其中启动进程的运行时提供种子的 [文件夹结构](bundle_zh.md) 。


## <a name="glossaryConfiguration" />Configuration

[捆绑包](#bundle) 中的 [`config.json`](config_zh.md) 文件定义了预期的 [容器](#container) 和容器进程。


## <a name="glossaryContainer" />Container

用于执行具有可配置隔离和资源限制的进程的环境。例如，命名空间、资源限制和挂载都是容器环境的一部分。


## <a name="glossaryContainerNamespace" />Container namespace

在 Linux 上，已 [配置进程](config_zh.md#process) 在其中执行的 [命名空间][namespaces.7] 。


## <a name="glossaryFeaturesDocument" />Features Structure

表示 [运行时](#runtime) [已实现功能](features_zh.md) 的 [JSON][] 结构。
与主机操作系统中功能的实际可用性无关。


## <a name="glossaryJson" />JSON

所有配置 [JSON][] MUST 以 [UTF-8][] 编码。 
JSON 对象 MUST NOT 包含重复的名称。 
JSON 对象中条目的顺序并不重要。


## <a name="glossaryRuntime" />Runtime

本规范的一个实现。
它从 [bundle](#bundle) 中读取 [配置文件](#configuration) ，使用该信息创建 [容器](#container) ，在容器内启动进程，并执行其他 [生命周期操作](runtime_zh.md) 。


## <a name="glossaryRuntimeCaller" />Runtime caller

直接或间接执行 [运行时](#runtime) 的外部程序。

直接调用者的示例包括 `containerd`、`CRI-O` 和`Podman`。
间接调用者的示例包括 `Docker/Moby` 和 `Kubernetes` 。

运行时调用者通常通过与 [runc][] 兼容的命令行接口执行运行时，但是，其交互接口目前超出了 OCI 运行时规范的范围。


## <a name="glossaryRuntimeNamespace" />Runtime namespace

在 Linux 上，[创建](config-linux_zh.md#namespaces) 新 [容器命名空间](#container-namespace) 以及访问某些已配置资源的命名空间。


[JSON]: https://tools.ietf.org/html/rfc8259
[UTF-8]: http://www.unicode.org/versions/Unicode8.0.0/ch03.pdf
[runc]: https://github.com/opencontainers/runc

[namespaces.7]: http://man7.org/linux/man-pages/man7/namespaces.7.html