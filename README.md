# qwen-vllm-remote-inference

This project demonstrates a **client–server ML inference** setup where a large language model is hosted on a dedicated GPU machine and accessed remotely from a client laptop. The model is deployed using **containerized deployment** via Docker and served through vLLM, enabling efficient **remote GPU inference** without requiring the client machine to have GPU resources. The vLLM server exposes **OpenAI-compatible inference APIs**, allowing standard OpenAI SDKs to be used for communication, making the system easy to integrate into existing applications and workflows.

# Prerequisites

Before starting, ensure you have the following installed:

* Docker (https://www.docker.com/)
* NVIDIA GPU drivers and NVIDIA Container Toolkit  if using GPU acceleration
* Python 3.10+ (optional, for API testing)

Keywords to search: docker install, nvidia container toolkit install, python 3 installation ubuntu

# Step 1: Setup Docker Environment

Create a Dockerfile (or use the existing one) with the following base:

```
# Use NVIDIA CUDA base image
FROM nvidia/cuda:12.2.0-devel-ubuntu22.04

# Install OS dependencies
RUN apt-get update && \
    apt-get install -y python3 python3-pip git curl && \
    apt-get clean

# Copy project files
COPY . /workspace
WORKDIR /workspace

# Install Python dependencies
RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt

# Expose port for API
EXPOSE 8000

# Run the vLLM server
CMD ["python3", "launch_server.py"]
```

Keywords: dockerfile cuda ubuntu, apt-get install python3 pip, dockerfile copy workdir, dockerfile expose port, dockerfile cmd

Note: Replace launch_server.py with the script that starts your vLLM API.

# Step 2: Build Docker Image

Build the Docker image:

```
docker build -t qwen-vllm:latest .
```

Keywords: docker build image, docker build -t tagname

# Step 3: Run the Docker Container

Run the container and map port 8000:

```
docker run --gpus all -p 8000:8000 --name qwen-vllm-container qwen-vllm:latest
```

* --gpus all ensures GPU usage (if available)
* -p 8000:8000 maps the container port to localhost
* --name gives the container a name for easier management

Keywords: docker run container, docker run port mapping, docker run with gpus

Tip: Use docker ps to check running containers.

# Step 4: Test the Server

Ping the server to ensure it’s running:

```
curl http://localhost:8000/ping
```

Send a test completion request:

```
curl -X POST http://localhost:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "Qwen/Qwen2.5-7B-Instruct",
        "prompt": "Hello vLLM! Say hi in 2 sentences.",
        "max_tokens": 50
      }'
```

Expected output: JSON with model-generated text.

Keywords: curl test api, curl post json, api ping test

# Step 5: Using the API in Python

Example Python script:

```
import requests

url = "http://localhost:8000/v1/completions"
data = {
    "model": "Qwen/Qwen2.5-7B-Instruct",
    "prompt": "Write a short greeting in 2 sentences.",
    "max_tokens": 50
}

response = requests.post(url, json=data)
print(response.json()["choices"][0]["text"])
```

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
