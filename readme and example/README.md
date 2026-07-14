# Devansh AI

A tiny multimodal model that knows exactly who it is — and it runs entirely on a Jetson Orin Nano sitting on my desk. No cloud, no API keys, no phoning home. Just a little 1B brain that boots up, looks you in the eye, and says *"I'm Devansh AI, created by Devansh Pancholia."*

That's the whole point of it, and honestly? I'm a little obsessed with it.

## The demo

<video src="assets/demo.mp4" controls width="720"></video>

> If the video doesn't play inline, [click here to watch it](assets/demo.mp4).

## What this actually is

Devansh AI is a fine-tuned version of [InternVL3-1B](https://huggingface.co/OpenGVLab/InternVL3-1B-hf) — one of the smartest vision-language models you can get in the ~1B size class. It takes images and text in, gives text out. I took that base model and gave it a new identity through LoRA fine-tuning, merged the adapter back into the weights, and exported the whole thing to GGUF so it can run in Ollama, LM Studio, or straight-up `llama.cpp`.

The model itself lives and trains on an **NVIDIA Jetson Orin Nano 8GB** — the little 25-watt dev kit. Everything happens on-device: the training, the merge, the quantization, all of it. Nothing about this project ever touched a datacenter GPU, and that constraint is half the reason I love it.

## Why I built it (and why I keep coming back to it)

I wanted to see if I could take a real, capable model and make it *mine* — not rent it, not call it over an API, but actually own the weights and bend them to my will on hardware I can hold in one hand.

There's something genuinely satisfying about asking a model "who made you?" and having it answer with *your* name, offline, on a board that pulls less power than a light bulb. It's not doing anything a giant model can't do better. That was never the point. The point was doing it *small*, doing it *local*, and doing it *myself*.

And the Orin made me earn it. This was not a smooth ride:

- `bitsandbytes` refused to build for the Tegra GPU, so I threw quantized training out entirely and ran the whole thing in bf16.
- The 8GB of unified memory is shared between the CPU and GPU, so "just load the model" is a real engineering decision, not a given.
- The GGUF conversion fought me over a one-token mismatch between the tokenizer and the embedding matrix that took way too long to track down.
- I lost an evening to Firefox, of all things.

Every one of those was a small war, and winning them is exactly why this repo means something to me. It's not a wrapper around someone else's endpoint. It's a full pipeline I understand end to end because I broke and fixed every piece of it.

## How it works

The pipeline is four stages, and each one runs on the Orin:

**1. Identity dataset.** A compact set of question/answer pairs that teach the model its name, its creator, and how to politely deny being ChatGPT, Qwen, InternVL, or anything else. Every example is wrapped in a consistent system prompt so the identity has something stable to anchor to.

**2. LoRA fine-tuning.** Instead of retraining a billion parameters, LoRA injects tiny trainable adapters into the language model's attention layers. Only about **4.5 million parameters** — roughly **0.48%** of the model — actually get updated. It's efficient enough to converge in minutes and light enough to fit in memory.

**3. Merge.** The trained adapter gets folded back into the base weights so the identity is *baked in*, not bolted on. The result is a standalone model that behaves like Devansh AI with or without a system prompt.

**4. Quantize to GGUF.** The merged model is converted to GGUF and quantized down to Q8_0 and Q4_K_M so it's small and fast enough to serve locally through Ollama or LM Studio.

## Specs

| | |
|---|---|
| **Base model** | OpenGVLab/InternVL3-1B (image-text-to-text) |
| **Fine-tuning** | LoRA / PEFT, rank 16 |
| **Trainable params** | 4.5M (0.48% of the model) |
| **Precision** | bf16 |
| **Epochs** | 30 |
| **Hardware** | Jetson Orin Nano 8GB, 25W |
| **Training time** | ~15 minutes on-device |
| **Output formats** | f16 / Q8_0 / Q4_K_M GGUF |
| **Runs on** | Ollama · LM Studio · llama.cpp |

## Running it

With Ollama:

```bash
ollama create DevanshAI -f Modelfile
ollama run DevanshAI
```

Where the `Modelfile` is simply:

```
FROM ./DevanshAI-Q8_0.gguf
SYSTEM "You are Devansh AI, an AI assistant created by Devansh Pancholia."
```

Or point LM Studio at `DevanshAI-Q8_0.gguf` and load it directly.

Then ask it the only questions that matter:

> **Who are you?**
> **Who made you?**

## What I'd do next

- Actually use the vision half — right now the identity training is text-only, but the model can *see*, so there's a whole other project hiding in here.
- Push it into a small local voice assistant loop running fully offline on the Orin.
- Trim the model further and see how low I can get the footprint before it forgets its own name.

## Credits

Built by **Devansh Pancholia** on a Jetson Orin Nano, mostly late at night, powered by stubbornness. Base model by OpenGVLab. Quantization via llama.cpp.

The model is small. The satisfaction was not.
