# CDNBuddy 🟣

> **Auto-Optimize. Auto-Sync. Zero Config.**

Background CDN sync & image optimization daemon. Watches your upload directories, auto-converts images to WebP at 4 responsive sizes, uploads to S3/CloudFront CDN, keeps local fallback copies. Set it and forget it.

[![version](https://img.shields.io/badge/version-1.0.0-purple)](https://cdnbuddy.boottify.com)
[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## Features

- 🔄 **Auto-watch** — monitors directories via inotify, processes new uploads in seconds
- 🖼️ **WebP conversion** — on-the-fly WebP at 4 sizes (200, 480, 768, 1280px)
- ☁️ **CDN sync** — uploads to S3/CloudFront with `Cache-Control: immutable`
- 📦 **Local fallback** — variant files saved locally alongside originals
- 📊 **API + CLI** — REST API at `localhost:8765` + full CLI
- 🗄️ **SQLite tracking** — every image, variant, and sync event logged
- ⚡ **Zero config** — drop it in, point at a directory, it works

## Quick Start

```bash
# Install
pip install cdnbuddy

# Start watching
cdnbuddy watch /var/www/uploads

# Or sync once
cdnbuddy sync /var/www/uploads

# Check status
cdnbuddy status

# Start API server
cdnbuddy serve --port 8765
```

## Configuration

Config lives at `~/.cdnbuddy/config.json`:

```json
{
  "watch_dirs": ["/var/www/uploads"],
  "s3_bucket": "aws-boottify",
  "s3_region": "eu-central-1",
  "cdn_base_url": "https://cdn.boottify.com",
  "local_base_path": "/var/www",
  "quality": 85,
  "poll_interval": 10,
  "api_port": 8765
}
```

Environment variables:
- `CDNBUDDY_HOME` — config/data directory (default `~/.cdnbuddy`)
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` — S3 credentials (boto3)

## API Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/status` | GET | Overall sync status, counts, recent logs |
| `/api/sync` | POST | Trigger a full sync scan |
| `/api/process` | POST | Process a single image by path |
| `/api/images` | GET | List tracked images (with status filter) |
| `/api/image/{id}` | GET | Get image detail + all variants |

## Responsive Sizes

| Label | Width | Typical Use |
|-------|-------|-------------|
| `xs` | 200px | Thumbnails, cards |
| `sm` | 480px | Mobile, list views |
| `md` | 768px | Tablet, feeds |
| `lg` | 1280px | Hero, full-width |

Images smaller than a size target are skipped (no upscaling).

## Systemd Service

```ini
# ~/.config/systemd/user/cdnbuddy.service
[Unit]
Description=CDNBuddy — CDN sync & image optimization daemon
After=network-online.target

[Service]
Type=simple
ExecStart=%h/.local/bin/cdnbuddy watch
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

```bash
systemctl --user enable --now cdnbuddy
```

## CLI Commands

```
cdnbuddy watch <dir>       Start watching (auto-sync on new files)
cdnbuddy sync <dir>        Manual full sync
cdnbuddy process <file>    Process single image
cdnbuddy status            Show sync status + recent errors
cdnbuddy serve             Start API server
cdnbuddy install           Generate systemd service file
```

## How It Works

1. CDNBuddy watches your upload directories
2. New PNG/JPG → generates 4 WebP variants at responsive widths
3. Each variant uploaded to S3/CloudFront CDN
4. Local copy kept as fallback (`image-xs.webp`, `image-md.webp`, etc.)
5. SQLite DB tracks every image + variant for dedup

## License

MIT — see [LICENSE](LICENSE)

---

Built by [Boottify](https://boottify.com) — part of the Boottify developer tools ecosystem.
