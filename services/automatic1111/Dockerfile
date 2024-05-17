FROM nvidia/cuda:11.6.2-cudnn8-runtime-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && \
    apt install -y bash \
        build-essential \
        vim \
        wget \
        screen \
        git \
        git-lfs \
        curl \
        ca-certificates \
        libsndfile1-dev \
        python3.8 \
        python3-pip \
        python3.8-venv \
        libgl1-mesa-glx && \
    rm -rf /var/lib/apt/lists

# make sure to use venv
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN python3 -m pip install --no-cache-dir --upgrade pip && \
    python3 -m pip install --no-cache-dir \
        wheel

WORKDIR /root/

# setup automatic1111
# rename the folder to use underscores since python doesn't like dashes
RUN git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git stable_diffusion_webui
WORKDIR /root/stable_diffusion_webui
RUN git checkout c9c8485

# install dreambooth extension
# WORKDIR /root/stable_diffusion_webui/extensions
# RUN git clone https://github.com/d8ahazard/sd_dreambooth_extension.git sd_dreambooth_extension
# WORKDIR /root/stable_diffusion_webui/extensions/sd_dreambooth_extension
# RUN git checkout fd51c0b2ed20566c60affa853a32ebce1b0a1139

# install controlnet extension
WORKDIR /root/stable_diffusion_webui/extensions
RUN git clone https://github.com/Mikubill/sd-webui-controlnet.git
WORKDIR /root/stable_diffusion_webui/extensions/sd-webui-controlnet/models
RUN git clone https://huggingface.co/lllyasviel/ControlNet-v1-1
RUN mv ./ControlNet-v1-1/* ./
RUN rm -rf ./ControlNet-v1-1/

# RUN git clone https://huggingface.co/TencentARC/T2I-Adapter
# RUN mv ./T2I-Adapter/models/* ./
# RUN rm -rf ./T2I-Adapter/
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/cldm_v15.yaml
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/cldm_v21.yaml
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_canny-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_depth-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_hed-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_mlsd-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_normal-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_openpose-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_scribble-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/control_seg-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_depth-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_keypose-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_seg-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_sketch-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_color-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_style-fp16.safetensors
# RUN wget https://huggingface.co/webui/ControlNet-modules-safetensors/resolve/main/t2iadapter_canny-fp16.safetensors

# COPY emptyroom-controlnet-center-crop.ckpt ./
# COPY emptyroom-controlnet-mlsd-center-crop.ckpt ./
# COPY emptyroom-latest.ckpt ./

# install dependencies
# note: we also make extra effort to call prepare_environment() which 
# downloads models, sets up other dependencies, etc. This way, we
# don't need to do it at runtime.
WORKDIR /root/stable_diffusion_webui
RUN pip install -r requirements_versions.txt && \
    COMMANDLINE_ARGS="--skip-torch-cuda-test --xformers" python -c "import sys; from launch import prepare_environment; prepare_environment();"

# Fixes issue with 8-bit adam bitsandbytes 
# see: https://github.com/TimDettmers/bitsandbytes/issues/52
RUN ln -s /usr/local/cuda/lib64/libcudart.so.11.0 /usr/local/cuda/lib64/libcudart.so 

# Downgrade to CUDA 116
RUN pip uninstall -y torch torchvision xformers && \
    pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cu116

# Install xformers - we need at least 0.0.17
RUN pip install xformers==0.0.17
   
# start the app
WORKDIR /root/stable_diffusion_webui
ENTRYPOINT ["python3", "./webui.py"]
