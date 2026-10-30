# kaomoji-skill

给 agent 的回复添一点颜文字——不是装饰，是语气。

## 为什么

GPT 回复里偶尔蹦个 😊，一下子就"有活人味"了。颜文字是同一件事的东亚版本：信息不变，温度变了。

但颜文字有个通病——**一加就上瘾**。每句缀一个，读起来像自动回复。所以这个 skill 的重点不是"有颜文字"，是**在哪加、加多少、什么时候不加**。

## 三个语言版本

| 目录 | 面向 | 默认风格 |
|---|---|---|
| [`kaomoji-zh/`](kaomoji-zh/SKILL.md) | 中文回复 | 全角颜文字 |
| [`kaomoji-ja/`](kaomoji-ja/SKILL.md) | 日语回复 | 全角颜文字，另分出「网络掲示板ノリ」一类 |
| [`kaomoji-en/`](kaomoji-en/SKILL.md) | 英语回复，**面向二次元/otaku 语域的英语用户** | 全角颜文字，ASCII 仅作纯文本环境兜底 |

英文版没有做"本土化成 `:)` `¯\_(ツ)_/¯`"——那是另一个产品的定位。这里假设用户看得懂 `(´･ω･｀)` 并想要这个调调。如果你的目标用户是普通英语使用者，应该改英文版里的「Style note」一节，把默认风格换成 ASCII。

各版本独有的注意事项：

- **中文**：颜文字与表情符号混用的边界（中文语境里两者都常见）。
- **日文**：敬语文书面（邮件、报告、申请文书）里一律不用；`(ﾟ∀ﾟ)` `( ＾ω＾)` 这类 2ch / なんJ 风格的单列一类，初対面不用。
- **英文**：ASCII 兜底何时该用（纯文本日志、终端、commit message），以及为什么它不是"风格选择"。

> ⚠️ **关于日文版：作者不懂日语。**
>
> `kaomoji-ja/` 只是初稿——条目和规则照着日语网络文化的通行用法整理，但语感和场合分寸没法保证。**用之前请自己过一遍**，特别是「禁区」一节：哪句在敬语场合会失礼，只有母语者判断得准。
>
> 中文版、英文版没有这个顾虑。

## 设计取舍

| 需求 | 做法 | 为什么 |
|---|---|---|
| 查得快 | 速查表直接写在 SKILL.md 里 | skill 触发时 SKILL.md 已在上下文，零工具调用就能挑 |
| 查得全 | `reference/kaomoji.md` 按情绪分节 | 需要时 `grep '^## 无奈'` 拿一节，不通读 |
| 程序可用 | `data/kaomoji.json` | 给自建工具 / MCP / 批量导出用 |
| 频率可调 | SKILL.md 里一行 `当前设置：low` | 改一个词就换档 |
| 不滥用 | 硬规则 + 禁区清单 + 反例 | 靠"上限"约束，不靠 agent 自觉 |

**没做的事**：不写查找脚本。多一次工具调用不值——速查表覆盖了 90% 的场合。

## 结构

```
kaomoji-zh/
  SKILL.md              频率档、硬规则、禁区、速查表（12 类）
  reference/kaomoji.md  扩展库，16 节约 125 个，带气质标注
  data/kaomoji.json     结构化数据，程序用
kaomoji-ja/
  SKILL.md              同上 + 敬语文书禁用、2ch 风格单列
  reference/kaomoji.md  17 节约 145 个
  data/kaomoji.json     同上
kaomoji-en/
  SKILL.md              同上 + ASCII 兜底说明，面向英语 + 二次元语域
  reference/kaomoji.md  17 节约 120 个
  data/kaomoji.json     同上
```

三个目录互相独立，装哪个用哪个。

## 简单使用

装一个就够。**挑和你回复语言对应的那一个目录**：

| 你的回复语言 | 用这个 |
|---|---|
| 中文 | `kaomoji-zh/` |
| 日语 | `kaomoji-ja/` |
| 英语 | `kaomoji-en/` |

核心动作只有一个：**把整个文件夹复制到 agent 的 skill 目录下**。

```
要复制的：kaomoji-zh/          ← 整个文件夹，里面自带 SKILL.md
         ├── SKILL.md
         ├── reference/
         └── data/
```

> 别只复制 `SKILL.md` 一个文件。`reference/kaomoji.md` 是扩展库，SKILL.md 里会引用它；只拷 SKILL.md 也能用，但「不够用时」那一步会找不到文件。

---

### Claude Code

