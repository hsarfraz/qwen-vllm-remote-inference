# qwen-vllm-remote-inference

This project demonstrates a **client–server ML inference** setup where a large language model is hosted on a dedicated GPU machine and accessed remotely from a client laptop. The model is deployed using **containerized deployment** via Docker and served through vLLM, enabling efficient **remote GPU inference** without requiring the client machine to have GPU resources. The vLLM server exposes **OpenAI-compatible inference APIs**, allowing standard OpenAI SDKs to be used for communication, making the system easy to integrate into existing applications and workflows.

huggingface token (qwen test): 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

## Step 0: Verifying GPU access from Docker

```
docker run --rm --gpus all nvidia/cuda:12.2.0-runtime-ubuntu22.04 nvidia-smi
```

## Step 1: Build a reusable Docker image (Dockerfile)

### Step 1.1: Build a vLLM Docker image

```
docker build -t qwen-vllm .
```

### Step 1.2: Example Dockerfile 

```
FROM pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime

WORKDIR /workspace

RUN pip install --upgrade pip && \
    pip install vllm transformers accelerate safetensors

ENV HF_HOME=/models

```

## Step 2: VLL serve

* more info about `Qwen3-VL-2B-Instruct` on huggingface [link](https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct)

```
vllm serve Qwen/Qwen3-VL-2B-Instruct \
  --port 8000 \
  --max-model-len 139200 \
  --gpu-memory-utilization 0.9
```

## Step 3: Python client code to interact with the OpenAI-compatible API

```
from openai import OpenAI

client = OpenAI(
    base_url="http://<GPU_MACHINE_IP>:8000/v1",
    api_key="EMPTY"
)

response = client.chat.completions.create(
    model="Qwen/Qwen3-VL-2B-Instruct",
    messages=[
        {"role": "user", "content": "Explain Docker in simple terms"}
    ],
    max_tokens=500,
    temperature=0.7
)

print(response.choices[0].message.content)
```
