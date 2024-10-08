## Requirements

Before proceeding, ensure that your system meets the following requirements:
- **NVIDIA Driver Version**: 555 or higher
- **NVIDIA Docker Toolkit**: Must be installed. You can install it by following the official instructions [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

## Getting started

### 1. Clone the Project and Pull Docker Image

First, clone the repository and navigate to the appropriate directory:

```bash
git clone https://github.com/LongNTKR/Research.git
cd Research/video_upscale
```

Now, pull the Docker image needed for the upscaling process:

```bash
docker pull krntl/videoupscale:24_10_08
```

### 2. Download engine

Download all the engine files from Hugging Face

[Download engine files here](https://huggingface.co/datasets/hg-ai/video_upscale_engine)

After downloading, place the engine files into the `engine` in your local directories.

**Important Note:** 
The provided engine files are optimized for NVIDIA GeForce RTX 4090, 3060 GPUs. If you're using a different GPU model, you'll need to rebuild the engine files to ensure compatibility and optimal performance with your specific hardware.

To rebuild the engine for your GPU:

```bash
# You need to be inside the videoupscale folder with cli before running the following step, git clone repo and cd into it
# the folderpath before ":" will be mounted in the path which follows afterwards
# contents of the videoupscale folder should appear inside /workspace/tensorrt

sudo docker run --privileged --gpus all -it --rm -v /home/videoupscale_path/:/workspace/tensorrt krntl/videoupscale:latest
```

After entering the container, download the ONNX model from the same Hugging Face link provided above for the engine files.
Move the downloaded ONNX model to the engine folder inside the api folder. Then run the following command to build the engine optimized for your GPU.

```bash
trtexec --bf16 --fp16 --onnx=model.onnx --minShapes=input:1x3x8x8 --optShapes=input:1x3x720x1280 --maxShapes=input:1x3x1080x1920 --saveEngine=model.engine --tacticSources=+CUDNN,-CUBLAS,-CUBLAS_LT --skipInference --useCudaGraph --noDataTransfers --builderOptimizationLevel=5
```

Ensure you rebuild the engine on a machine with the same GPU model you intend to use for video upscaling.

### 3. Update the `compose.yaml` File

Before running the project, you'll need to make sure the volume paths in the `compose.yaml` file are correctly set to your local directories.

```yaml
volumes:
  - /your/local/path/inputs:/workspace/tensorrt/inputs
  - /your/local/path/outputs:/workspace/tensorrt/outputs
  - /your/local/path/engine:/workspace/tensorrt/app/engine
```

Replace `/your/local/path/inputs`, `/your/local/path/outputs`, and `/your/local/path/engine` with the actual paths on your system where your input files, output files, and engine files are stored.

### 4. Run the Project

After you've set up the correct paths, you can start the service using docker-compose. Ensure that Docker is running, then execute:

```bash
docker-compose -f compose.yaml up -d
```

This will spin up the container, and the video upscaling service will be available at http://localhost:8080.
