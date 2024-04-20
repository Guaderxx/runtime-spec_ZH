# <a name="filesystemBundle" />Filesystem Bundle

## <a name="containerFormat" />Container Format

本节定义了一种将容器编码为 *文件系统包* 的格式--这是一组以特定方式组织的文件，包含所有必要的数据和元数据，任何兼容的运行时都可以对其执行所有标准操作。
另请参阅 [MacOS 应用程序捆绑包][macos_bundle] ，了解术语 *bundle* 的类似用法。

捆绑包的定义仅涉及容器及其配置数据如何存储在本地文件系统上，以便合规运行时可以使用它。

标准容器捆绑包包含加载和运行容器所需的全部信息。
其中包括以下工件：

1. `config.json`: 包含配置数据。
    这个 `REQUIRED` 文件必须位于捆绑包目录的根目录中，并且 `MUST` 命名为 config.json。
    有关更多详细信息，请参阅 [config.json](config_zh.md)。

2. 容器的根文件系统： [`root.path`](config_zh.md#root) 引用的文件夹（如果在 `config.json` 中设置了该属性）。

提供后，虽然这些工件必须全部存在于本地文件系统的单个文件夹中，但该文件夹本身并不是捆绑包的一部分。
换句话说，*bundle* 的 tar 存档将在存档的根文件夹中包含这些工件，而不是嵌套在顶级文件夹中。

[macos_bundle]: https://en.wikipedia.org/wiki/Bundle_%28macOS%29

