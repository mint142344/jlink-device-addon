# JLink Device Addon

将 CMSIS Pack（`.pack`）里的 MCU 器件信息转成 J-Link 的 `JLinkDevices.xml` 条目，并导出 Flash 算法（`.FLM`），让 J-Link Commander / Flasher 能识别并烧录该芯片。

> 运行环境：**Python 3.14+**（用 uv 管理），**不需要 Python 3.6**。

## 前置要求

- 已安装 [uv](https://docs.astral.sh/uv/)（在 PATH 内）
- Python 3.14：uv 会自动使用仓库内 `.venv`；若缺失执行 `uv python install 3.14`

## 用法

```bash
cd <仓库目录>
uv run python __main__.py -x "<JLinkDevices.xml 路径>" "<你的 .pack 路径>"
```

示例（追加到 J-Link 安装目录的数据库）：

```bash
uv run python __main__.py \
  -x "C:/Program Files/SEGGER/JLink_V756d/JLinkDevices.xml" \
  "C:/path/to/HDSC.HC32F052.1.1.0.pack"
```

要点：

- **没有 `jlink-device-addon` 命令**（pyproject 未定义 console script），必须写 `uv run python __main__.py`。
- `-x` 指定目标 `JLinkDevices.xml`。不传会尝试 `where jlink` 自动定位安装目录，但 J-Link 通常不在 PATH，建议显式传入。
- 脚本流程：解析 pack → 在 XML 中追加 `<Device>` 节点（自动跳过已存在的重复器件）→ 将 Flash 算法（`.FLM`）导出到 XML 同级的 `Devices/<Vendor>/<Family>/` 目录。
- 目标 `JLinkDevices.xml` 不存在时自动创建（根节点 `<DataBase>`）。

## 首次初始化（可选）

脚本会自动建库，也可手动建空库：

```xml
<DataBase>
</DataBase>
```

注意是 `<DataBase>`（首字母大写 B）。

## 安装到 J-Link（需管理员）

`C:/Program Files/...` 是受保护目录，普通终端写不进去。推荐做法：

1. 在**可写目录**用上面的命令生成（产出 `JLinkDevices.xml` + `Devices/`）；
2. 以**管理员**身份把两者一起拷进 J-Link 安装目录（Loader 路径是相对的，XML 与 `Devices` 必须同时拷，少一个都会烧录报错）。

```powershell
$src = "C:\path\to\generated"
$dst = "C:\Program Files\SEGGER\JLink_V756d"
Copy-Item "$src\JLinkDevices.xml" -Destination "$dst\JLinkDevices.xml" -Force
Copy-Item "$src\Devices" -Destination "$dst\" -Recurse -Force
```

## 验证

连上 J-Link 后，在 J-Link Commander 中：

```
J-Link>Device HC32F052JAxA
J-Link>Connect
```

不接硬件也能确认器件已注册：

```
J-Link>ExpDevList C:\temp\devlist.txt
```

然后（PowerShell）检索：

```powershell
Select-String -Path C:\temp\devlist.txt -Pattern "HC32F052"
```

应能列出该系列器件。

## 已知告警（无害）

- `pkg_resources is deprecated`：vendored pyocd 依赖带出，忽略。
- `if element:` 的 `DeprecationWarning`：Python 3.14 下正常工作，忽略。

## 备注

- 仓库内 `pyocd/` 是冻结的旧版 fork，仅服务于本转换工具。
- 老方案 `setup.sh`（Python 3.6 + `pip -t`）已废弃，不要使用。
