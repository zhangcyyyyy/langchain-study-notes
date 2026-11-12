# 课程笔记复习助手

课程笔记复习助手是一款基于 **LangChain + Ollama 本地大模型** 的命令行应用。给它一份课程笔记，它会生成中文 Markdown 复习提纲，包括核心概念、简答自测题和分开排版的参考答案，方便先做题再核对。

[GitHub 开源仓库](https://github.com/zhangcyyyyy/langchain-study-notes)

## 快速开始

准备 Python 3.10 或更新版本和 [Ollama](https://ollama.com/download)。安装依赖和首次下载模型需要联网。准备完成后，生成复习材料可离线进行。

### 1. 获取源码

已下载 ZIP 的话，先解压，再在包含 `study_notes.py` 和 `requirements.txt` 的目录打开 PowerShell。也可以用 Git 将仓库克隆到「文档」目录。把下面整个代码块一起粘贴到 PowerShell：

```powershell
& {
    $project = Join-Path ([Environment]::GetFolderPath('MyDocuments')) 'langchain-study-notes'
    git clone https://github.com/zhangcyyyyy/langchain-study-notes.git "$project"
    if ($LASTEXITCODE -ne 0) { throw '克隆失败。请检查 Git 的报错，或下载并解压源码 ZIP。' }
    Set-Location -LiteralPath $project -ErrorAction Stop
}
```

如果已经克隆过，直接进入现有项目目录即可。只有两个源码文件都能在当前目录找到时，才继续下一步。

### 2. 安装并运行

打开 Ollama，确保它在后台运行。如果尚未下载模型，先运行 `ollama pull qwen2.5:7b`，看到 `success` 后继续。

在项目目录把下面整个代码块一起粘贴到 PowerShell：

```powershell
& {
    if (!(Test-Path -LiteralPath 'study_notes.py' -PathType Leaf) -or
        !(Test-Path -LiteralPath 'requirements.txt' -PathType Leaf)) {
        throw '当前目录缺少项目文件。请先进入解压或克隆后的项目目录。'
    }
    python -m venv .venv
    if ($LASTEXITCODE -ne 0) { throw '创建虚拟环境失败。' }
    .\.venv\Scripts\python.exe -m pip install -r requirements.txt
    if ($LASTEXITCODE -ne 0) { throw '安装依赖失败。' }
    .\.venv\Scripts\python.exe study_notes.py examples/git-notes.md -o review.md
}
```

生成后打开项目目录里的 `review.md`。再次运行时可指定新的文件名，或省略 `-o review.md`，把结果显示在终端。

macOS 或 Linux 上，在包含源码的目录运行 `python3 -m venv .venv`，然后用 `.venv/bin/python -m pip install -r requirements.txt` 安装依赖，最后运行 `.venv/bin/python study_notes.py examples/git-notes.md -o review.md`。

输入支持 UTF-8 编码的 `.txt` 和 `.md` 文件，单份最多 4,000 字符。输出路径须指向新文件。默认模型约 4.7 GB。也可以先下载更小的模型，再用 `--model qwen2.5:1.5b` 指定模型标签。[Ollama 模型说明](https://ollama.com/library/qwen2.5)

## 设计与实现

```text
笔记文件 → 输入检查 → ChatPromptTemplate → ChatOllama → 结构化复习材料 → Markdown
```

1. **读取笔记**。用 Python 检查文件类型、UTF-8 编码、空内容和长度，尽早提示输入问题。
2. **构建提示**。LangChain 的 `ChatPromptTemplate` 将复习任务放在系统消息中，将笔记正文作为单独的用户消息，并要求题目和答案以笔记为依据。
3. **调用模型**。`ChatOllama` 通过本机服务执行一次推理，使用较低温度减少输出波动。LangChain 的结构化输出接口要求模型按 JSON Schema 返回核心概念和题目、答案配对。
4. **输出结果**。程序把题目和答案分别渲染为 Markdown 小节，使用同一组题号，最后输出到终端或保存到文件。

长度上限和生成长度限制用于控制推理开销。应用会对服务未启动、模型未下载和请求超时给出提示。实测发现，仅靠提示词有时会让答案紧跟在题目后面，因此改用结构化输出，由程序固定排版。模型仍可能遗漏知识点或生成有误的答案，使用时可对照原笔记核对。

## 示例

[示例输入](examples/git-notes.md)介绍了 Git 的工作区、暂存区、提交和分支。[实测输出](examples/git-review.md)由 Windows 上的 Qwen2.5 7B 生成，包含 3 条核心概念和 3 道简答题。例如，第 3 题问「如何创建并切换到新分支」，答案可以在最后一个小节按编号找到。

另行检查了空笔记、超长笔记、错误编码、文件不存在、文件类型不支持，以及输出文件已经存在时的提示。

## 开源许可

应用代码和示例采用 [MIT 许可证](LICENSE)。模型由 Ollama 单独下载，模型许可见 [Qwen2.5 模型页面](https://ollama.com/library/qwen2.5)。

## 技术资料

- [LangChain 的 ChatOllama 集成文档](https://docs.langchain.com/oss/python/integrations/chat/ollama)
- [Ollama 的 Qwen2.5 模型页面](https://ollama.com/library/qwen2.5)
