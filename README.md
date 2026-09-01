# wechat-draft-publisher-skill

把一篇 Markdown（或已排版的 HTML）安全地转换并上传为**微信公众号草稿**，它只创建草稿，不直接群发、不读浏览器 Cookie、不跨平台同步。

本仓库是 [jiji262/wechat-publisher](https://github.com/jiji262/wechat-publisher) 的 fork，保留了本地化且更安全的"转 Markdown → 传图 → 建草稿"流程，并把容易出风险的环节砍掉了。

## 它做什么 / 不做什么

**做：**

- 把 Markdown 转成微信兼容的 HTML（内联样式排版）。
- 把文章/封面图片下载并上传到微信。
- 调用微信公众号 API 创建草稿（`news` 图文 或 `newspic` 贴图/图片消息两种模式）。
- 发布前可先跑本地的 `ai_score.py` 做一遍"AI 味"写作检查。

**不做（本 fork 有意移除）：**

- 基于浏览器 Cookie 的 Gemini Web 自动化。
- Wechatsync / 多平台同步（知乎、掘金、CSDN、微博、小红书等）。
- 任何直接群发/公开发布流程——脚本只创建草稿，需登录 `mp.weixin.qq.com` 手动确认发布。

## 仓库结构

```text
wechat-draft-publisher-skill/
├── SKILL.md                        # Skill 指令（流程、边界、检测开关）
├── README.md
├── wechat-publisher.yaml.example   # 配置模板（复制后填真实凭据，勿提交）
├── brief.md.example                # 贴图(newspic)模式的输入模板
├── scripts/
│   ├── publish.py                  # 主入口：Markdown/HTML/brief → 草稿
│   ├── html_converter.py           # Markdown → 微信兼容 HTML
│   ├── image_handler.py            # 图片下载 + 上传微信
│   ├── wechat_api.py               # 公众号 API 封装（含 list-image-styles）
│   ├── wechat_token.py             # access_token 获取/缓存
│   ├── config.py                   # 统一配置读取
│   ├── ai_score.py                 # "AI 味"写作检测
│   ├── newspic_build.py            # brief.md → 卡片出图计划 card_plan.json
│   ├── generate_image.py           # 统一生图入口（可选，需 bun + API 凭据）
│   ├── baoyu_image_gen.ts / baoyu_image_gen_core.ts   # 生图调用脚本
│   └── api.py
├── assets/
│   ├── themes/*.json               # 15 套排版主题
│   ├── image-styles/*.json         # 22 种配图风格
│   └── theme-previews/             # 主题静态渲染预览
├── references/api_reference.md     # 微信公众号 API 接口参考
├── tests/                          # pytest 测试
└── generated/                      # 运行时产物输出目录（git 忽略）
```

## 安装

```bash
git clone https://github.com/onlyforchris/wechat-draft-publisher-skill.git
cd wechat-draft-publisher-skill
pip install requests pyyaml
```

仅当你要使用生图功能时，还需要 bun 和 API 凭据：

```bash
python scripts/generate_image.py --help
```

## 配置

```bash
cp wechat-publisher.yaml.example wechat-publisher.yaml
```

编辑 `wechat-publisher.yaml`：

```yaml
default: main

accounts:
  main:
    name: "My Official Account"
    app_id: "wx..."
    app_secret: "..."
    author: "Author"
    theme: "refined-blue"
    image_style: "warm-handdrawn"
    newspic_image_style: "infographic-warm"

image_generation:
  generator: "baoyu-image-gen"
  openai:
    api_key: ""
    base_url: ""
    image_model: ""
  gemini_proxy:
    api_key: ""
    base_url: "https://generativelanguage.googleapis.com"
    image_model: ""
```

`image_style`（文章配图）与 `newspic_image_style`（贴图）是分开的两套默认，不要混用。**不要提交 `wechat-publisher.yaml`**，它包含 `app_secret`。

## 使用方式

### 发布一篇 Markdown 图文草稿

```bash
python scripts/publish.py \
  --account main \
  --input article.md \
  --cover cover.jpg \
  --title "Article title" \
  --digest "Short summary"
```

命令返回草稿的 `media_id`。登录 `mp.weixin.qq.com` 查看草稿箱并手动发布。

### 发布已排版的 HTML

```bash
python scripts/publish.py --html article.html --cover cover.jpg --title "Article title"
```

> HTML 模式要求同时提供 `--title` 和 `--cover`，且该模式不做"AI 味"检测。

### 发布贴图（newspic 图片消息）

贴图模式用 `brief.md` 描述内容，先生成出图计划，再逐张生图、上传、建草稿：

```bash
python3 scripts/newspic_build.py brief.md        # 生成 card_plan.json
python3 scripts/publish.py --account main --type newspic --brief brief.md
```

`brief.md` 的模板见 [`brief.md.example`](brief.md.example)：frontmatter 里可配 `topic`、`card_count`、`title`、`account`、`image_style`，正文分 `# 要点` 和 `# 短文本` 两节。

### 发布前做"AI 味"检查

```bash
python scripts/ai_score.py article.md --threshold 45
```

默认阈值 45 分（总分 0–100），总分 ≥ 阈值时 `publish.py` 会**阻止发布**。可用 `--skip-ai-score` 跳过（不推荐），或用 `--ai-score-threshold 55` 放宽阈值。

## 常用参数

`publish.py` 主要参数：

| 参数 | 说明 |
| --- | --- |
| `--input` / `-i` | Markdown 文件路径 |
| `--html` | 已排版的 HTML 文件 |
| `--brief` | 贴图模式的 brief.md 路径 |
| `--type` | `news`（图文，默认）/ `newspic`（贴图） |
| `--title` / `-t`、`--cover` / `-c`、`--author` / `-a`、`--digest` / `-d` | 标题 / 封面 / 作者 / 摘要 |
| `--theme` | 排版主题名（对应 `assets/themes/<name>.json`） |
| `--image-style` | 配图风格名（对应 `assets/image-styles/<name>.json`） |
| `--account` | 指定公众号账号（对应 `wechat-publisher.yaml` 的账号名） |
| `--ai-score-threshold` | "AI 味"检测阈值（默认与 `ai_score.DEFAULT_THRESHOLD` 一致） |
| `--skip-ai-score` | 跳过 AI 味检测 |
| `--allow-missing-images` | 允许部分图片上传失败仍继续发布 |
| `--debug` | 把生成的 HTML 保存到临时目录便于排查 |

> 缺图守卫：若文章引用了图片但未成功上传，默认*中止发布*避免产出缺图草稿；显式传 `--allow-missing-images` 才继续。

## 排版主题与配图风格

- **排版主题**（`assets/themes/`）：`academic-paper` · `business-navy` · `elegant-ink` · `girly-pink` · `ink-wash` · `magazine-grid` · `minimal-bw` · `minimal-mono` · `mint-fresh` · `news-bold` · `refined-blue` · `sage-premium` · `sunset-coral` · `warm-editorial` · `warm-orange`。默认：`refined-blue`（main 账号）、`minimal-mono`（tech 账号）。可视化对比见 [`assets/theme-previews/`](assets/theme-previews/)。
- **配图风格**（`assets/image-styles/`）：共 22 种，覆盖手绘手账、叙事漫画、荧光大字卡、卡片排版、数据图表、高密度水彩信息图等。文章默认 `warm-handdrawn`，贴图默认 `infographic-warm`。列表与用法见 [`assets/image-styles/README.md`](assets/image-styles/README.md)。

列出全部配图风格（可用于 `--image-style`）：

```bash
python3 scripts/wechat_api.py list-image-styles
```

## 检查与测试

```bash
python -m py_compile scripts/*.py
python -m pytest tests -q
```

## 安全说明

- Skill 将微信凭据保存在本地 `wechat-publisher.yaml`（已被 git 忽略）。
- `scripts/.token_cache*` 令牌缓存文件也被 git 忽略。
- 创建草稿会上传正文和图片到微信 API。
- 本 fork **不读取浏览器 Cookie**，也**不做第三方平台同步**。
- 更多接口细节见 [`references/api_reference.md`](references/api_reference.md)（token 获取、封面/正文图片上传、新建草稿、常见错误码与频率限制）。

## 如何自定义 / 扩展

- **加一套排版主题**：在 `assets/themes/` 下新增一份 `<name>.json`，目录内 HTML 预览可放 `assets/theme-previews/<name>.html`。
- **加一种配图风格**：在 `assets/image-styles/` 下复制现有 `.json` 改 `style_name` / `display_name` / `description` 与 `prompt_template`，再生成一张 `previews/<name>.webp` 预览图。具体步骤见 `assets/image-styles/README.md`。
- **改每账号默认**：在 `wechat-publisher.yaml` 的对应账号里改 `theme`、`image_style`、`newspic_image_style`；单篇可用 `--theme` / `--image-style` 覆盖。
- **调整"AI 味"严格度**：改 `ai_score.DEFAULT_THRESHOLD`，或用 `--ai-score-threshold` / `--skip-ai-score` 控制是否/多严地拦截。
- **改 Skill 行为**：编辑 `SKILL.md`（流程、边界、何时使用）即可调整 Agent 侧的编排方式。

## 上游与相关

- 上游：本仓库 fork 自 [jiji262/wechat-publisher](https://github.com/jiji262/wechat-publisher)，并裁剪为更安全的"仅建草稿"流程。
- [match-resumes-to-jd](https://github.com/onlyforchris/match-resumes-to-jd)：根据 JD 建立岗位画像并持续筛选简历的 AI Skill。
- [bid-response-production](https://github.com/onlyforchris/bid-response-production)：生成、审查、规范化中文投标材料的 Skill。
