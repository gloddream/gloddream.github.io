---
layout: post
title:  "Setup Amazon AWS GPU instance with MXnet"
date:   2016-09-21 15:14:54
categories: jekyll
---

* content
{:toc}

在训练CNN的时候,需要使用GPU进行运算,简单对比了下阿里云的HPC和亚马逊AWS中EC2中的GPU实例,选择尝试使用AWS竞价请求实例和自定义AMI来开始.[五大主流深度学习框架](http://blog.revolutionanalytics.com/2016/08/deep-learning-part-1.html),对比选择使用[MXNET](http://mxnet.io/)进行开始.

## Create AWS GPU instance

* GPU实例
* Ubuntu 14.04 instance
* 竞价请求
* 安装完MXNET后,保存该AMI IMAGE,方便将来使用


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

>blacklist nouveau
>blacklist lbm-nouveau
>options nouveau modeset=0
>alias nouveau off
>alias lbm-nouveau off

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
	
	export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
	
**important**: Please reboot the instance for loading the driver:
	
	sudo reboot
	
If everything is fine, nvidia-smi should look like this (4 GPU instance) for example:
![nvidia-smi]({{ "/css/pics/nvidia-smi.png"}})  

---

### Optional: cuDNN

One can apply for the developer program here [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn). When approved, download cuDNN for Linux, upload the cuDNN package from the local computer to the instance:

	
>scp -i dnn.pem cudnn-7.5-linux-x64-v5.1.tgz ubuntu@ec2-52-43-236-249.us-west-2.compute.amazonaws.com:~

	
and install cuDNN:
 
	tar -zxf cudnn-7.5-linux-x64-v5.1.tgz
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

**important** "#if you have cuDNN, `uncomment` the following line #echo "USE_CUDNN=1" >>config.mk"


---

### Add some link lib path

	echo "export LD_LIBRARY_PATH=/home/ubuntu/mxnet/lib/:/usr/local/cuda-7.5/targets/x86_64-linux/lib/" >> ~/.bashrc
	

---

### Install python package

One can use the Anaconda python, do:
	
	wget https://repo.continuum.io/archive/Anaconda2-4.1.1-Linux-x86_64.sh
	
	bash Anaconda2-4.1.1-Linux-x86_64.sh 
	
**important** [不同的python版本具体安装方法参照这里](https://www.continuum.io/downloads#linux)

---

### Test it

	python example/image-classification/train_mnist.py --network lenet --gpus 0

---

## Some trouble shooting

**error1**

One may see some annoying message when starting training with GPU

`libdc1394 error: Failed to initialize libdc1394`

It is OpenCV problem on AWS. One can simply disable it by:

	sudo ln /dev/null /dev/raw1394

**error2**
	
always get:

`libdc1394 error: Failed to initialize libdc1394`
`Traceback (most recent call last):`
	`File "run.py", line 8, in <module>`
    `from skimage import io, transform`
	`ImportError: No module named skimage`
		
do:
	
	conda install opencv 
	conda install scikit-image	

**error3**

`Mac OS X: ValueError: unknown locale: UTF-8 in Python`

If you have faced the error on MacOS X, here's the quick fix - add these lines to your ~/.bash_profile:

	export LC_ALL=en_US.UTF-8
	export LANG=en_US.UTF-8


---
