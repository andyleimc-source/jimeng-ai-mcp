# jimeng-ai-mcp

**MCP server for Jimeng AI (即梦 · ByteDance)** — Let Claude, Cursor, and other AI assistants generate images and videos directly through Jimeng's official Volcengine API.

[中文](README.md) | English

---

## What is this?

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/) is an open standard by Anthropic that lets AI models call external tools and services through a standardized interface.

**jimeng-ai-mcp** wraps [Jimeng AI](https://jimeng.jianying.com/)'s official Volcengine API as an MCP server. Once connected, you can drive Jimeng to generate images and videos through natural language in Claude, Cursor, or any MCP-compatible client — no browser switching, no manual uploads, results saved locally.

## What can it do?

### Image

| Tool | Description | Model |
|---|---|---|
| `generate_image` | Text-to-image, supports 4.0 / 4.6 / 3.x models | Jimeng Image Series |
| `image_to_image` | Image editing with text guidance | jimeng_i2i 3.0 |
| `inpaint_image` | Local inpainting / object removal | inpaint |
| `upscale_image` | AI upscaling to 4K / 8K | Seedream SR |

### Video

| Tool | Description | Model |
|---|---|---|
| `generate_video` | Text-to-video, 720P / 1080P / Pro | Jimeng Video 3.0 |
| `image_to_video` | Image-to-video: first frame, first+last frame, camera movement, Pro | Jimeng Video 3.0 |
| `imitate_motion` | Motion imitation: transfer motion from reference video to target person | DreamActor 2.0 |
| `generate_digital_human` | Talking head: image + audio → lip-synced video | OmniHuman 1.5 |
| `translate_video` | Video translation with voice preservation and lip-sync | Translation 2.0 |

## Quick Start

### Step 1: Enable the service

1. Log in to the [Volcengine Console](https://console.volcengine.com/ai/ability/detail/2) and enable **Jimeng AI**
2. Get your Access Key ID and Secret Access Key from [IAM Key Management](https://console.volcengine.com/iam/keymanage/)

### Step 2: Install

Recommended — run with `uvx` (no install needed):

```bash
uvx jimeng-ai-mcp
```

Or install with pip:

```bash
pip install jimeng-ai-mcp
```

### Step 3: Configure your MCP client

#### Claude Desktop

Config file location:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "jimeng": {
      "command": "uvx",
      "args": ["jimeng-ai-mcp"],
      "env": {
        "JIMENG_ACCESS_KEY_ID": "your_access_key_id",
        "JIMENG_SECRET_ACCESS_KEY": "your_secret_access_key"
      }
    }
  }
}
```

#### Cursor

Config file: `~/.cursor/mcp.json`

```json
{
  "mcpServers": {
    "jimeng": {
      "command": "uvx",
      "args": ["jimeng-ai-mcp"],
      "env": {
        "JIMENG_ACCESS_KEY_ID": "your_access_key_id",
        "JIMENG_SECRET_ACCESS_KEY": "your_secret_access_key"
      }
    }
  }
}
```

> After restarting the client, just type in the chat: *"Generate a cyberpunk cityscape, 16:9 widescreen."*

### Local development

```bash
git clone https://github.com/andyleimc-source/jimeng-ai-mcp
cd jimeng-ai-mcp
cp .env.example .env
# Fill in your keys in .env
uv sync
uv run jimeng-mcp
```

Local debug (start MCP Inspector):

```bash
uv run mcp dev src/jimeng_mcp/server.py
```

## Examples

**Text-to-image**
> Generate a Song Dynasty landscape painting, 16:9, using the Jimeng 4.0 model

**Image-to-image**
> Replace the background of this product photo with a clean white studio style, keep the product unchanged
> [attach image URL]

**Text-to-video**
> Generate a 10-second video: a fox running in the snow, slow motion, cinematic, 16:9

**Image-to-video (first + last frame)**
> I have two images, generate a video driven by first and last frames. Opening: this sunrise photo, ending: this sunset photo, 5 seconds
> First frame: [URL1], last frame: [URL2]

**Digital human**
> Use this portrait image and this audio to generate a lip-synced digital human video, output 1080P
> Image: [URL], Audio: [URL]

**Video translation**
> Translate this Chinese video to English, with lip-sync
> [video URL]

## Tool Parameters

### `generate_image`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | required | Image description |
| `model` | string | `jimeng_t2i_v40` | `jimeng_t2i_v40` / `jimeng_seedream46_cvtob` / `jimeng_t2i_v31` / `jimeng_t2i_v30` |
| `aspect_ratio` | string | `1:1` | `1:1` / `16:9` / `9:16` / `4:3` / `3:4` / `3:2` / `2:3` |
| `quality` | string | `2k` | `2k` or `normal` |
| `negative_prompt` | string | `""` | Content to exclude |

### `generate_video`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | required | Video description |
| `model` | string | `jimeng_t2v_v30` | `jimeng_t2v_v30` (720P) / `jimeng_t2v_v30_1080p` (1080P) / `jimeng_ti2v_v30_pro` (Pro) |
| `aspect_ratio` | string | `16:9` | `16:9` / `4:3` / `1:1` / `3:4` / `9:16` / `21:9` |
| `duration_sec` | int | `5` | `5` or `10` seconds |

### `image_to_video`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `image_url` | string | required | First frame image URL |
| `prompt` | string | required | Video description |
| `mode` | string | `first` | `first` / `first_1080p` / `first_tail` / `first_tail_1080p` / `recamera` / `pro` |
| `duration_sec` | int | `5` | `5` or `10` seconds |
| `tail_image_url` | string | `""` | Last frame image (required for `first_tail` / `first_tail_1080p` mode) |

### `translate_video`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `video_url` | string | required | Source video URL |
| `target_language` | string | required | Target language code |
| `src_language` | string | `zh` | Source language code |

Language codes: `zh` Chinese · `en` English · `ja` Japanese · `ko` Korean · `fr` French · `de` German · `es` Spanish · `pt` Portuguese · `ru` Russian · `ar` Arabic · `it` Italian

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `JIMENG_ACCESS_KEY_ID` | ✅ | Volcengine Access Key ID |
| `JIMENG_SECRET_ACCESS_KEY` | ✅ | Volcengine Secret Access Key |
| `JIMENG_OUTPUT_DIR` | optional | Output directory, default `~/jimeng-output` |

## Resources

- [Jimeng AI Open Platform — API Overview](https://www.volcengine.com/docs/85621/1544716?lang=zh)
- [Jimeng AI Official Site](https://jimeng.jianying.com/)
- [Volcengine Console](https://console.volcengine.com/ai/ability/detail/2)
- [MCP Protocol Spec](https://modelcontextprotocol.io/)
- [Claude Desktop](https://claude.ai/download)
- [Volcengine Signature Mechanism](https://www.volcengine.com/docs/6369/65)

## License

[MIT](LICENSE)
