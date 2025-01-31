---
title: Running AI models in your terminal
author: petman
date: 2025-01-31 10:00:00 +0800
categories: [Artificial intelligence, Large language models]
tags: [ai, llms, ollama]
render_with_liquid: false
---

If you haven't been living under a rock for the past couple of years, you're probably familiar with terms like `ChatGPT` or `DeepSeek`. 
These are popular *artificial intelligence*  models which have recently taken the internet by storm. The more technical term for these AI models is **large language models** (LLMs), which belong to a specific family of machine learning models designed for [natural language processing](https://www.ibm.com/think/topics/natural-language-processing) tasks such as text generation, summarization, and language translation.

The purpose of this post is not to delve into the theory behind LLMs as there are a myriad of articles out there that already cover this. 
Rather, we will focus on how to download and interact with some of the most popular LLMs directly from a PC terminal.

## Downloading LLMs
- It may come as a surprise to many, but there are numerous pre-trained open-source LLMs available for download and local setup. The most popular platform for this is [Hugging Face](https://huggingface.co/), which hosts a vast collection of AI models, ranging from computer vision to natural language processing.
Hugging Face equally provides APIs (e.g., in Python) which allow you to easily integrate these models into your applications.

- Recently, I stumbled on [Ollama](https://ollama.com/), an even charmier framework that lets you download and run LLMs in your terminal with fewer than five lines of code--ideal for the lazy programmer that I am (and probably you too!). So, for this tutorial, we will focus on Ollama.

- To download and install Ollama locally, follow the instructions on their website [here](https://ollama.com/download). For Linux users, you can simply run the following command in your terminal:
  
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
- Once Ollama is installed, you can download (i.e., `pull`) and run a model locally by using the command `ollama run <model-name>`. For example, to pull DeepSeek R1, enter the following command in your terminal:
  
```bash
ollama run deepseek-r1
```
- This downloads the model to your local system and opens a prompt (as shown below), allowing you to interact with the model. 
![DeepSeek R1 Linux Prompt](assets/imgs/llms/deepseek-ollama.png)

- Enter a query like `what is an LLM` and DeepSeek will generate a response for you!
![DeepSeek what is an LLM? ](assets/imgs/llms/deepseek-what-is-an-llm.png)

- To summarize a large text file, do:

```bash
ollama run deepseek-r1 "Summarize the following text:" < document.txt
```

- To use other models like [Microsoft's Phi4](https://ollama.com/library/phi4) or [Meta's LLama3](https://ollama.com/library/llama3.3), just head to [Ollama](https://ollama.com/search) and find the exact name to use with `ollama run`.
  
- Personally, I find this approach way cooler than opening a web page every time I need to run queries. Even better, you can have multiple models available offline! 

> If you're concerned about privacy, running LLMs locally can be a great option, as it allows you to avoid sending data to cloud services where you have no control over who accesses or handles it.
{: .prompt-warning }


## Model sizes
- If you visited the [Ollama website](https://ollama.com/search), you have likely noticed that the models have various sizes. 
![DeepSeek R1 sizes](assets/imgs/llms/deepseek-7b.png)

For example, DeepSeek R1 is available in multiple sizes:  `1.5b`, `7b`, ..., `671b`. These numbers represent the number of **parameters** in the model. So `1.5b` corresponds to the smallest model with **1.5 billion parameters**, while `671b` is the largest model with **671 billion** parameters. A helpful way to understand these numbers is to think of them as analogous to "IQ scores"--the larger the number, the "smarter" the model, and vice versa. 

- That said, running the largest DeepSeek R1 models (like `671b`) requires additional hardware like [GPUs](https://www.ibm.com/think/topics/gpu) to handle the computational load. For a regular CPU-only computer, it's best to stick with the smaller models, ranging from `1.5b` to around `32b`. 

## Ollama commands
- Here are other practical commands for using Ollama.
  
```bash
ollama pull <model-name> # Downloads the model to your local system without running it
ollama list              # Lists all downloaded models
ollama rm <model-name>   # Removes/deletes the model locally
Ctrl + C                 # Stops text generation by the model  
Ctrl + D                 # Closes the interactive prompt
/bye                     # Same as Ctrl + D: closes the interactive prompt
```

## Using Ollama in a Python program
- If you're a programmer like myself, you might also be wondering how to integrate these models into a Python program. Thankfully, the Ollama framework provides tools to do just that. 
  
- To use Ollama in a Python program, first install the `ollama` Python package with `pip install ollama`. Once installed, you can use the models you've downloaed with Ollama in your Python program as follows.
  
```python
import ollama
response = ollama.chat(model='deepseek-r1', messages=[
    {
        'role': 'user',
        'content': 'Why is the sky blue?',
    },
])
print(response['message']['content'])
```

> To use a different model, simply change the value in `model= ` to the target model. For example, setting `model='llama3'` will use Meta's `LLaMA3` model instead.
{: .prompt-info }

- I'm sure you'll agree that this is really a fun way to experiment with AI models!!. If you are not comfortable with the terminal or CLI, I suggest you go for a tool like [LM Studio](https://lmstudio.ai/) which allows you to download and interact with LLMs via a user-friendly GUI.

- In the next tutorial, we will build a simple web search application that runs on a local model.

