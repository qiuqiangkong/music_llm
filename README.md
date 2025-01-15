# Music Generation/TTS with Large Language Models

This repository contains a PyTorch implementation of music generation and text-to-speech (TTS) using LLaMA-based large language models (LLMs). During training, the system converts both text and audio into discrete tokens. the LLM system is trained to predict the next discrete ID. During sampling, the system generates discrete audio IDs in an autoregressive manner. The figure below illustrates the training process of the LLM. Users can train a music generation model in less than 10 hours using a single RTX 4090 GPU.

<img src="./assets/llm.png" width="600">

## 0. Install dependencies

```bash
# Clone the repo
git clone https://github.com/qiuqiangkong/music_llm
cd music_llm

# Install Python environment
conda create --name music_llm python=3.10

# Activate environment
conda activate music_llm

# Install Python packages dependencies
bash env.sh
```

## 0. Download datasets

To train TTS system, download LJSpeech dataset containing 24 hours of speech from a single speaker.

```bash
wget -O LJSpeech-1.1.tar.bz2 https://data.keithito.com/data/speech/LJSpeech-1.1.tar.bz2
mkdir -p ./datasets
tar -xvf LJSpeech-1.1.tar.bz2 -C ./datasets/
wget -O ./datasets/LJSpeech-1.1/train.txt https://huggingface.co/datasets/flexthink/ljspeech/resolve/main/train.txt?download=true
wget -O ./datasets/LJSpeech-1.1/valid.txt https://huggingface.co/datasets/flexthink/ljspeech/resolve/main/valid.txt?download=true
wget -O ./datasets/LJSpeech-1.1/test.txt https://huggingface.co/datasets/flexthink/ljspeech/resolve/main/test.txt?download=true
```

To train music generation system, download GTZAN music dataset containing 8 hours of music with 10 genres.

```bash
wget -O genres.tar.gz https://huggingface.co/datasets/marsyas/gtzan/resolve/main/data/genres.tar.gz?download=true
mkdir -p ./datasets/gtzan
tar -zxvf genres.tar.gz -C ./datasets/gtzan/
```

## 1. Train

```python
CUDA_VISIBLE_DEVICES=0 python train.py --config="./configs/ljspeech.yaml"
```






The training takes around 10 hours to train for 100,000 steps on a single RTX4090 card. 

![Training & Validation Loss](assets/loss.png)

### Train on Multiple GPUs.

We use Huggingface accelerate library to train the systems on multiple GPUs. train_accelerate.py just adds a few lines to train.py. Here is an example to run with 4 GPUs:

```python
CUDA_VISIBLE_DEVICES=0,1,2,3 accelerate launch --multi_gpu --num_processes 4 train_accelerate.py --model_name=Llama
```

Then, the training can speed up by 4x times. The code can also train with multiple nodes such as 32 GPUs with 4 nodes.

## 2. Sample

```python
CUDA_VISIBLE_DEVICES=0 python sample.py --model_name=Llama --ckpt_path="checkpoints/train/Llama/step=10000.pth"
```

The sampled texts look like:

<pre>
We may! though a bald prove. We three, I say! What                    
must I see so, most heart?

Servant:
He hath ribbons of an the city, which he main for her
voices of the same winder. What say you to yours?

Provost:
It was commanded so willingly I do at ever.
So fortune
</pre>

## External links

This repo is benefited from the following repos.

NanoGPT: https://github.com/karpathy/nanoGPT

Lit-Llama: https://github.com/Lightning-AI/lit-llama

## License

MIT