# Install Caffe with GPU on Ubuntu 16.04
-----------------------
**Tested under:**
* Ubuntu 16.04 64 bit desktop
* Geforce GT 730m/750m GPU

**Before installation**
* Make sure your graphic cards is made by Nvidia instead of AMD/ATI, or you cannot install CUDA
* Please do not install cudnn if your card is not supported by cudnn

**Steps to install caffe with GPU on Ubuntu 16.04:**
* Install proprietary graphic driver
* Install CUDA and cudnn
* Install operating system prerequisites
* Install caffe

Go to [https://github.com/BVLC/caffe/wiki/Ubuntu-16.04-or-15.10-Installation-Guide](https://github.com/BVLC/caffe/wiki/Ubuntu-16.04-or-15.10-Installation-Guide) for more detail of installing caffe on Ubuntu 16.04.

## Check hardware
### Graphic card
* Windows 10: right click `This PC`/`这台电脑`, select `Manage`/`管理`, then `Hardware Manage`/`硬件管理`, go to graphic cards, if there is `Nvidia` then your graphic card is made by Nvidia. And there will be something like "Geforce GT 750M", it means the GPU is Geforce GT 750M(M means mobile, it is for laptop).
* Linux: `lspci |grep VGA`.

### Whether supported by cudnn
* Open https://developer.nvidia.com/cuda-gpus and click `CUDA-Enabled GeForce Products`, then search your graphic cards, and you will see `Compute Capability`, do not install cudnn if the value is less than 3.0
* More info: [http://blog.csdn.net/eagelangel/article/details/50562065](http://blog.csdn.net/eagelangel/article/details/50562065)


## Install Proprietaty Graphic Driver
	sudo apt-get update

Then go to `Additional Drivers` in Ubuntu to install Nvidia driver, if there is no additional driver, try to change system source to Main Server and try again.

For more info: [https://help.ubuntu.com/community/BinaryDriverHowto/Nvidia](https://help.ubuntu.com/community/BinaryDriverHowto/Nvidia) (the part of Installation)

## Install CUDA

### Install CUDA

Download cuda-repo-ubuntu1604-8-0-local_8.0.44-1_amd64.deb from Nvidia website, then:

	sudo dpkg -i cuda-repo-ubuntu1604-8-0-local_8.0.44-1_amd64.deb
	sudo apt-get update
	sudo apt-get install cuda


**DO NOT INSTALL nvidia-cuda-toolkit, or it will fail while make runtest, since nvidia-cuda-toolkit from Ubuntu official repository is 7.5, which is not compatible with CUDA 8.0**

**DO NOT INSTALL nvidia-cuda-toolkit, or it will fail while make runtest, since nvidia-cuda-toolkit from Ubuntu official repository is 7.5, which is not compatible with CUDA 8.0**

**DO NOT INSTALL nvidia-cuda-toolkit, or it will fail while make runtest, since nvidia-cuda-toolkit from Ubuntu official repository is 7.5, which is not compatible with CUDA 8.0**

### Install CUDNN

Make sure your card is supported by cudnn before download cudnn-8.0-linux-x64-v5.1.tgz from Nvidia website and then:

	tar zxf cudnn-8.0-linux-x64-v5.1.tgz
	cd cuda
	sudo cp include/cudnn.h /usr/local/cuda/include
	sudo cp lib64/libcudnn* /usr/local/cuda/lib64
	sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
	
Verify installation of CUDNN: `cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2`, if `could not set cudnn filter descriptor` is print, cudnn installation is failed
Note:After you have done things above,you should restart your computer.

## Install system prerequisites

	sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get install -y build-essential cmake git pkg-config
	sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler
	sudo apt-get install -y libatlas-base-dev 
	sudo apt-get install -y --no-install-recommends libboost-all-dev
	sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev
	# (Python general)
	sudo apt-get install -y python-pip
	# (Python 2.7 development files)
	sudo apt-get install -y python-dev
	sudo apt-get install -y python-numpy python-scipy
	# (or, Python 3.5 development files)
	# sudo apt-get install -y python3-dev
	# sudo apt-get install -y python3-numpy python3-scipy
	# (OpenCV 2.4)
	sudo apt-get install -y libopencv-dev

## Install Caffe

Download caffe souce code as zip from Github, and：

	unzip caffe-master.zip
	cd caffe-master
	cp Makefile.config.example Makefile.config
	vim Makefile.config

Makefile.config should looks like:

	USE_CUDNN := 1		# only when cudnn is supported
	PYTHON_INCLUDE := /usr/include/python2.7 /usr/lib/python2.7/dist-packages/numpy/core/include
	WITH_PYTHON_LAYER := 1
	INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
	LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial

Then execute:

	find . -type f -exec sed -i -e 's^"hdf5.h"^"hdf5/serial/hdf5.h"^g' -e 's^"hdf5_hl.h"^"hdf5/serial/hdf5_hl.h"^g' '{}' \;

	cd /usr/lib/x86_64-linux-gnu
	sudo ln -s libhdf5_serial.so.10.1.0 libhdf5.so
	sudo ln -s libhdf5_serial_hl.so.10.0.2 libhdf5_hl.so

Then go back to caffe folder and execute:(locate in caffe-master)

	cd python
	for req in $(cat requirements.txt); do pip install $req; done

If there is any error, execute `for req in $(cat requirements.txt); do pip install $req; done` again. Then:

	cd ..
	vim Makefile

Replace `NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)` with `NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)`

Then:

	make all -j $(($(nproc) + 1))
	make test -j $(($(nproc) + 1))
	make runtest -j $(($(nproc) + 1))
	make pycaffe -j $(($(nproc) + 1))
	make distribute -j $(($(nproc) + 1))

## Post installation

### Add PYTHONPATH

In order to make the Python work with Caffe, open the file ~/.bashrc for editing in your favorite text editor. There, add the following line at the end of file:

	export PYTHONPATH=/path/to/caffe-master/python:$PYTHONPATH
	source ~/.bashrc

Validate caffe installation:

	python
	>>> import caffe;

Success if there is no error.

### Download official models

Then download some official models. This step is unnecessary.

	cd scripts
	./download_model_binary.py ../models/bvlc_alexnet/
	./download_model_binary.py ../models/bvlc_googlenet/
	./download_model_binary.py ../models/bvlc_reference_caffenet/
	./download_model_binary.py ../models/bvlc_reference_rcnn_ilsvrc13/
	./download_model_binary.py ../models/finetune_flickr_style/
