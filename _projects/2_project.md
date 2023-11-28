---
layout: page
title: LocalWhisper
description: Run Whisper locally on your machine
img: assets/img/whisper.png
importance: 1
category: 
---

This project is a fork of G. Gerganov's awesome Whisper.cpp project. My code is public here: [LocalWhisper](https://github.com/chinardankhara/localwhisper) and the original project is here: [Whisper.cpp](https://github.com/ggerganov/whisper.cpp).

I wanted to test the limits of locally run large models. I strongly believe in the future of LLMs as being one of edge inference where users control their data and the model weights. I selected OpenAI's Whisper transcription model as a starting since I needed the model for other uses already and it has a rich open source ecosystem with fasterwhisper, distilwhisper, WhiserJAX, and whisper.cpp open source projects. The original unoptimized model requires over 6 GB of of storage and 9 GB of RAM. It also runs with any acceptable speed only on a GPU. I defined two goals: reduce storage and RAM size by 75% and run on a CPU with acceptable speed. I tried the four major open source projects and decided whisper.cpp would be the best starting point.

Because of its pure C++ architecture powered by the GGUF framework, whisper.cpp runs on a CPU with faster than real-time performance. I started with the large model and used GGUF quanitzation with the q4_0 scheme. What this means is that most model weights were dropped usin an activation aware quantization technique. The dropped weights are usually the ones with least information content and due the way deep models tend to work, performance loss is minimal when pruning the model until it drops suddenly. After many experiments, I landed with the medium model with q4_1 quantization and a batched streaming service that enables real-time transcription. The final application size is 600 MB with RAM usage 1.1 GB. CPU usage experiments are pending but the load on my Macbook M2 Air was minimal. The model has a real-time factor of 0.4.