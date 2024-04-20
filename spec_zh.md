# <a name="openContainerInitiativeRuntimeSpecification" />Open Container Initiative Runtime Specification

开放容器倡议运行时规范

[OCI][oci] 计划制定了操作系统进程和应用程序容器标准的规范。


# <a name="ociRuntimeSpecAbstract" />Abstract

> 摘要

开放容器倡议运行时规范旨在指定容器的配置、执行环境和生命周期。

容器的配置被指定为受支持平台的 `config.json` ，并详细说明了支持创建容器的字段。
指定执行环境是为了确保在容器内运行的应用程序在运行时之间具有一致的环境以及为容器生命周期定义的常见操作。


# <a name="ociRuntimeSpecPlatforms" />Platforms

> 平台

本规范定义的平台有：


* `linux`: [runtime_zh.md](runtime_zh.md), [config_zh.md](config_zh.md), [features_zh.md](features_zh.md), [config-linux_zh.md](config-linux_zh.md), [runtime-linux_zh.md](runtime-linux_zh.md), and [features-linux_zh.md](features-linux_zh.md).
* `solaris`: [runtime_zh.md](runtime_zh.md), [config_zh.md](config_zh.md), [features_zh.md](features_zh.md), and [config-solaris.md](config-solaris.md).
* `windows`: [runtime_zh.md](runtime_zh.md), [config_zh.md](config_zh.md), [features_zh.md](features_zh.md), and [config-windows.md](config-windows.md).
* `vm`: [runtime_zh.md](runtime_zh.md), [config_zh.md](config_zh.md), [features_zh.md](features_zh.md), and [config-vm.md](config-vm.md).
* `zos`: [runtime_zh.md](runtime_zh.md), [config_zh.md](config_zh.md), [features_zh.md](features_zh.md), and [config-zos.md](config-zos.md).


# <a name="ociRuntimeSpecTOC" />Table of Contents

* [介绍](spec_zh.md)
  * [符号约定](#notational-conventions)
  * [容器原理](principles_zh.md)
* [文件系统包](bundle_zh.md)
* [运行时和生命周期](runtime_zh.md)
  * [Linux 特定的运行时和生命周期](runtime-linux_zh.md)
* [配置](config_zh.md)
  * [Linux 特定配置](config-linux_zh.md)
  * [Solaris 特定配置](config-solaris.md)
  * [Windows 特定配置](config-windows.md)
  * [虚拟机特定配置](config-vm.md)
  * [z/OS 特定配置](config-zos.md)
* [功能结构](features_zh.md)
  * [Linux 特定的功能结构](features-linux_zh.md)
* [词汇表](glossary_zh.md)


# <a name="ociRuntimeSpecNotationalConventions" />Notational Conventions

关键字 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" 的解释与 [RFC 2119][rfc2119] 中所述相同。

关键字 "unspecified", "undefined", and "implementation-defined" 应按照 [C99 标准的基本原理][c99-unspecified] 中的描述进行解释。

对于给定的 CPU 体系结构，如果某个实现无法满足其所实现 [平台](#platforms) 的一个或多个 "MUST"、"REQUIRED" 或 "SHALL" 要求，则该实现不符合要求。
对于给定的 CPU 架构，如果一个实施方案满足了它所实施 [平台](#platforms) 的所有 "MUST"、"REQUIRED" 和 "SHALL" 要求，那么这个实施方案就是符合要求的。

[c99-unspecified]: http://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf#page=18
[oci]: http://www.opencontainers.org
[rfc2119]: https://www.rfc-editor.org/rfc/rfc2119.html
