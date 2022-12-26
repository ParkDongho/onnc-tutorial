## Introduction

NVIDIA Deep Learning Accelerator는 추론 애플리케이션에 심층 신경망을 사용하는 칩을 구축하려는 모든 사람에게 무료 지적 재산 라이선스를 제공합니다. 광범위한 문서 및 도구를 사용하여 많은 비즈니스 제안 및 연구 프로젝트에서 추론 엔진 설계로 NVDLA를 선택합니다. 그러나 확장 가능한 컴파일러 지원의 부족은 더 많은 AI 모델과 최적화를 지원하는 데 있어 주요 병목 현상이 됩니다. 이 튜토리얼은 NVDLA 기반 설계를 지원하는 최초의 오픈 소스 컴파일러를 제시합니다. ONNC 컴파일러는 공식 NVDLA 컴파일러보다 더 많은 지원을 제공하며 프로그래머가 공식 NVDLA 컴파일러에서 지원하지 않는 모델의 하위 수준 세부 정보를 수동으로 지정하지 않아도 됩니다. 또한 하드웨어 맞춤화 및 독점 최적화의 기회를 제공합니다. 세 개의 하위 섹션에서 개요, 포팅 및 최적화를 다룰 것입니다. 각 하위 섹션에는 제품 개발 및 연구 프로젝트를 위해 ONNC에서 NVDLA 백엔드를 실행하고 사용자 정의하는 방법을 보여주는 실습 랩이 있습니다.

ONNC(Open Neural Network Compiler)는 독점 딥 러닝 가속기를 위해 특별히 설계된 대상 변경 가능한 컴파일 프레임워크입니다. 소프트웨어 아키텍처는 ONNC를 [ONNX(Open Neural Network Exchange)](https://onnx.ai/) 연산자를 지원하는 DLA(Deep Learning Accelerator) 설계로 신속하게 포팅합니다. ONNC는 ONNX 모델을 DLA 관련 이진 형식으로 변환하고 효과적인 알고리즘과 함께 ONNX의 IR(중간 표현) 디자인을 활용하여 데이터 이동의 오버헤드를 제거함으로써 모든 DLA에서 실행 가능성을 보장합니다. **ONNC는 NVDLA 기반 하드웨어 설계에 사용할 수 있는 최초의 오픈 소스 컴파일러입니다**. NVDLA 백엔드는 모델을 실행 가능한 NVDLA Loadable 파일로 컴파일할 수 있습니다. ONNC를 NVDLA 소프트웨어 스택과 통합하면 개발자와 연구원이 시스템 수준에서 NVDLA 기반 추론 설계를 탐색할 수 있는 기회가 열립니다.

이 튜토리얼은 오하이오주 콜럼버스에서 개최된 [MICRO 2019: The 52nd IEEE/ACM International Symposium on Microarchitecture (10월 12일)](https://www.microarch.org/micro52/program/workshops.html#onnc)에서 발표되었습니다.

## Intended Audience

Researchers and practitioners in academia or industry looking for an open-source AI compiler for NVDLA-based neural network inference engines.

## Contributors

* Wei-Fen Lin (weifen@skymizer.com)
* Cheng-Tao Hsieh (cthsieh@skymizer.com)

## Hands-on Labs

* Lab 1. [ONNC Working Environment Setup](https://github.com/ONNC/onnc-tutorial/blob/master/lab_1_Environment_Setup/lab_1.md)
* Lab 2. [Digit Recognition with ARM Cortex-M](https://github.com/ONNC/onnc-tutorial/blob/master/lab_2_Digit_Recognition_with_ARM_CortexM/lab_2.md)
* Lab 3. [Starting a New Backend](https://github.com/ONNC/onnc-tutorial/blob/master/lab_3_Starting_New_Backend/lab_3.md)
* Lab 4. [Code Emitting](https://github.com/ONNC/onnc-tutorial/blob/master/lab_4_Code_Emitting/lab_4.md)
* Lab 5. [CPU Fallback Support](https://github.com/ONNC/onnc-tutorial/blob/master/lab_5_CPU_Fallback/lab_5.md)
* Lab 6. [Manipulating ONNC IR and Optimization](https://github.com/ONNC/onnc-tutorial/blob/master/lab_6_Manipulating_ONNC_IR/lab_6.md)
* Lab 7. [ONNC IR Extension](https://github.com/ONNC/onnc-tutorial/blob/master/lab_7_ONNC_IR_Extension/lab_7.md)
* Lab 8. [Hardware-specific Optimization](https://github.com/ONNC/onnc-tutorial/blob/master/lab_8_Mul_Add_Reordering_and_Fusion/lab_8.md)

## References

### Papers

* W. F. Lin, D. Y. Tsai, L. Tang, C. T. Hsieh, C. Y. Chou, P. H. Chang, and L. Hsu, “ONNC: A compilation framework connecting ONNX to proprietary deep learning accelerators,” in IEEE International Conference on Artificial Intelligence Circuits and Systems (AICAS 2019). IEEE, 2019. 
Download PDF: [Link](https://skymizer.com/publications/Skymizer-AICAS2019.pdf)


* W.F. Lin, C. T. Hsieh, C. Y. Chou, "ONNC-based Software Development Platform for Configurable NVDLA Designs", to appear in IEEE International Symposium on VLSI Design, Automation and Test (VLSI-DAT 2019). IEEE, 2019
Download PDF: [Link](https://skymizer.com/publications/Skymizer-VLSIDAT2019.pdf)

### Documentation

- [ONNC Utilities](https://github.com/ONNC/onnc/blob/master/docs/ONNC-Utilities.md)
- [ONNC Pass Manager Getting Started Guide](https://github.com/ONNC/onnc/blob/master/docs/ONNC-Pass-Manager-Getting-Started-Guide.md)
- [ONNC Backend Developer Guide](https://github.com/ONNC/onnc/blob/master/docs/ONNC-Backend-Porting-Guide.md)
- [The Code Emitting Pass User Guide](https://github.com/ONNC/onnc/blob/master/docs/The-Code-Emitting-Pass-User-Guide.md)
- [ONNC IR Extension Guide](https://github.com/ONNC/onnc/blob/master/docs/ONNC-IR-Extension-Guide.md)


