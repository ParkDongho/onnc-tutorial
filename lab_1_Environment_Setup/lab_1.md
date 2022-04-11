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

* `<absolute/path/to/onnc>`는 ONNC 소스 코드를 복제하는 디렉토리입니다. 상대 경로가 아닌 절대 경로여야 합니다.
* `<absolute/path/to/tutorial>`은 ONNC 튜토리얼 자료를 복제하는 디렉토리입니다.
* `-ti` 옵션은 컨테이너에 대한 대화형 인터페이스를 제공합니다.
* `--rm` 옵션은 컨테이너가 종료될 때 컨테이너를 자동으로 정리합니다.
* `-v` 옵션은 디렉토리를 Docker 컨테이너에 마운트합니다. 이 옵션을 사용하면 선호하는 편집기를 사용하여 Docker 컨테이너 외부의 소스 코드(<path/to/onnc>)를 변경할 수 있으며 변경 사항은 Docker 컨테이너 내부에서 확인되고 컴파일될 수 있습니다.

Docker 컨테이너 내에서 다음 명령을 사용하여 ONNC를 빌드합니다.

```sh
# Within onnc/onnc-community Docker container

$ cd /onnc/onnc-umbrella/build-normal

# Build ONNC.
$ smake -j8 install
```

* `smake` 명령은 빌드 디렉토리를 `<path/to/onnc>/onnc`와 동기화하고 make 명령을 호출하여 ONNC를 빌드합니다.
* `-j8` 옵션은 8개의 CPU 코어로 컴파일을 병렬화하는 것입니다.
* 이 명령은 이 컨테이너 환경에 컴파일된 바이너리를 자동으로 설치합니다.

```sh
# Run ONNC to compile a DNN model.
$ onnc -mquadruple nvdla /tutorial/models/lenet/lenet.onnx

# Prepare the compiled output file for the virtual platform to run.
$ sudo mv out.nvdla /tutorial/models/lenet/
```

다음 명령을 사용하여 Docker 프롬프트를 종료할 수 있습니다.

```sh
# Within the onnc/onnc-community Docker container
$ exit
```

## Performing Model Inference on Virtual Platform

ONNC 빌드 및 DNN 모델 컴파일을 마치면 'onnc/onnc-community' Docker가 더 이상 필요하지 않습니다. 컴퓨터에서 다른 콘솔/터미널을 시작하여 모델 추론을 위해 `onnc/vp`라는 다른 Docker 이미지를 입력합니다.

```sh
# Within your computer console

$ docker run -ti --rm -v <absolute/path/to/tutorial>:/tutorial onnc/vp
```

이 Docker의 가상 플랫폼은 NVDLA 런타임 환경을 시뮬레이션하는 데 사용됩니다. 다음 그림에서 볼 수 있듯이 가상 플랫폼에는 NVDLA 하드웨어용 systemC 모델과 CPU 에뮬레이터가 포함되어 있습니다. 여기서 Linux OS 및 NVDLA 드라이버가 실행되어 NVDLA 하드웨어를 구동합니다.

<img src="../figures/runtime_env.png" width="400">

VP Docker 컨테이너 내에서 다음 명령을 사용하여 가상 플랫폼을 활성화합니다.

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

가상 플랫폼을 시작하면 Linux 커널이 실행되고 로그인 프롬프트에서 중지됩니다.

* nvdla 로그인: 루트
* 비밀번호: nvdla

Linux 프롬프트에 로그인한 후 다음 명령을 사용하여 드라이버를 설치합니다.

```sh
# Within the virtual platform

$ mount -t 9p -o trans=virtio r /mnt && cd /mnt

# Install KMD.
$ insmod drm.ko && insmod opendla.ko
[  469.730339] opendla: loading out-of-tree module taints kernel.
[  469.734509] reset engine done
[  469.737998] [drm] Initialized nvdla 0.0.0 20171017 for 10200000.nvdla on minor 0
```

이 시점까지 모든 것이 모델 추론을 실행할 준비가 되었습니다. 이 실습에서는 손으로 쓴 숫자 인식에 사용되는 실제 모델인 LeNet으로 시연합니다. 0에서 9까지의 숫자를 나타내기 위해 28x28 이미지(`.pgm` 파일)를 준비했습니다. 입력 파일 `input0.pgm`으로 숫자 0을 인식하는 모델 추론을 실행하는 것으로 시작합니다. 추론 시뮬레이션은 몇 분 정도 걸립니다.

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

시뮬레이션이 완료되면 모델 출력 값이 포함된 출력 파일 'output.dimg'를 파생합니다.
이 예에서 출력 파일은 다음과 같아야 합니다.

```sh
$ more output.dimg
149.25 -49.625 13.875 11.2344 -59.8125 -2.61523 7.80078 -44.7188 30.8594 17.3594
```

파일에는 각각 0에서 9까지 10자리의 신뢰 수준을 나타내는 10개의 숫자가 있습니다.
예를 들어, 첫 번째 숫자 149.25는 숫자 0의 신뢰 수준을 나타내고 숫자 1의 다음 -49.625 등을 나타냅니다. 그 중 가장 큰 숫자가 인식 결과를 의미합니다. 이 경우 첫 번째 숫자 149.25가 가장 큰 숫자이므로 해당 숫자 0이 인식 결과입니다.

실험 후 다음 명령을 사용하여 가상 플랫폼을 종료할 수 있습니다.

```sh
# Within the virtual platform
$ poweroff
```

다음 명령을 사용하여 `onnc/vp` Docker 프롬프트를 종료합니다.

```sh
# Within the onnc/vp Docker container
$ exit
```
