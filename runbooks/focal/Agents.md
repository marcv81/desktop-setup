# llama.cpp

## Build

Clone and build.

    git clone https://github.com/ggml-org/llama.cpp
    cd llama.cpp
    cmake -B build -DGGML_CUDA=ON
    cmake --build build --config Release -j 16

## Models

Run `gemma-4-26b` MoE.

    ./build/bin/llama-server \
      -hf ggml-org/gemma-4-26B-A4B-it-GGUF:Q4_K_M \
      --alias gemma-4-26b-a4b \
      -ngl all \
      -ot "exps=CPU" \
      -c 262144

Run `qwen3.6-35b` MoE.

    ./build/bin/llama-server \
      -hf unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q4_K_M \
      --alias qwen3.6-35b-a3b \
      -ngl all \
      -ot "exps=CPU" \
      -c 262144

# OpenCode

Install with `curl -fsSL https://opencode.ai/install | bash`.

Configure `~/.config/opencode/opencode.json` as follows

```
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"],
  "provider": {
    "llama.cpp": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp (local)",
      "options": {
        "baseURL": "http://localhost:8080/v1"
      },
      "models": {
        "llama.cpp": {
          "name": "llama.cpp"
        }
      }
    }
  }
}
```

Install `xclip` to support copying to the clipboard.

```
sudo apt install xclip
```
