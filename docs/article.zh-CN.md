# LTspice Model Downloader：本地符号检索与 SPICE 模型下载工具

> 输入芯片型号，先查找本机 LTspice 符号库；如果没有找到，再从 GitHub 搜索并下载候选 SPICE 模型文件。支持 Windows 双击运行和命令行批量处理。

使用 LTspice 搭建电路时，查找器件符号和仿真模型往往需要反复翻找目录、搜索仓库、保存文件。LTspice Model Downloader 将这些查找步骤集中到一个小工具中，帮助你确认本机是否已有对应符号，并在缺少符号时获取网上的候选模型。

工具采用控制台交互界面，适合在 Windows 电脑上本地运行。下载的模型仍需要自行检查并接入 LTspice。

## 它能做什么

- **优先查找本地符号**：自动发现常见 LTspice / LTspice XVII 符号库目录，包含部分文档目录和 OneDrive 场景。
- **按型号匹配**：检查 `.asy` 文件名，以及符号中的 `Value`、`Value2`、`SpiceModel` 属性，匹配时忽略大小写。
- **搜索并下载候选模型**：本地未命中时，在 GitHub 中检索 `.lib`、`.sub`、`.cir` 文件，下载搜索结果中的第一项。
- **支持连续查询和批量处理**：可以在窗口中逐个输入型号，也可以用多个 `-c` 参数一次处理多个型号。
- **支持自定义目录**：可指定本地符号库位置和下载文件的保存位置。

整体流程是：**输入型号 → 查找本地符号 → 找到则显示路径；未找到则尝试 GitHub 搜索与下载。**

## 快速开始

### 1. 下载并解压

将压缩包解压到一个有写入权限的文件夹，例如 `D:\Tools\LTspiceModelDownloader`，然后双击 `LTspiceModelDownloader.exe`。

使用打包好的 EXE 无需另外安装 Python。本地符号检索可以离线使用；GitHub 搜索和下载需要网络连接。如果电脑上没有安装 LTspice，仍可尝试在线搜索，但后续仿真需要安装 LTspice。

建议先解压再运行，便于保留下载的模型和程序配置。

### 2. 查看符号库识别结果

启动后，窗口会显示模型保存目录、Token 文件位置，以及发现的 LTspice 本地符号库。

如果没有发现符号库，或你使用了自定义安装路径，可以按照后文的 `--sym-dir` 示例手动添加目录。

### 3. 输入芯片型号

在提示处输入一个型号并按回车，例如：

```text
请输入芯片型号 (例如 LM358，退出请输入 q): LM358
```

程序会依次执行：

1. 查找本地符号。找到时显示符号文件路径，并结束该型号的查询。
2. 本地未找到时，尝试从 GitHub 搜索候选模型。
3. 搜索和下载成功后，显示文件名、来源仓库及保存路径。

一次查询结束后，可以继续输入下一个型号。输入 `q` 并回车退出程序。

这里的 `LM358` 仅用于演示输入方式，实际结果取决于本机符号库、GitHub 搜索结果和网络状态。

### 4. 按提示配置 GitHub Token

