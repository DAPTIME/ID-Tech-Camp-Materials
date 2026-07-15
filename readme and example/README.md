# Devansh AI

A tiny SLM (Small Language Model), runs entirely on some Jetson Orin Nanos sitting on my desk. No cloud, no API keys, no phoning home for someone else to boot up your custom made AI. Just a little 1B pramiter brain that boots up, looks you in the eye, and innocently says "Hello! How may I assist you today?", just happy to help you.

That's the whole point of it, and honestly? I'm a little obsessed with it.

## What this actually is

Devansh AI is a fine tuned version DAP AI lite, also made by me but smarter. I made DAP AI to publish to google for collages to see but I didn't know it would end up being as successful as it is today. Here is the link to DAP AI: [DAP AI Website](https://dapai.example.com) . It took me 3 years to make, I made the model, but I was too lazy to make the website so lovable made that for me. The model runs locally on my 2 Nvidia Jetson Orin Nano Dev Kits and I can't access any of your daa if you use it. Its off most of the time but soon, I will be able to keep it on almost indefinatly due to new model acetacture I will use to create a new model. Both DAP AI and Devansh AI take images and text in and give text output, sadly no image generation, that would take months.

Everything happens on the Orin Nanos. The training, the merge, the quantization, all of it, on 2 dev kit with 8GB of memory and low power consumtion ( No water consumption like a datacenter ). Nothing here ever touched a datacenter GPU, touched a big company, or had to make anyone pay ransom for their data.

Also a small disclamer this model doesn't work in LM Studio as LM Studio breaks some of the models dependencies like bittsandbytes

## Why I built it

I wanted to know if I start off with command prompt and an empty vs code terminal, and make it mine. Not rent it, not call it over an API, but own the weights and bend them to my will on hardware I can hold in one hand.

There's something about asking a model who made you and having it answer with your name, offline, on a board that pulls less power than a light bulb. It isn't doing anything a giant model can't do better. That was never the point. The point was doing it small, doing it local, and doing it myself instead of some big company hogging up my data and making me pay ransom for it.

The Orin made me earn it. Bitsandbytes refused to build for the Tegra GPU, so quantized training was out and everything ran in bf16. But Devansh AI is based off DAP AI which is trained in FP 8. Keep in mind the DAP AI Pro model is still behind a pay wall, 1.5 dollars for lifetime pro (I need to make up to the electricity costs somehow). The 8GB is shared between CPU and GPU, which turns "just load the model" into an actual engineering decision. The GGUF conversion fought me over a one token mismatch between the tokenizer and the embedding matrix that took an embarrassing amount of time to find. I lost an entire evening to the RAM gobbler, Chrome, of all things so I switched to firefox and kept going.

Every one of those was a small war, and winning them is why this repo means something to me. It isn't a wrapper around someone else's endpoint. It's a full pipeline I understand end to end because I broke and fixed every piece of it myself.

## How it works

It's four stages, all of them on the Orin.

First, the identity data. A compact set of question and answer pairs that teach the model its name, who built it, and how to turn down being ChatGPT or Qwen and defo not Copilot (Because nobody likes copilot) .

Then LoRA. Instead of retraining a billion parameters, LoRA drops small trainable adapters into the language model's attention layers, so only a fraction of a percent of the model actually changes. Light enough to fit in memory, fast enough to cover in minutes.

Next, I train the AI model to make it smarer because, to be honest, the only way to have a very smart AI on DAP AI is to have the pro plan and I didn't want to submit a kinda smart, average, free AI model.

Then the merge, where the adapter gets folded into the base weights. This is the part that matters. It means the identity lives in the model itself rather than being bolted on at runtime by a prompt.

Then quantization to GGUF, down to BF16 which is acctually a lack of quantinization, Q8_0, Q6_K, and Q4_K_M, small and fast enough to serve locally. So we drink out water instead of a random GPU in the US.

## Example Video:

https://github.com/user-attachments/assets/7d1a90bf-6c91-4559-9b46-1fdea1fcedbc
