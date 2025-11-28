# Cuda

## 安装

Win：https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_network 镜像：https://github.com/futureflsl/cuda_cudnn_mirror

只安装驱动（无nvcc，但已经可以跑torch）：apt search nvidia-driver

官方两种安装方式：rpm/deb包、runfile包。还可用conda装cuda toolkit。还有docker

https://docs.nvidia.com/cuda/cuda-installation-guide-linux/

* runfile
  * sudo /usr/local/cuda-11/bin/cuda-uninstaller 如果没有，说明不是此方法安装的。apt装的也在此目录里
  * curl https://developer.download.nvidia.com/compute/cuda/12.2.0/local_installers/cuda_12.2.0_535.54.03_linux.run | sudo sh
* apt
  * 如果已经安装了驱动：使用nvidia-smi能看到支持的最高cuda版本，再安装cuda-toolkit-12-1
  * 未安装驱动，连带安装：sudo apt install cuda
  * 好像debian内置的要加nvidia-前缀，在non-free里。nv官方repo则不用，而是要加keyring：wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb; dpkg -i cuda-keyring_1.1-1_all.deb

查看版本：nvcc -V

# TensorRT

## 安装

* pip install wheel; pip install tensorrt
* 不需要cuDNN。可选cuBLAS，能加速某些层
* pypi包：普通版、lean版、dispatch版。普通版是自包含的，lean好像需要系统里有tensorrt；dispatch用于加载旧版本lean，只是一个shim。目前支持3.10-3.13，linux和win。默认-cu13

## 使用

* Builder：优化模型，产生Engine
* Engine：特定于创建它的TRT版本（可放宽，新版运行时一般可运行老版的）、GPU（同代高端的可运行低端生成的）、操作系统、CPU架构
* Torch-TensorRT：转换模型，从torch到trt。其他类型要导出为ONNX，再转换为TRT（第二步内置）。

## 相关项目

### TensorRT Model Optimizer

* 量化、剪枝、蒸馏
* 已经做好的模型：https://huggingface.co/collections/nvidia/inference-optimized-checkpoints-with-model-optimizer

### triton-inference-server

部署Engine，提供HTTP/GRPC API，后端支持多个，支持serve多个模型。对于windows的支持较差，曾经可以，后来不行，有人提issue没人处理

### TensorRT-LLM

* https://github.com/NVIDIA/TensorRT-LLM https://nvidia.github.io/TensorRT-LLM/latest/overview.html
* 包含了对LLM的优化，如custom attention kernels, inflight batching, paged KV caching, quantization (FP8, FP4, INT4 AWQ, INT8 SmoothQuant, ...), speculative decoding。前身叫FasterTransformer，甚至不需要TensorRT，后来才改了
* 是python-native的，提供不同于HF的API
* 必须用系统级别的CUDA，推荐使用容器，只支持linux。目前py轮子不支持cu13
* 不支持win
* trtllm-serve "TinyLlama/TinyLlama-1.1B-Chat-v1.0"
* trtllm-bench、trtllm-eval

# TODO

[torch-tensorrt](https://docs.pytorch.org/TensorRT/)
