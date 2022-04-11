# ONNC Working Environment Setup

## Preface

이 튜토리얼은 ONNC를 사용하여 NVDLA 기반 SoC에서 추론을 실행하기 위한 DNN 모델 그래프 정보가 포함된 Loadables를 생성하는 것을 목표로 합니다. 이 튜토리얼의 대부분의 정보는 NVDLA 백엔드 포팅에 특별히 맞춰져 있습니다.
소프트웨어 개발 프로세스를 용이하게 하기 위해 ONNC Docker 이미지가 Docker Hub에서 제공되어 빠른 배포가 가능합니다. 사전 설치된 종속 라이브러리가 있으며 바로 실행할 수 있는 작업 환경입니다. 사용자는 ONNC 소스 코드를 Docker 컨테이너에 탑재하고 컨테이너 내부에 소스 코드를 빌드할 수 있습니다. 또한 빌드된 ONNC 바이너리를 실행하여 컨테이너 내부의 심층 신경망(DNN) 모델을 컴파일할 수 있습니다. ONNC는 현재 GitHub 릴리스 v1.2에서 두 가지 백엔드 구현을 제공합니다. x86 백엔드의 경우 사용자는 내장 인터프리터인 ONNI를 사용하여 모델 추론을 실행할 수 있습니다. NVDLA 백엔드의 경우 컴파일 후 모델 그래프 정보가 포함된 로드 가능 파일이 생성됩니다. 사용자는 NVDLA 가상 플랫폼에서 로드 가능한 파일을 실행하여 모델 추론을 시뮬레이션할 수 있습니다. [NVIDIA Deep Learning Accelerator (NVDLA)](http://nvdla.org/index.html) 릴리스는 전체 시스템 소프트웨어 시뮬레이션을 위한 모든 기능을 갖춘 가상 플랫폼을 제공합니다. 우리는 공식적으로 출시된 가상 플랫폼을 활용하고 이 튜토리얼을 위해 약간의 변경을 합니다.

첫 번째 실습에서는 ONNC를 구축하고, ONNC를 사용하여 모델을 컴파일하고, 사전 포장된 가상 플랫폼에서 모델 추론을 시뮬레이션하는 방법을 설명하고 시연합니다.

## Prerequisite

시스템에 Docker가 설치되어 있지 않은 경우 Docker(http://www.docker.com)를 먼저 다운로드하여 설치하십시오.
또한 Git 서버에서 소스 코드를 검색하려면 Git(https://git-scm.com/)을 설치해야 합니다.

## Preparing Source Code and Docker Images

최신 ONNC 소스 코드는 GitHub에서 사용할 수 있습니다. 소스 코드를 다운로드하려면 다음 명령을 따르십시오.

```sh
$ git clone https://github.com/ONNC/onnc.git
$ cd onnc
$ git checkout tags/1.3.0
$ cd ..
```

다음 명령을 사용하여 튜토리얼 자료를 다운로드하십시오. 다음 실습에서 사용할 몇 가지 예시 DNN 모델 및 코드 조각이 있습니다.

```sh
$ git clone https://github.com/ONNC/onnc-tutorial.git
```

다음 명령을 사용하여 Docker Hub에서 Docker 이미지를 가져옵니다.

```sh
# We need two Docker images.

$ docker pull onnc/onnc-community
$ docker pull onnc/vp
```

Docker 이미지가 성공적으로 다운로드되었는지 확인하려면 다음 명령을 사용하여 사용 가능한 모든 Docker 이미지를 표시하십시오. 'onnc/onnc-community' 및 'onnc/vp' 이미지가 모두 표시되어야 합니다.

```sh
$ docker images
REPOSITORY                           TAG                                IMAGE ID            CREATED             SIZE
onnc/onnc-community                  latest                             fdd06c76c519        2 days ago          5.58GB
onnc/vp                              latest                             889c00396ea1        2 days ago          2.16GB
```


## Building ONNC and Compiling DNN Models

다음 명령을 사용하여 ONNC 커뮤니티 Docker를 불러옵니다.

```sh
$ docker run -ti --rm -v <absolute/path/to/onnc>:/onnc/onnc -v <absolute/path/to/tutorial>:/tutorial onnc/onnc-community
```

* `<absolute/path/to/onnc>` is the directory where you clone the ONNC source code. Note that it must be the absolute path other than a relative path.
* `<absolute/path/to/tutorial>` is the directory where you clone the ONNC tutorial material.
* The `-ti` option provides an interactive interface for the container.
* The `--rm` option will automatically clean up the container when the container exits.
* The `-v` option mounts the directory to the Docker container. With this option, you can make change to the source code (<path/to/onnc>) outside the Docker container with your favorite editor, and the change can be seen inside the Docker container and gets compiled.

Within the Docker container, use the following commands to build ONNC.

```sh
# Within onnc/onnc-community Docker container

$ cd /onnc/onnc-umbrella/build-normal

# Build ONNC.
$ smake -j8 install
```

* The `smake` command synchronizes the build directory with `<path/to/onnc>/onnc` and invokes the make command to build ONNC. 
* The `-j8` option is to parallelize compilation with 8 CPU cores.
* This command will automatically install the compiled binary in this container environment.

```sh
# Run ONNC to compile a DNN model.
$ onnc -mquadruple nvdla /tutorial/models/lenet/lenet.onnx

# Prepare the compiled output file for the virtual platform to run.
$ sudo mv out.nvdla /tutorial/models/lenet/
```

You may use the following command to exit the Docker prompt, 

```sh
# Within the onnc/onnc-community Docker container
$ exit
```

## Performing Model Inference on Virtual Platform

When you finish building ONNC and compiling a DNN model, you do not need the `onnc/onnc-community` Docker anymore. Start another console/terminal on your computer to enter the other Docker image called `onnc/vp` for model inference.

```sh
# Within your computer console

$ docker run -ti --rm -v <absolute/path/to/tutorial>:/tutorial onnc/vp
```

The virtual platform in this Docker is used to simulate the NVDLA runtime environment. As the following figure shows, the virtual platform contains a systemC model for the NVDLA hardware as well as a CPU emulator, where a Linux OS and NVDLA drivers are running to drive the NVDLA hardware.

<img src="../figures/runtime_env.png" width="400">

Within the VP Docker container, use the following commands to activate the virtual platform.

```sh
# Within onnc/vp Docker container

$ cd /usr/local/nvdla

# Prepare loadable, input, and golden output for the future use.
$ cp /tutorial/models/lenet/* .

# Run the virtual platform.
$ aarch64_toplevel -c aarch64_nvdla.lua

             SystemC 2.3.0-ASI --- Oct  9 2017 04:21:14
        Copyright (c) 1996-2012 by all Contributors,
        ALL RIGHTS RESERVED

No sc_log specified, will use the default setting
verbosity_level = SC_MEDIUM
bridge: tlm2c_elaborate..
[    0.000000] Booting Linux on physical CPU 0x0
# ...
Initializing random number generator... done.
Starting network: udhcpc: started, v1.27.2
udhcpc: sending discover
udhcpc: sending select for 10.0.2.15
udhcpc: lease of 10.0.2.15 obtained, lease time 86400
deleting routers
adding dns 10.0.2.3
OK
Starting sshd: [    4.590433] NET: Registered protocol family 10
[    4.606182] Segment Routing with IPv6
OK

Welcome to Buildroot
nvdla login:
```

By starting the virtual platform, a Linux kernel is brought up and stops at the login prompt.

* nvdla login: root
* Password: nvdla

After logging into the Linux prompt, use the following commands to install the drivers.

```sh
# Within the virtual platform

$ mount -t 9p -o trans=virtio r /mnt && cd /mnt

# Install KMD.
$ insmod drm.ko && insmod opendla.ko
[  469.730339] opendla: loading out-of-tree module taints kernel.
[  469.734509] reset engine done
[  469.737998] [drm] Initialized nvdla 0.0.0 20171017 for 10200000.nvdla on minor 0
```

Up to this point, everything is ready for running model inference. In this lab, we demonstrate with a real-world model, LeNet, which is used for hand-written digit recognition. We have prepared some 28x28 images (`.pgm` files) to represent digit numbers 0 to 9. We begin with running model inference to recognize digit number 0 with input file `input0.pgm`. The inference simulation will take about a few minutes. 

```sh
# Within the virtual platform

# Run the NVDLA runtime (containing UMD) to do model inference.
$ ./nvdla_runtime --loadable out.nvdla --image input0.pgm --rawdump
creating new runtime context...
Emulator starting
# ...
[  126.029817] Enter:dla_handle_events, processor:CDP
[  126.029995] Exit:dla_handle_events, ret:0
[  126.030146] Enter:dla_handle_events, processor:RUBIK
[  126.030323] Exit:dla_handle_events, ret:0
[  126.032432] reset engine done
Shutdown signal received, exiting
Test pass
```

After the simulation is done, we will derive an output file `output.dimg` containing the model output values.
In this example, the output file should look like the follows:

```sh
$ more output.dimg
149.25 -49.625 13.875 11.2344 -59.8125 -2.61523 7.80078 -44.7188 30.8594 17.3594
```

In the file, there are ten numbers indicating the confidence level of the 10 digits from 0 to 9, respectively.
For example, the first number 149.25 indicates the confidence level of digit 0, and the next -49.625 of digit 1, and so on. Among those numbers, the largest one implies the recognition result. In this case, the first number 149.25 is the largest one, so the corresponding digit 0 is the recognition result.

After the experiment, you can use the following command to exit the virtual platform.

```sh
# Within the virtual platform
$ poweroff
```

Use the following command to exit the `onnc/vp` Docker prompt.

```sh
# Within the onnc/vp Docker container
$ exit
```