**用户级**（所有项目都能用）：

| 系统 | 目录 |
|---|---|
| Windows | `C:\Users\<你的用户名>\.claude\skills\` |
| macOS / Linux | `~/.claude/skills/` |

**项目级**（只在某个项目里用）：项目根目录下的 `.claude\skills\`。

复制完之后，目录长这样：

```
C:\Users\你\.claude\skills\kaomoji-zh\SKILL.md
```

使用命令行：

```bash
git clone https://github.com/Mioayaya/kaomoji-skill.git && cp -r kaomoji-skill/kaomoji-zh ~/.claude/skills/
```

不想敲命令：在文件管理器里打开 `C:\Users\你的用户名\.claude\skills\`（没有 `skills` 文件夹就自己建一个），把下载下来的 `kaomoji-zh` 文件夹整个拖进去。

装完**新开一个对话**，问一句「你现在有哪些 skill」，能看到 `kaomoji-zh` 就成了。之后不用特意召唤，它会按 `description` 自动触发。

---

### Codex

Codex 已经原生支持 `SKILL.md` 这种 skill 格式（早期版本不支持，靠往 `AGENTS.md` 里贴）。

**推荐位置**：`~/.agents/skills/`（Windows 是 `C:\Users\<你>\.agents\skills\`）。这个目录 Codex 和 dsh 都能扫到，装一次两边都能用。

```
C:\Users\你\.agents\skills\kaomoji-zh\SKILL.md
```

如果放了没反应，可以再试 `~/.codex/skills/`——不同版本的搜索路径有过变动，这两个是社区里最常见的两种说法。

**退路**：如果你的 Codex 版本确实不认 skill 目录，就把 `SKILL.md` 的正文贴进 `AGENTS.md`——项目根目录放一份，或者放 `~/.codex/AGENTS.md` 全局生效。文件本身是自足的，去掉 frontmatter 也能用。

---

### dsh（DeepSeek Harness）

| 级别 | 目录 |
|---|---|
| 用户级 | `~/.dsh/skills/` |
| 项目级 | 项目根目录下 `.dsh/skills/` |
| 通用（和 Codex 共用） | `~/.agents/skills/` |

```
C:\Users\你\.dsh\skills\kaomoji-zh\SKILL.md
```

放完重启 dsh，或者刷新一下页面。也可以直接在 dsh 的「设置 → Agent Skills」里确认它有没有被列出来。

> 这几个路径来自社区文档，dsh 迭代较快，版本之间可能不同。最稳的判断方式是在「设置 → Agent Skills」里看它有没有被扫到；扫不到就换个目录再试。

---

### 装完怎么验证

随便问一句：

> 你现在有哪些 skill？

agent 能报出 `kaomoji-zh` 就说明装好了。之后聊几句轻松的、看看回复结尾有没有偶尔冒出一个颜文字——**注意是"偶尔"**，默认档位就是 `low`，第一次没出现也正常。

### 只想抄到系统提示里

不装 skill 也行，把 `SKILL.md` 里的这四节粘进系统提示或 `AGENTS.md` 就够了：

- 使用频率
- 四条硬规则
- 禁区
- 速查表

其余（结构说明、示例）是给人看的，可以不要。frontmatter 那一段（`---` 包住的部分）是给 skill 系统读的，拼进系统提示时删掉。

> 三个目录的 `name:` 字段和文件夹名是对齐的（`kaomoji-zh` / `kaomoji-ja` / `kaomoji-en`），可以同时装、互不冲突。但**没必要**——每个 skill 各自有频率档，多装不会让颜文字变多，只会多占一点上下文。

## 调频

编辑 `SKILL.md` 中的这一行：

```markdown
当前设置：`low`
```

英文版是 `Current setting: \`low\``。可选 `off` / `low` / `medium` / `high`，默认 `low`——**宁缺毋滥**是这个 skill 的立场。

运行时也能口头切："说人话，别加颜文字" → 当次对话停用；"放松点" → 回默认档。

## 贡献

加颜文字：往 `reference/kaomoji.md` 对应分节里加，行末标气质 `[轻][柔][强][酷][丧]`（英文版 `[good][soft][strong][deadpan][low]`）；同步补 `data/kaomoji.json`。合并前跑：

```bash
for f in kaomoji-*/data/kaomoji.json; do node -e "JSON.parse(require('fs').readFileSync('$f','utf8'))" || echo "BAD: $f"; done
```

加规则：优先改「禁区」而不是加"应该用"的场景——**限制比鼓励有效**。
