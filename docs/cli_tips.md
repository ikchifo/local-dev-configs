# CLI tips

## ffmpeg

### Compress video for smaller file size

```bash
ffmpeg -y -i input.mp4 \
  -c:v libx264 -crf 28 -preset slow \
  -vf "scale=1280:-2" -r 30 \
  -c:a aac -b:a 64k \
  output.mp4
```

| Flag | Purpose |
|------|---------|
| `-y` | Overwrite output without asking |
| `-i` | Input file |
| `-c:v libx264` | Video codec: H.264 |
| `-crf 28` | Quality (0-51, lower=better, 23=default) |
| `-preset slow` | Slower encoding = better compression |
| `-vf "scale=1280:-2"` | Scale width to 1280px, auto height (divisible by 2) |
| `-r 30` | Output framerate |
| `-c:a aac` | Audio codec: AAC |
| `-b:a 64k` | Audio bitrate |
