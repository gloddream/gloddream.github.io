---
layout: post
title:  "Setup Amazon AWS GPU instance with MXnet"
date:   2016-09-21 15:14:54
categories: jekyll
---

* content
{:toc}

## Create AWS GPU instance

竞价请求实例,使用自定义iam


---

## Install dependencies

	

---

### Preparation

	sudo apt-get update && sudo apt-get upgrade
	sudo apt-get install -y build-essential git libcurl4-openssl-dev libatlas-base-dev libopencv-dev python-numpy unzip

---

### Update Linux kernel on AWS

	sudo apt-get install linux-image-extra-virtual

---

### Disable nouveau

	sudo vi /etc/modprobe.d/blacklist-nouveau.conf

    	blacklist nouveau
    	blacklist lbm-nouveau
    	options nouveau modeset=0
    	alias nouveau off
    	alias lbm-nouveau off

	echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
	sudo update-initramfs -u
	sudo reboot

Wait before finish rebooting, usually 1 min, and login back to the instance to continue installation:

	sudo apt-get install -y linux-source linux-headers-`uname -r`
	
---

## Install CUDA

	wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.5-18_amd64.deb
	sudo dpkg -i cuda-repo-ubuntu1404_7.5-18_amd64.deb
	sudo apt-get update
	sudo apt-get install -y cuda
Important: Please reboot the instance for loading the driver
	
	sudo reboot
If everything is fine, nvidia-smi should look like this (4 GPU instance) for example:

### Optional: cuDNN

	tar -zxf cudnn-7.0-linux-x64-v4.0-rc.tgz #or cudnn-7.0-linux-x64-v3.0-prod.tgz
	cd cuda
	sudo cp lib64/* /usr/local/cuda/lib64/
	sudo cp include/cudnn.h /usr/local/cuda/include/

---

## Install MXnet

	
	git clone --recursive https://github.com/dmlc/mxnet
	cd mxnet; cp make/config.mk .

	#add CUDA options
	echo "USE_CUDA=1" >>config.mk
	echo "USE_CUDA_PATH=/usr/local/cuda" >>config.mk
	#if you have cuDNN, uncomment the following line
	#echo "USE_CUDNN=1" >>config.mk
	echo "USE_BLAS=atlas" >> config.mk
	echo "USE_DIST_KVSTORE = 1" >>config.mk
	echo "USE_S3=1" >>config.mk
	make -j8

---

### Add some link lib path

	echo "export LD_LIBRARY_PATH=/home/ubuntu/mxnet/lib/:/usr/local/cuda-7.5/targets/x86_64-linux/lib/" >>> ~/.bashrc
	

---

### Install python package

One can either use the system’s python or Miniconda/Anaconda python as mentioned in my previous blog. If use system’s python, do:


---

### Test it

	python example/image-classification/train_mnist.py --network lenet --gpus 0
	

---

## Some trouble shooting

One may see some annoying message when starting training with GPU

`libdc1394 error: Failed to initialize libdc1394`

It is OpenCV problem on AWS. One can simply disable it by:

	sudo ln /dev/null /dev/raw1394
	

---
