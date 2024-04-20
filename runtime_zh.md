# <a name="runtimeAndLifecycle" />Runtime and Lifecycle


## <a name="runtimeScopeContainer" />Scope of a Container

使用运行时创建容器的实体 MUST 能够针对同一容器使用本规范中定义的操作。
至于使用运行时的同一实例或其他实例的其他实体能否看到该容器，则不属于本规范的范围。


## <a name="runtimeState" />State

容器的状态包括以下属性：

* **`ociVersion`** (string, REQUIRED) 是状态遵守的 OCI 运行时规范 的版本。
* **`id`** (string, REQUIRED) 是容器的ID。
  这 MUST 是这个主机上所有容器的唯一值。
  跨主机不需要满足唯一性。
* **`status`** (string, REQUIRED) 是容器运行时的状态。
  该值 MAY 是以下之一：

  * `creating`: 容器正在被创建，[生命周期](#lifecycle) 的第二步。
  * `created`: 容器结束 [创建操作](#create) （[生命周期](#lifecycle) 的第二步之后），且容器进程既没有退出也没有执行用户指定的程序
  * `running`: 容器进程处理完用户指定的程序到还没退出（ [生命周期](#lifecycle) 的第8步之后）
  * `stopped`: 容器进程已退出（ [生命周期](#lifecycle) 的第10步 ）

  附加值 MAY 由运行时定义，但是，它们 MUST 用于表示上面未定义的新运行时状态。
* **`pid`** (int, 在 Linux 上，当 `status` 是 `created` 或 `running` 时，REQUIRED。在其他平台 OPTIONAL) 容器进程的ID
  对于在运行时命名空间中执行的钩子，它是运行时看到的 `pid` 。
  对于在容器命名空间中执行的钩子，它是容器看到的 `pid` 。
* **`bundle`** (string, REQUIRED) 是容器的捆绑包文件夹的绝对路径。
  提供此信息是为了让消费者可以在主机上找到容器的配置和根文件系统。
* **`annotations`** (map, OPTIONAL) 包含与容器关联的注解列表。
  如果未提供注解，则此属性 MAY 不存在或为空对象。

状态 MAY 包含额外的属性。

当以 JSON 形式序列化时，格式 MUST 遵守 JSON 模式 [`schema/state-schema.json`](schema/state-schema.json)。

有关检索容器状态的信息，请参阅 [查询状态](#query-state) 。


### Example

```json
{
    "ociVersion": "0.2.0",
    "id": "oci-container1",
    "status": "running",
    "pid": 4422,
    "bundle": "/containers/redis",
    "annotations": {
        "myKey": "myValue"
    }
}
```


## <a name="runtimeLifecycle" />Lifecycle

生命周期描述了从容器被创建到其不再存在所发生的事件的时间线。

1. 在调用兼容 OCI 的运行时 [`create`](runtime_zh.md#create) 命令时，会引用捆绑包的位置和唯一标识符。
2. MUST 根据 [`config.json`](config_zh.md) 中的配置创建容器的运行时环境。
  如果运行时无法创建 [`config.json`](config_zh.md) 中指定的环境，则 MUST [生成错误](#errors) 。
  虽然 [`config.json`](config_zh.md) 中请求的资源 MUST 创建，但用户指定的程序（来自 [`process`](config_zh.md#process)）此时 MUST NOT 运行。
  此步骤后对 [`config.json`](config_zh.md) 的任何更新都 MUST NOT 影响容器。
3. 运行时 MUST 调用 [`prestart` 钩子](config_zh.md#prestart)。
  如果任何 `prestart` 钩子失败，运行时 MUST [生成错误](#errors) 、停止容器并在 **第 12 步**  继续生命周期。
4. 运行时 MUST 调用 [`createRuntime` 钩子](config_zh.md#createRuntime-hooks)。
  如果任何 `createRuntime` 钩子失败，运行时 MUST [生成错误](#errors) 、停止容器并在 **第 12 步** 继续生命周期。
5. 运行时 MUST 调用 [`createContainer` 钩子](config_zh.md#createContainer-hooks)。
  如果任何 `createContainer` 钩子失败，运行时 MUST [生成错误](#errors) 、停止容器并在 **第 12 步** 继续生命周期。
6. 运行时的 [`start`](runtime_zh.md#start) 命令是用容器的唯一标识符调用的。
7. 运行时 MUST 调用 [`startContainer` 钩子]。
  如果任何 `startContainer` 钩子失败，运行时 MUST [生成错误](#errors)、停止容器并在 **步骤 12** 继续生命周期。
8. 运行时 MUST 运行 [`process`](config_zh.md#process) 中指定的用户指定的程序。
9. 运行时 MUST 调用[`poststart` 钩子](config_zh.md#poststart)。
  如果任何 `poststart` 钩子失败，运行时 MUST [记录警告](#warnings) ，但其余钩子和生命周期仍将继续，就好像该钩子成功了一样。
10. 容器进程退出。
  发生这种情况的原因 MAY 是出错、退出、崩溃或调用了运行时的 [`kill`](runtime_zh.md#kill) 操作。
11. 运行时的 [`delete`](runtime_zh.md#delete) 命令将使用容器的唯一标识符调用。
12. MUST 通过撤销创建阶段（第 2 步）执行的步骤来销毁容器。
13. 运行时 MUST 调用[`poststop` 钩子](config_zh.md#poststop) 。
  如果任何 `poststop` 钩子失败，运行时必须 [记录警告](#warnings) ，但其余钩子和生命周期将继续进行，就好像该钩子已经成功一样。


## <a name="runtimeErrors" />Errors

在指定操作生成错误的情况下，本规范不强制要求如何（或者是否）将该错误返回或暴露给实现的用户。
除非另有说明，生成错误 MUST 使环境状态就像从未尝试过操作一样 - 对任何可能的微不足道的辅助更改（例如日志记录）进行取模。


## <a name="runtimeWarnings" />Warnings

在指定操作记录警告的情况下，本规范不强制要求如何（或者是否）将该警告返回或暴露给实现的用户。
除非另有说明，记录警告不会改变操作流程；它必须继续，就像警告没有被记录一样。


## <a name="runtimeOperations" />Operations

除非另有说明，运行时 MUST 支持以下操作。

注意：这些操作没有指定任何命令行 API，参数是一般操作的输入。


### <a name="runtimeQueryState" />Query State

`state <container-id>`

如果未提供容器的 ID，则此操作 MUST [生成错误](#errors) 。
尝试查询不存在的容器 MUST [生成错误](#errors) 。
此操作 MUST 返回 [State](#state) 部分中指定的容器状态。


### <a name="runtimeCreate" />Create

`create <container-id> <path-to-bundle>`

如果未提供包的路径以及与容器关联的容器 ID，则此操作 MUST [生成错误](#errors) 。
如果提供的 ID 在运行时范围内的所有容器中不是唯一的，或者以任何其他方式无效，则实现 MUST [生成错误](#errors) ，并且不得创建新容器。
此操作 MUST 创建一个新容器。

MUST 应用 [`config.json`](config_zh.md) 中配置的所有属性（[`process`](config_zh.md#process) 除外）。
[`process.args`](config_zh.md#process) MUST NOT 应用，直到由 [`start`](#start) 操作触发。
其余的 `process` 属性 MAY 通过此操作应用。
如果运行时无法应用 [配置](config_zh.md) 中指定的属性，则它 MUST [生成错误](#errors) 并且 MUST NOT 创建新容器。

在创建容器（[第 2 步](#lifecycle)）之前，运行时 MAY 根据此规范对 `config.json` 进行一般验证或根据本地系统能力进行验证。
对创建前验证感兴趣的 [运行时调用者](glossary_zh.md#runtime-caller)可以在调用创建操作前运行 [捆绑验证工具](implementations_zh.md#testing--tools)。

此操作后对 [`config.json`](config_zh.md) 文件所做的任何更改都不会对容器产生影响。


### <a name="runtimeStart" />Start

`start <container-id>`

如果未提供容器 ID，则此操作 MUST [生成错误](#errors) 。
尝试 `start` 未 [`created`](#state) 的容器 MUST 对容器没有影响并且 MUST [生成错误](#errors) 。
此操作必须按照 [`process`](config_zh.md#process) 指定的方式运行用户指定的程序。
如果未设置 `process` ，则此操作 MUST 生成错误。


### <a name="runtimeKill" />Kill

`kill <container-id> <signal>`

如果没有提供容器 ID，此操作 MUST [产生错误](#errors)。
试图向既未 [`created` 或 `running`](#state) 的容器发送信号时，MUST 对容器不起作用，并且 MUST [产生错误](#errors) 。
此操作 MUST 向容器进程发送指定的信号。


### <a name="runtimeDelete" />Delete

`delete <container-id>`

如果未提供容器 ID，则此操作 MUST [产生错误](#errors)。
尝试 `delete` 未 [`stopped`](#state) 的容器 MUST 对容器没有影响并且 MUST [产生错误](#errors) 。
删除容器 MUST 删除在 `create` 步骤中创建的资源。
请注意，与容器关联但不是由该容器创建的资源 MUST NOT 删除。
容器删除后，其 ID MAY 会被后续容器使用。


## <a name="runtimeHooks" />Hooks

本规范中指定的许多操作都有 “钩子”，允许在每个操作之前或之后采取附加操作。
有关详细信息，请参阅 [钩子的运行时配置](config_zh.md#posix-platform-hooks)。
