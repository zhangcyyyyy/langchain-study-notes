# 课程笔记复习助手

课程笔记复习助手是一款基于 **LangChain + Ollama 本地大模型** 的命令行应用。给它一份课程笔记，它会生成中文 Markdown 复习提纲，包括核心概念、简答自测题和分开排版的参考答案，方便先做题再核对。

[GitHub 开源仓库](https://github.com/zhangcyyyyy/langchain-study-notes)

## 快速开始

准备 Python 3.10 或更新版本和 [Ollama](https://ollama.com/download)。安装依赖和首次下载模型需要联网。准备完成后，生成复习材料可离线进行。

在项目目录打开终端。以下是 Windows PowerShell 命令：

```powershell
git clone https://github.com/zhangcyyyyy/langchain-study-notes.git
cd langchain-study-notes
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
ollama pull qwen2.5:7b
.\.venv\Scripts\python.exe study_notes.py examples/git-notes.md -o review.md
```

macOS 或 Linux 上可用 `python3 -m venv .venv` 创建环境，并把上面的 `.\.venv\Scripts\python.exe` 替换为 `.venv/bin/python`。

运行前确认 Ollama 正在后台运行。也可以在另一个终端用 `ollama serve` 启动服务。生成后打开 `review.md` 即可阅读；省略 `-o review.md` 会把结果显示在终端。

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
