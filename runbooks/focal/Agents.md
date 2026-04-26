# LM Studio

Download the AppImage from the official website (`LM-Studio-0.4.2-2-x64.AppImage`).

Install the AppImage.

```
mv LM-Studio-0.4.2-2-x64.AppImage ~/.local/bin/LM-Studio.AppImage
chmod 755 ~/.local/bin/LM-Studio.AppImage
```

Extract the icon from the AppImage.

```
~/.local/bin/LM-Studio.AppImage --appimage-extract
cp squashfs-root/lm-studio.png ~/.local/share/icons/lm-studio.png
rm squashfs-root -rf
```

Create `~/.local/share/applications/LM-Studio.desktop`.

```
[Desktop Entry]
Name=LM Studio
Exec=/home/user/.local/bin/LM-Studio.AppImage --no-sandbox
Icon=/home/user/.local/share/icons/lm-studio.png
Type=Application
```

Run LM Studio. Download `gpt-oss:20b`. Increase the context window to 16384 before loading the model. Set the reasoning to high. Enable the local server.

# OpenCode

Install with `curl -fsSL https://opencode.ai/install | bash`.

Configure `~/.config/opencode/opencode.json` as follows

```
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": {
        "baseURL": "http://localhost:1234/v1"
      }
    }
  }
}
```

Install `xclip` to support copying to the clipboard.

```
sudo apt install xclip
```

As of `1.1.53`, OpenCode works reasonably well.

# Models

I have reasonable success with `google/gemma-4-26b-a4b`.

- Increase the context window to the maximum (262144 tokens).
- Decrease GPU offload to 10.
