---
title: AI language translator with Flask-based web interface - Part 1
author: petman
date: 2025-01-11 10:10:00 +0800
categories: [AI, Fairseq, Transformers, Flask, DIY]
tags: [ai, fairseq, seq2seq, transformers, web, diy]
render_with_liquid: false
---

Happy New Year, and welcome back! In this series, we will build a simple language translation web app based on a pre-trained AI model. For this, we will be using Fairseq: [Facebook AI Research Sequence-to-Sequence](https://github.com/facebookresearch/fairseq) toolkit, which allows to train and use custom models for translation, summarization, language modeling and other text generation tasks. This tutorial will focus solely on leveraging pre-trained models for building something useful, rather than on the theory of sequence-to-sequence models and machine learning in general.

## System setup
- 