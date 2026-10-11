# LTspice Model Downloader

一个用于在本机 LTspice 符号库中查找芯片型号，并在找不到时从 GitHub 搜索 SPICE 模型的 Windows 工具。

## 现在怎么用

直接双击 `LTspiceModelDownloader.exe`，或在命令行里运行：

```powershell
LTspiceModelDownloader.exe
```

常用参数：

```powershell
LTspiceModelDownloader.exe --help
LTspiceModelDownloader.exe -c LM358
LTspiceModelDownloader.exe -c LM358 -c TL431 -o D:\models
LTspiceModelDownloader.exe --sym-dir "D:\LTspice\lib\sym"
```

## 产物放哪

- 下载到的模型默认保存到程序同目录下的 `ltspice_models`
- GitHub Token 默认保存到程序同目录下的 `.github_token`
- 打包脚本会额外生成 `release` 目录，里面放可直接分享的 exe 和说明文件

## 构建 exe

先安装依赖：

```powershell
pip install -r requirements.txt
```

然后运行：

```powershell
python build_exe.py
```

构建完成后，`release` 目录就是适合发给别人的版本。

## 提示

- 如果你有自己的 LTspice 符号库，可以设置环境变量 `LTSPICE_SYM_DIR`
- 也可以直接用 `--sym-dir` 指定目录
- 如果 GitHub API 触发限流，输入一个 Personal Access Token 可以提高成功率
