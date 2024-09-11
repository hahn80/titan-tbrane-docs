# Documentation for TBrane


This service heavily depends on the GPU tasks from pytorch and CUDA libs.

1. Install cuda toolkits

Download cuda keyring and install cuda-toolkit

```sh
	wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
	sudo dpkg -i cuda-keyring_1.1-1_all.deb
	sudo apt update
	sudo apt -y install cuda
	sudo apt -y install cuda-toolkit-12-5
	sudo apt install nvidia-driver-555
```

Check CUDA version from commandline:

```sh
	nvidia-smi
```

You might get the information about the CUDA version:

```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI xxx.yy.zz              Driver Version: xxx.yy.zz      CUDA Version: 12.5     |
|-----------------------------------------+------------------------+----------------------+
```

2. Install pytorch with supported cuda libs

```sh
	sudo pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

3. Install titan-tbrane and Java 21

4. Run the test case from Java side.

Please check the sample source to see how to connect to the services.
