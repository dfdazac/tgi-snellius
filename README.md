# TGI on Snellius

[Text Generation Inference](TGI) (TGI) is a toolkit created by Hugging Face for deploying and serving Large Language Models (LLMs). TGI takes care of downloading models, running quantized versions of them, and distributing them over multiple GPUs.

This repository contains a guide for installing it on Snellius, where we can take advantage of 4-GPU nodes.

**Why not use a Docker image?**
The easiest way to run TGI is to download their preconfigured Docker image. This, however, requires installing the NVIDIA Container Toolkit, which to the best of my knowledge is not available on Snellius. The alternative then is to download and build the source code.

## Instructions for building TGI

1. Install Rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

2. Install Protoc (TODO: modify with instructions not using sudo)

```
PROTOC_ZIP=protoc-21.12-linux-x86_64.zip
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v21.12/$PROTOC_ZIP
sudo unzip -o $PROTOC_ZIP -d /usr/local bin/protoc
sudo unzip -o $PROTOC_ZIP -d /usr/local 'include/*'
rm -f $PROTOC_ZIP
```

3. The 2022 module is needed to load GCC 11.3.0, and the 2023 for CUDA 12.1.1. We also need CMake 3.24.3:

```
module load 2022
module load 2023
module load GCC/11.3.0
module load CUDA/12.1.1
module load CMake/3.24.3-GCCcore-11.3.0
```

4. Clone the TGI repository

```
git clone https://github.com/huggingface/text-generation-inference.git
```

5. Things get a bit messy here. As of today, there is a [bug](https://github.com/huggingface/text-generation-inference/issues/2355) related to a CUDA dependency that breaks building from source. A quick fix is to edit `text-generation-inference/server/Makefile`. From this file, remove the following line:

```
include Makefile-fbgemm
```

and change the line

```
install-cuda: install-server install-flash-attention-v2-cuda install-vllm-cuda install-flash-attention install-fbgemm
# to
install-cuda: install-server install-flash-attention-v2-cuda install-vllm-cuda install-flash-attention
```

6. Create and activate a new conda environment for TGI:

```
conda create -n tgi python=3.11
conda activate tgi
```

7. Enter the `text-generation-inference` directory, build, and cross your fingers. Building should take around 15 minutes.

```
cd text-generation-inference
BUILD_EXTENSIONS=True make install
```

