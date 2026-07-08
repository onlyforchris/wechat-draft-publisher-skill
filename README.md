# wechat-draft-publisher-skill

Safe WeChat Official Account draft publisher skill.

This fork keeps the useful local workflow:

- Convert Markdown to WeChat-compatible HTML.
- Upload article and cover images to WeChat.
- Create a draft with the WeChat Official Account API.
- Optionally run the local `ai_score.py` writing check before upload.

This fork intentionally removes:

- Browser-cookie based Gemini Web automation.
- Wechatsync / multi-platform synchronization.
- Any direct mass-send flow. The script creates drafts only.

## Install

```bash
git clone https://github.com/onlyforchris/wechat-draft-publisher-skill.git
cd wechat-draft-publisher-skill
pip install requests pyyaml
```

Optional, only if you use image generation:

```bash
# requires bun and API credentials configured in wechat-publisher.yaml
python scripts/generate_image.py --help
```

## Configure

```bash
cp wechat-publisher.yaml.example wechat-publisher.yaml
```

Edit `wechat-publisher.yaml`:

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

Do not commit `wechat-publisher.yaml`. It contains `app_secret`.

## Publish A Draft

```bash
python scripts/publish.py \
  --account main \
  --input article.md \
  --cover cover.jpg \
  --title "Article title" \
  --digest "Short summary"
```

The result is a WeChat draft `media_id`. Log in to `mp.weixin.qq.com` to review and publish manually.

For already formatted HTML:

```bash
python scripts/publish.py --html article.html --cover cover.jpg --title "Article title"
```

## Checks

```bash
python -m py_compile scripts/*.py
python -m pytest tests -q
```

## Security Notes

- The skill stores WeChat credentials locally in `wechat-publisher.yaml`.
- Token cache files under `scripts/.token_cache*` are git-ignored.
- Draft creation uploads content and images to WeChat APIs.
- No browser cookies are read by this fork.
- No third-party platform sync is included in this fork.

## Upstream

Forked from `jiji262/wechat-publisher`, then reduced to a safer draft-only workflow.
