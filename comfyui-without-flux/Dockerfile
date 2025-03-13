ARG CUDA_VERSION="12.8.0"
ARG UBUNTU_VERSION="24.04"
ARG DOCKER_FROM=nvidia/cuda:$CUDA_VERSION-cudnn-devel-ubuntu$UBUNTU_VERSION

# Base NVidia CUDA Ubuntu image
FROM $DOCKER_FROM AS base

# Install Python and required packages
RUN apt-get update -y && \
    apt-get install -y python3-full python3-pip python3-venv software-properties-common && \
    add-apt-repository universe && \
    add-apt-repository multiverse && \
    apt-get update && \
    apt-get install -y --no-install-recommends openssh-server openssh-client git git-lfs wget vim zip unzip curl nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create and activate a virtual environment
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Upgrade pip inside the virtual environment
RUN /opt/venv/bin/pip install --upgrade pip

# Install PyTorch inside the virtual environment
ARG PYTORCH="2.6.0"
ARG CUDA="126"
RUN /opt/venv/bin/pip install --no-cache-dir -U torch==$PYTORCH torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu$CUDA

# Copy nginx default config
COPY default /etc/nginx/sites-available/default

ENV PATH="/usr/local/cuda/bin:${PATH}"

# Install missing dependencies
RUN apt-get update && apt-get install -y libgl1 libglib2.0-bin

# Clone ComfyUI and install dependencies
RUN git clone https://github.com/comfyanonymous/ComfyUI.git && \
    cd ComfyUI && \
    /opt/venv/bin/pip install -r requirements.txt && \
    cd custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Manager.git && \
    git clone https://github.com/pythongosssss/ComfyUI-Custom-Scripts.git && \
    cd /ComfyUI && \
    mkdir pysssss-workflows

# Copy required files
COPY --chmod=644 workflows/ /ComfyUI/user/default/workflows/
COPY --chmod=644 comfy.settings.json /ComfyUI/user/default/comfy.settings.json
COPY --chmod=644 character_sheet_example.png /ComfyUI/input/character_sheet_example.png
COPY --chmod=644 example_photo.png /ComfyUI/input/example_photo.png
COPY --chmod=644 example_photo_small.png /ComfyUI/input/example_photo_small.png
COPY --chmod=644 example_pose.png /ComfyUI/input/example_pose.png
COPY --chmod=644 example2.png /ComfyUI/input/example2.png
COPY --chmod=644 driving_video.mp4 /ComfyUI/input/driving_video.mp4

WORKDIR /workspace
EXPOSE 8188

# Fix incorrect file download
RUN wget -O /ComfyUI/flux_dev_example.png "https://raw.githubusercontent.com/comfyanonymous/ComfyUI_examples/master/flux/flux_dev_example.png"

# Install Xlabs-AI/flux-RealismLora
RUN cd /ComfyUI/custom_nodes && \
    git clone https://github.com/XLabs-AI/x-flux-comfyui.git && \
    cd x-flux-comfyui && \
    /opt/venv/bin/python setup.py

# Install Jupyter Notebook
RUN /opt/venv/bin/pip install jupyterlab
EXPOSE 8888

# Install additional custom nodes
RUN cd /ComfyUI/custom_nodes && \
    git clone https://github.com/flowtyone/ComfyUI-Flowty-LDSR.git && \
    cd ComfyUI-Flowty-LDSR && \
    /opt/venv/bin/pip install -r requirements.txt

# Additional models and custom nodes
RUN cd /ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-SUPIR.git && \
    cd ComfyUI-SUPIR && \
    /opt/venv/bin/pip install -r requirements.txt

# AI-Toolkit installation
RUN cd / && \
    git clone https://github.com/ostris/ai-toolkit.git && \
    cd ai-toolkit && \
    git submodule update --init --recursive && \
    /opt/venv/bin/pip install -r requirements.txt

# Copy AI-Toolkit config files
COPY --chmod=644 ai-toolkit/train_lora.yaml /ai-toolkit/config/train_lora.yaml
COPY --chmod=755 ai-toolkit/caption_images.py /caption_images.py
EXPOSE 7860

# Download model for tutorial workflow
RUN mkdir -p /ComfyUI/models/checkpoints && \
    wget -O /ComfyUI/models/checkpoints/v1-5-pruned-emaonly-fp16.safetensors "https://huggingface.co/Comfy-Org/stable-diffusion-v1-5-archive/resolve/main/v1-5-pruned-emaonly-fp16.safetensors"

# Install shell scripts
COPY --chmod=755 start-ssh-only.sh /start.sh
COPY --chmod=755 start-original.sh /start-original.sh
COPY --chmod=755 comfyui-on-workspace.sh /comfyui-on-workspace.sh
COPY --chmod=755 ai-toolkit-on-workspace.sh /ai-toolkit-on-workspace.sh
COPY --chmod=755 check_files.sh /check_files.sh
COPY --chmod=755 download_Controlnet.sh /download_Controlnet.sh
COPY --chmod=755 download_fp16.sh /download_fp16.sh
COPY --chmod=755 download_LDSR.sh /download_LDSR.sh
COPY --chmod=755 download_SUPIR.sh /download_SUPIR.sh
COPY --chmod=755 download_Florence-2.sh /download_Florence-2.sh
COPY --chmod=755 download_Upscalers.sh /download_Upscalers.sh
COPY --chmod=755 download_Outpainting.sh /download_Outpainting.sh
COPY --chmod=755 download_Workflows.sh /download_Workflows.sh
COPY --chmod=755 update_Workflows.sh /update_Workflows.sh
COPY --chmod=755 download_ALL.sh /download_ALL.sh
COPY --chmod=755 make_venv.sh /make_venv.sh
COPY --chmod=755 download_Files.sh /download_Files.sh
COPY --chmod=755 download_wan2.1.sh /download_wan2.1.sh
COPY --chmod=755 disable_mixlab.sh /disable_mixlab.sh

# Make directory and download face detection model
RUN mkdir -p /ComfyUI/models/ultralytics/bbox && \
    wget -O /ComfyUI/models/ultralytics/bbox/face_yolov8m.pt "https://huggingface.co/Bingsu/adetailer/resolve/main/face_yolov8m.pt"

# Make directory and download swap model file
RUN mkdir -p /ComfyUI/models/insightface && \
    wget -O /ComfyUI/models/insightface/inswapper_128.onnx "https://huggingface.co/ezioruan/inswapper_128.onnx/resolve/main/inswapper_128.onnx"

# Default startup command
CMD [ "/start.sh" ]
