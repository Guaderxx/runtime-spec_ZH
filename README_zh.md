# 开放容器倡议 OCI 运行时规范

[OCI][oci] 计划制定了操作系统进程和应用程序容器标准的规范。

规范可以在 [这里](spec_zh.md) 查看。


## 目录

有关该组运作的其他补充文档：

- [行为守则][code-of-conduct]
- [格式和惯例](style.md)
- [实现](implementations_zh.md)
- [版本](RELEASES.md)
- [章程][charter]


## 用例

为了向用户提供上下文，以下部分给出了规范每个部分的使用示例。


### 应用包构建工具

应用包构建工具可以创建将应用作为容器启动时所需的所有文件的 [包](bundle_zh.md) 文件夹。
该包包含了一个OCI [配置文件](config_zh.md)，构建工具可以在其中指定独立于主机的详细信息，例如 [要启动的可执行文件](config_zh.md#process) 以及特定于主机的设置，例如 [挂载](config_zh.md#mounts) 位置、 [钩子](config_zh.md#posix-platform-hooks) 路径、Linux [命名空间](config-linux_zh.md#namespaces) 和 [cgroups](config-linux_zh.md#control-groups)。
由于配置包括特定于主机的设置，因此在两个主机之间复制的应用程序包文件夹可能需要配置调整。


### 钩子开发人员

[钩子](config_zh.md#posix-platform-hooks)开发人员可以通过使用外部应用程序挂钩容器的生命周期来扩展符合 OCI 的运行时的功能。
示例用例包括复杂的网络配置、卷垃圾收集等。


### 运行时开发人员

运行时开发人员可以构建在特定平台上运行符合 OCI 的捆绑包和容器配置的运行时实现，其中包含底层操作系统和特定于主机的详细信息。


## 贡献

该规范的开发在 GitHub 上进行。
Issues 用于错误和可操作的项目，并且可以在 [邮件列表](#mailing-list) 上进行更长的讨论。

该规范和代码已根据 [LICENSE](./LICENSE) 文件 中的 Apache 2.0 许可证获得许可。


### 讨论你的设计

该项目欢迎提交，但请让每个人都知道您正在做什么。

在对本规范进行重大更改之前，请向 [邮件列表](#mailing-list) 发送邮件以讨论您计划做什么。
这让每个人都有机会验证设计，有助于防止重复工作，并确保想法适合。
它还保证在编写代码之前设计是合理的； GitHub PR 不是进行高层讨论的地方。

拼写错误和语法错误可能会直接进入拉取请求。
如有疑问，请从 [邮件列表](#mailing-list) 开始


### 会议

请参阅 [OCI 组织存储库自述文件](https://github.com/opencontainers/org#meetings) ，了解有关 OCI 贡献者和维护者会议安排的最新信息。您还可以找到所有先前会议的会议议程和会议记录的链接。

### Mailing List

你可以在 [Google Groups][dev-list] 上订阅并加入邮件列表。


### 聊天

OCI 讨论发生在以下聊天室中，这些聊天室全部桥接在一起：

- #general channel on [OCI Slack](https://opencontainers.org/community/overview/#chat)
- #opencontainers:matrix.org


### Git commit

#### Sign your work

...


#### Commit Style

...


[oci]: https://www.opencontainers.org
[charter]: https://github.com/opencontainers/tob/blob/master/CHARTER.md
[code-of-conduct]: https://github.com/opencontainers/org/blob/master/CODE_OF_CONDUCT.md
[dev-list]: https://groups.google.com/a/opencontainers.org/forum/#!forum/dev
