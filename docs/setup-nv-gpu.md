# Nvidia GPU Container Toolkit

NVIDIA Container Tookit is a collection of libraries and utilities designed to expose NVIDIA GPU hardware
to containerized environments. It allows Linux containers to run GPU-accelerated applications—such as 
Deep Learning frameworks (PyTorch, TensorFlow), CUDA applications, and graphics APIs 
(OpenGL, Vulkan)—without bundling the heavy host GPU drivers inside the container image.

To enable nvidia_gpu_exporter on Docker, install nvidia-container-toolkit as follow:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.dev/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

sudo nvidia-ctk runtime configure --runtime=docker

sudo systemctl restart docker
```
