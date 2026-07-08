---
name: wechat-draft-publisher
description: Create WeChat Official Account draft articles from Markdown or prepared HTML. Use when the user explicitly asks to prepare or upload a WeChat Official Account draft. This skill creates drafts only; it does not mass-send, read browser cookies, or sync to other platforms.
---

# WeChat Draft Publisher

Use this skill only when the user explicitly asks for WeChat Official Account draft work.

## Boundaries

- Create drafts in WeChat Official Account only.
- Do not mass-send or publish publicly.
- Do not read browser cookies.
- Do not sync to Zhihu, Juejin, CSDN, Weibo, Xiaohongshu, or other platforms.
- Do not invent credentials. Require `wechat-publisher.yaml` with `app_id` and `app_secret`.

## Configuration

Copy the example config once:

```bash
cp wechat-publisher.yaml.example wechat-publisher.yaml
```

Required account fields:

```yaml
default: main
accounts:
  main:
    name: "My Official Account"
    app_id: "wx..."
    app_secret: "..."
    author: "Author"
    theme: "refined-blue"
```

`wechat-publisher.yaml` is ignored by git. Never print or commit `app_secret`.

## Markdown Draft Flow

1. Write or receive `article.md`.
2. Ensure a cover image exists.
3. Run:

```bash
python scripts/publish.py \
  --account main \
  --input article.md \
  --cover cover.jpg \
  --title "Title" \
  --digest "Digest"
```

The command returns a draft `media_id`. Tell the user to review it in `mp.weixin.qq.com` before publishing.

## HTML Draft Flow

Use this when HTML is already formatted:

```bash
python scripts/publish.py --account main --html article.html --cover cover.jpg --title "Title"
```

## Optional Checks

Run before publishing when editing text:

```bash
python scripts/ai_score.py article.md --threshold 45
```

Run basic code checks after modifying this skill:

```bash
python -m py_compile scripts/*.py
python -m pytest tests -q
```

## Removed From This Fork

- Gemini Web / browser-cookie image generation.
- Wechatsync multi-platform synchronization.
- Direct public publishing / mass send.
