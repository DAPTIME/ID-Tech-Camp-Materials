# Devansh AI

A tiny SLM (Small Language Model), runs entirely on some Jetson Orin Nanos sitting on my desk. No cloud, no API keys, no phoning home for someone else to boot up your custom made AI. Just a little 1B pramiter brain that boots up, looks you in the eye, and innocently says "Hello! How may I assist you today?" and just exited to help you.

That's the whole point of it, and honestly? I'm a little obsessed with it.

## What this actually is

Devansh AI is a fine tuned version DAP AI lite, also made by me but smarter. I made DAP AI to publish to google for collages to see but I didn't know it wuld end up being as sucsessful as it is today. Here is the link to DAP AI: [DAP AI WEbsite](https://dapai.lovabke.app) . It took me 3 year to make, I make the model, but I was too lazy to make the website so lovable made that for me. The model runs locally on my 2 Nvidia Jetson Orin Nano Dev Kits and I can't access any of your daa if you use it. Its off most of the time but soon, I will be able to keep it on almost indefinatly due to new model acetacture I will use. Both DAP AI and Devansh AI take images and text in, gives text out.

Everything happens on the Orin Nanos. The training, the merge, the quantization, all of it, on a 25 watt dev kit with 8GB of memory. Nothing here ever touched a datacenter GPU, touched a big company, or had to make anyone pay ransom for their data.
## Why I built it

I wanted to know if I could take a real model and make it mine. Not rent it, not call it over an API, but own the weights and bend them to my will on hardware I can hold in one hand.

There's something about asking a model who made you and having it answer with your name, offline, on a board that pulls less power than a light bulb. It isn't doing anything a giant model can't do better. That was never the point. The point was doing it small, doing it local, and doing it myself instead of some big company hogging up my data and making me pay ransom for it.

The Orin made me earn it. bitsandbytes refused to build for the Tegra GPU, so quantized training was out and everything ran in bf16. The 8GB is shared between CPU and GPU, which turns "just load the model" into an actual engineering decision. The GGUF conversion fought me over a one token mismatch between the tokenizer and the embedding matrix that took an embarrassing amount of time to find. I lost an entire evening to Firefox, of all things.

Every one of those was a small war, and winning them is why this repo means something to me. It isn't a wrapper around someone else's endpoint. It's a full pipeline I understand end to end because I broke and fixed every piece of it myself.

## How it works

It's four stages, all of them on the Orin.

First, the identity data. A compact set of question and answer pairs that teach the model its name, who built it, and how to  turn down being ChatGPT or Qwen and defo not Copilot (Cause I want people to like my AI, but nobody likes copilot) .

Then LoRA. Instead of retraining a billion parameters, LoRA drops small trainable adapters into the language model's attention layers, so only a fraction of a percent of the model actually changes. Light enough to fit in memory, fast enough to converge in minutes.

Next, I train the AI model to make it smarer because, to be honest, the only way to have a very smart AI on DAP AI is to hae the pro plan and I didn't want to submit a somewhat dumb, free model.

Then the merge, where the adapter gets folded into the base weights. This is the part that matters. It means the identity lives in the model itself rather than being bolted on at runtime by a prompt.

Then quantization to GGUF, down to Q8_0 and Q4_K_M, small and fast enough to serve locally.
