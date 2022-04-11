# Digit Recognition with ARM Cortex-M


## Preface

머신 러닝이 엣지로 이동하고 있습니다. 사람들은 스마트 스피커를 위한 음성 인식 및 감시 카메라를 위한 얼굴 감지와 같은 고급 서비스를 제공하기 위해 임베디드 장치에 에지 컴퓨팅 기능을 갖기를 원합니다. Arm Cortex-M 프로세서 제품군은 스마트하고 연결된 임베디드 애플리케이션의 요구 사항을 충족하는 확장 가능하고 에너지 효율적이며 사용하기 쉬운 프로세서입니다. Cortex CMSIS(마이크로컨트롤러 소프트웨어 인터페이스 표준)는 Cortex-M 프로세서 시리즈를 위한 공급업체 독립적인 하드웨어 추상화 계층입니다. 연구(https://arxiv.org/abs/1801.06601)에 따르면 기계 학습은 새로운 CMSIS-NN 소프트웨어 프레임워크가 있는 Cortex-M 플랫폼에서 입증된 4.6배 향상되었습니다. 이 실습에서는 ARM Cortex-M 마이크로프로세서를 위한 ONNC 백엔드 'CortexM' 백엔드를 소개하고 종단 간 애플리케이션, 필기 인식을 시연합니다. CortexM 백엔드는 [CMSIS-NN 라이브러리](https://github.com/ARM-software/CMSIS_5)를 통합하여 다음과 같은 심층 신경망(DNN) 모델에서 여러 인기 있는 운영자를 위한 컴퓨팅 기능 세트를 제공합니다. 컨볼루션, 최대 풀링 등. 라이브러리는 ARM Cortex-M CPU에서 모델 추론 속도를 높이는 데 최적화되어 있습니다. 다음 그림은 모델 연산자와 CMSIS-NN 함수 호출 간의 매핑 예를 보여줍니다.

<img src="../figures/cortexm_code_snapshot.png" width="450">

이 실습에서는 종단 간 응용 프로그램을 사용하여 ONNC 프레임워크가 대상 하드웨어에 대한 AI 추론을 쉽게 지원하는 방법을 보여줍니다.

## Deploying MNIST Model Inference on an Embedded System

다음 다이어그램은 Cortex-M 플랫폼에서 MNIST 모델 추론을 배포하는 방법을 보여줍니다.

<img src="../figures/cortexm_flow.png" width="450">

일반적으로 기계 학습 모델은 GPU 그래픽 카드 또는 서버에서 부동 소수점 데이터로 훈련되지만 제한된 계산 능력으로 인해 임베디드 장치에서 더 낮은 정밀도로 추론을 실행하는 것이 선호됩니다. 다행히도 여러 연구 논문에서 데이터를 정수로 양자화하는 것이 일반적으로 성능(즉, 정확도) 손실 없이 수행될 수 있음을 입증했습니다. 이 실습에서는 ONNX 형식의 양자화된 MNIST 모델을 준비했습니다. 입력 데이터와 가중치는 모두 8비트 정수입니다. 추론을 실행할 때 내부 계산 데이터 경로는 정확도 손실을 피하기 위해 8비트보다 더 높은 정밀도를 가질 수 있지만 활성화 데이터 정밀도는 구현에서 다시 8비트로 변환됩니다. 많은 CMSIS-NN 기능은 단순히 "오른쪽으로 이동" 논리를 사용하여 비트 너비 변환을 수행합니다. 오른쪽 시프트의 양은 일반적으로 가중치 양자화와 함께 결정되므로 Cortex-M 백엔드에서 하나의 사용자 입력으로 남겨둡니다. ONNX 모델 형식에는 활성화 데이터에 대한 보정 정보가 포함되어 있지 않습니다. 오른쪽 시프트 정보를 저장하기 위해 보정 파일이라는 별도의 파일을 준비했습니다.

ONNC를 사용하여 MNIST 모델 추론 응용 프로그램(`.cpp' 파일)을 컴파일한 후 ARM 크로스 컴파일러를 사용하여 응용 프로그램과 CMSIS-NN 라이브러리를 함께 컴파일하고 연결합니다. 응용 프로그램 소프트웨어는 기본 임베디드 시스템과 대상 응용 프로그램에 따라 다릅니다. 사용자는 공급업체로부터 하드웨어 종속 정보를 찾을 수 있습니다. 펌웨어 바이너리가 준비되면 보드 공급업체에서 제공해야 하는 ISP 도구를 통해 바이너리 파일을 대상 보드에 업로드합니다.

## Prerequisite

시스템에 Docker가 설치되어 있지 않은 경우 Docker(http://www.docker.com)를 먼저 다운로드하여 설치하십시오. 또한 GitHub 서버에서 소스 코드를 가져오려면 Git(https://git-scm.com/)을 설치해야 합니다. 또한 데모에서는 널리 사용되는 GUI 프로그래밍 프레임워크인 Processing을 사용하므로 Processing(https://processing.org/)도 설치하세요. 마지막으로 ARM Cortex-M CPU가 탑재된 개발 보드를 준비해야 합니다. 펌웨어 컴파일에 [Mbed compatible boards](https://os.mbed.com/platforms/)를 사용하기 때문에 [Mbed framework](https://www.mbed.com/en/)를 사용하는 것이 좋습니다. 보드가 Mbed와 호환되지 않는 경우 보드 공급업체의 규정에 따라 일부 데모 코드를 다시 작성해야 할 수 있습니다.

## Preparing Source Code and Docker Images

Cortex-M의 ONNC 소스 코드는 온라인에서 사용할 수 있습니다. 다음 명령을 사용하여 ONNC 소스 코드를 다운로드합니다.

```sh
$ git clone -b CortexM https://github.com/ONNC/onnc.git
```

그런 다음 다음 명령을 사용하여 자습서 소스 코드를 다운로드합니다. 이 실습에서 사용할 몇 가지 예시 DNN 모델이 있습니다.

```sh
$ git clone https://github.com/ONNC/onnc-tutorial.git
```

다음 명령을 사용하여 Docker Hub에서 Docker 이미지를 가져옵니다.

```sh
# Obtain the ONNC compilation environment.
$ docker pull onnc/onnc-community

# Obtain the ARM cross-compilation environment.
$ docker pull misegr/mbed-cli
```

Docker 이미지가 성공적으로 다운로드되었는지 확인하려면 다음 명령을 사용하여 사용 가능한 모든 Docker 이미지를 표시하십시오. `onnc/onnc-community` 및 `misegr/mbed-cli` 이미지가 모두 표시되어야 합니다.


```sh
$ docker images
REPOSITORY                           TAG                                IMAGE ID            CREATED             SIZE
onnc/onnc-community                  latest                             fdd06c76c519        2 days ago          5.58GB
misegr/mbed-cli                      latest                             a708c25bd4d9        2 weeks ago         2.85GB
```

## Building ONNC and Compiling Digit-Recognition Models

이 명령을 사용하여 ONNC 커뮤니티 Docker를 불러옵니다.

```sh
$ docker run -ti --rm -v <absolute/path/to/onnc>:/onnc/onnc -v <absolute/path/to/tutorial>:/tutorial onnc/onnc-community
```
Docker 명령어 사용법은 [lab 1: Environment Setup](../lab_1_Environment_Setup/lab_1.md)을 참고하세요. Docker 컨테이너 내에서 다음 명령을 사용하여 ONNC를 빌드합니다.

```sh
##############################################
# Within onnc/onnc-community Docker container
##############################################

$ cd /onnc/onnc-umbrella/build-normal

# Build ONNC.
$ smake -j8 install
```

이 시점까지 ONNC 바이너리가 DNN 모델을 컴파일할 준비가 되어 있어야 합니다. 앞서 언급했듯이 이 실습에 사용되는 DNN 모델은 양자화되어야 하며 모든 가중치는 8비트 정수입니다. 또한 모든 활성화 데이터에 대한 오른쪽 시프트 값이 포함된 보정 파일도 준비해야 합니다. 이 실습에서는 ONNX 모델 동물원에서 [mnist model](https://github.com/onnx/models/tree/master/vision/classification/mnist)을 얻었고, 훈련 후 양자화를 수행하여 양자화된 버전을 도출했습니다. 모든 파일이 준비되면(`<onnc-tutorial>/models/quantized_mnist/` 폴더에서 복사본을 찾을 수 있음) 다음 명령을 사용하여 모델을 컴파일하고 C 코드를 생성합니다.

```sh
##############################################
# Within onnc/onnc-community Docker container
##############################################

# Run ONNC to compile a quantized model with calibration information.
$ onnc -mquadruple cortexm /tutorial/models/quantized_mnist/quantized_mnist.onnx \
    --load-calibration-file=/tutorial/models/quantized_mnist/mnist_calibration.txt

# Check the output files of the Cortex-M backend.
$ ls cortexm*
cortexm_main.cpp  cortexm_main.h  cortexm_weight.h

# Prepare the resulting files for the later cross-compilation.
$ sudo mv cortexm* /tutorial/models/quantized_mnist
```
By now, you may find the generated files in the `<onnc-tutorial>/models/quantized_mnist/` folder. In case where you want to exit the Docker prompt, use the following command.

```sh
##############################################
# Within onnc/onnc-community Docker container
##############################################

$ exit
```

## Cross-compilation of CortexM machine code

When you finish the previous steps of building ONNC and compiling a DNN model, you do not need the `onnc/onnc-community` Docker anymore. You need to enter the other Docker image `misegr/mbed-cli` to compile the generated C code for the Cortex-M platform. 

```sh
###############################
# Within your computer console
###############################

# Move CortexM files to the onnc-cmsis-example folder
$ cd <path/to/onnc-cmsis-example>
$ cp <path/to/tutorial>/models/quantized_mnist/cortexm* .

# Enter the cross-compilation Docker.
$ docker run -ti --rm -v <absolute/path/to/onnc-cmsis-example>:/src misegr/mbed-cli bash
```

```sh
##########################################
# Within misegr/mbed-cli Docker container
##########################################

$ cd /src
$ mbed deploy
[mbed] Working path "/src" (program)
[mbed] Adding library "mbed-os" from "https://github.com/ARMmbed/mbed-os" at rev #367dbdf5145f
[mbed] Adding library "CMSIS_5" from "https://github.com/ARM-software/CMSIS_5" at rev #c4c089d6333d
[mbed] WARNING: File "RTX_V8MMF.lib" in "/src/CMSIS_5/CMSIS/RTOS2/RTX/Library/ARM" uses a non-standard .lib file extension, which is not compatible with the mbed build tools.
...
[mbed] Auto-installing missing Python modules (fuzzywuzzy)...

# Compile the firmware for a specific target by appointing the --target option.
# Here we use NuMaker_PFM_NUC472 as an example.
# Another example is DISCO_L475VG_IOT01A by STM.
$ mbed compile --target NuMaker_PFM_NUC472
[mbed] Working path "/src" (program)
Building project src (NUMAKER_PFM_NUC472, GCC_ARM)
Scan: .
Scan: env
...
Compile [ 99.7%]: serial_api.c
Compile [ 99.9%]: spi_api.c
Compile [100.0%]: test_env.cpp
Link: src
Elf2Bin: src
+------------------+--------+-------+-------+
| Module           |  .text | .data |  .bss |
+------------------+--------+-------+-------+
| CMSIS_5/CMSIS    |   1748 |     0 |     0 |
| [fill]           |    471 |    25 |    23 |
| [lib]/c.a        |  63801 |  2548 |   127 |
| [lib]/gcc.a      |   7200 |     0 |     0 |
| [lib]/misc       |    252 |    12 |    28 |
| [lib]/nosys.a    |     32 |     0 |     0 |
| [lib]/stdc++.a   | 171534 |   165 |  5676 |
| add.o            |    192 |     4 |     1 |
| cortexm_main.o   |    384 |  6082 | 15768 |
| main.o           |    344 |     4 |  4200 |
| matmul.o         |    118 |     0 |     0 |
| mbed-os/drivers  |   1219 |     0 |     0 |
| mbed-os/features |    112 |     0 | 12345 |
| mbed-os/hal      |   1720 |     4 |    68 |
| mbed-os/platform |   3934 |   256 |   105 |
| mbed-os/rtos     |  10917 |   168 |  6073 |
| mbed-os/targets  |   5656 |   212 |   142 |
| Subtotals        | 269634 |  9480 | 44556 |
+------------------+--------+-------+-------+
Total Static RAM memory (data + bss): 54036 bytes
Total Flash memory (text + data): 279114 bytes

Image: ./BUILD/NUMAKER_PFM_NUC472/GCC_ARM/src.bin
```

The generated firmware binary file is located at <path/to/onnc-cmsis-example>/BUILD/NUMAKER_PFM_NUC472/GCC_ARM/src.bin. You can upload it to the board following the suggestion from the board vendor. The procedure is simple for an Mbed-compatible board. Connect the target board to a (Mac, Linux, or Windows) computer via a USB cable. Then you should see an Mbed drive to appear in the file browser window. Copy the `src.bin` file into that drive.

## Digit Recognition Demo

The demo setup is shown as below.

<img src="../figures/mnist_demo_setup.png" width="450">

The board is connected to a PC via the UART connection. On the PC, we have prepared a GUI software where you can draw digit numbers on. Please open the [GUI program](mnist_demo_gui/mnist_demo_gui.pde) by Processing as shown in the following diagram. The file path is `<path/to/tutorial>/lab_2_Digit_Recognition_with_ARM_CortexM/mnist_demo_gui/mnist_demo_gui.pde`.

<img src="../figures/processing_open_file.png" width="300">

Then run the program by clicking the "run" button as below.

<img src="../figures/processing_run.png" width="400">

This demo accepts only one single-digit numnber at a time. Once you are done and click the "Submit" button on the GUI, the software will take a screenshot, transform it into a 28x28 image, and send the image to the board via the UART connection. The board will perform the model inference, and then send the classification answer back to the PC.
A screenshot of the demo is shown as below.

<img src="../figures/mnist_demo.gif" width="400">