需要联网搜索且程序没有检测到 Token 时，窗口会提示输入 GitHub Personal Access Token。可在 [GitHub Token 设置页面](https://github.com/settings/tokens) 创建；程序中的提示以 Classic Token 为例。

可以直接回车跳过，但跳过不代表能够匿名完成代码搜索，仍可能遇到认证失败或限流。若出现 `401` 或 `403`，请结合窗口中的服务器信息检查 Token 是否有效、是否过期，以及请求限制情况。

程序读取 Token 的顺序为：

1. 环境变量 `GITHUB_TOKEN`。
2. 程序同目录下的 `.github_token` 文件。
3. 以上均未提供时，在需要联网搜索时提示输入。

**当前版本的 Token 输入不会遮蔽，输入后会明文保存在 `.github_token` 中。** 分享工具、打包文件或上传网站时，请排除这个文件，也不要将 Token 放进截图或公开命令示例。

### 5. 找到下载文件

默认下载位置为 EXE 同目录下的 `ltspice_models` 文件夹，例如：

```text
LTspiceModelDownloader/
├── LTspiceModelDownloader.exe
├── README.md
└── ltspice_models/
    └── 下载得到的模型文件
```

程序保留远端文件名。不同查询如果下载到同名文件，后一次下载会覆盖目标目录中的已有文件；需要保留不同版本时，可通过 `-o` 指定不同目录。

## 命令行用法

在 EXE 所在文件夹打开 PowerShell，执行以下命令。PowerShell 中运行当前目录的程序时，使用 `.\` 前缀，即示例里的 `.\LTspiceModelDownloader.exe`。

### 查看帮助或进入交互模式

```powershell
.\LTspiceModelDownloader.exe --help
.\LTspiceModelDownloader.exe
```

### 查询一个或多个型号

```powershell
.\LTspiceModelDownloader.exe -c LM358
.\LTspiceModelDownloader.exe -c LM358 -c TL431
```

每个型号使用一个 `-c` 参数。程序逐个处理，完成后退出；如果途中需要 GitHub Token，仍可能提示输入。

### 指定模型保存目录

```powershell
.\LTspiceModelDownloader.exe -c LM358 -o "D:\models"
```

### 添加自定义符号库

```powershell
.\LTspiceModelDownloader.exe --sym-dir "D:\LTspice\lib\sym" -c LM358
```

请指定实际存放 `.asy` 符号的目录，程序会递归查找其子目录。`--sym-dir` 用于添加查找位置，程序仍会尝试发现其他常见符号库。

多个目录可以重复传入：

```powershell
.\LTspiceModelDownloader.exe --sym-dir "D:\LTspice\lib\sym" --sym-dir "D:\MySymbols" -c LM358
```

也可以在当前 PowerShell 会话中设置环境变量；Windows 下用分号分隔多个路径：

```powershell
$env:LTSPICE_SYM_DIR = 'D:\LTspice\lib\sym;D:\MySymbols'
.\LTspiceModelDownloader.exe
```

上述环境变量设置只作用于当前 PowerShell 会话及其启动的程序。

### 参数速查

| 参数 | 作用 |
| --- | --- |
| `-h` / `--help` | 显示帮助并退出 |
| `-c` / `--chip` | 指定型号；可重复使用，不指定则进入交互模式 |
| `-o` / `--output` | 指定下载保存目录 |
| `--sym-dir` | 添加本地符号库目录；可重复使用 |

## 下载后，怎样在 LTspice 中使用

下载完成表示候选文件已经保存，接下来需要根据模型内容完成接入。文件后缀并不能单独决定用法，建议先用文本编辑器打开文件，查看注释、模型声明和外部依赖。

### 对于 `.SUBCKT` 子电路模型

一种常见做法是：

1. 将模型文件放到 LTspice 原理图所在文件夹。
2. 在原理图中添加 SPICE 指令，例如 `.include "example.lib"`，将文件名替换为实际文件名。
3. 选择或创建合适的符号，将其 `Prefix` 设置为 `X`。对于普通子电路符号，将 `Value` 设置为 `.SUBCKT` 后面的名称；带有其他模型调用属性的现有符号则按其说明配置。
4. 核对符号引脚的 `SpiceOrder` 与 `.SUBCKT` 声明中的端口顺序，确认电源和信号引脚对应正确。
5. 如果文件还引用其他模型或库，一并补齐，然后运行一个简单测试电路。

例如，假设文件包含以下声明：

```spice
.SUBCKT EXAMPLE_AMP INP INN VP VN OUT
```

那么调用名称是 `EXAMPLE_AMP`，端口顺序依次为正输入、负输入、正电源、负电源、输出。这里仅演示声明格式，不是一份可运行模型；具体器件必须按实际文件的声明和说明接线。

### 对于其他类型的文件

- **`.MODEL` 器件模型**：需使用对应器件类型的符号并引用正确的模型名，不应直接套用子电路的 `Prefix = X` 设置。
- **完整 `.cir` 电路文件**：可能包含完整测试电路和仿真指令，需要先判断是直接打开运行，还是提取其中的模型部分。

该工具不会自动生成符号、导入模型、调整引脚映射或验证 LTspice 兼容性。

## 常见问题

### 提示本地已存在，为什么仿真仍然失败？

本地命中表示找到了匹配的 `.asy` 符号，不代表其引用的模型文件完整，也不代表电路连接正确。程序不会继续检查模型依赖，且本地命中后会跳过该型号的在线下载。

### 明明有相近型号，为什么没有匹配到？

当前本地检索按文件名或指定属性的完整值匹配，忽略大小写，但不做模糊匹配。例如，`LM358` 与带有额外后缀的型号不一定匹配。可核对实际符号名称，或换用模型声明中的名称重新查询。

### 为什么 GitHub 没有搜索结果？

可能是型号写法不同、文件后缀不在检索范围、文件未被代码搜索收录，或相关模型没有发布在可搜索的仓库中。可以尝试基础型号，并到器件厂商的产品页面查找仿真模型。

### 下载的文件为什么不一定能直接使用？

当前程序自动采用 GitHub 返回的第一项结果，尚不提供候选列表选择，也不检查文件是否包含所需模型、是否缺少依赖或是否使用其他 SPICE 软件的专有语法。使用前请结合来源仓库说明和器件资料核对；转载模型时也应查看原仓库的许可要求。

### 双击窗口很快关闭，怎么看错误信息？

在程序所在目录打开 PowerShell，再执行 `.\LTspiceModelDownloader.exe`，即可保留窗口中的错误信息。根据提示检查程序目录的写入权限、网络连接和配置路径。

## 可选：从源码运行

如果你使用的是完整源码目录，可以在已安装 Python 的环境中执行：

```powershell
python -m pip install -r requirements.txt
python ltspice_model_downloader.py
```

源码运行支持相同的参数；默认模型目录和 `.github_token` 位于脚本同目录。使用 EXE 便携版的用户无需执行这一步。

## 下载资源

[下载 LTspice Model Downloader Windows 便携版](https://nailoong0721.com/downloads/LTspiceModelDownloader-share.zip)

- 文件大小：15,023,171 字节，约 14.3 MiB。
- SHA-256：`84558A48B772DEDEAB1EFAD264495C51878F71041C1A0276A3F82236AE28B35A`。
- 下载包不包含 GitHub Token；需要联网搜索时，请仅在自己的电脑上按本文说明配置。

## GitHub

[查看 LTspice Model Downloader 项目仓库](https://github.com/nailoong0721/ltspice-model-downloader)。仓库包含本文与对应工具包。
