# STM32F103C8T6 FFT Example

STM32F103C8T6에서의 빠른 푸리에 변환(FFT) 예제입니다.

---

## 요구사항

1. [I2C LCD 라이브러리](https://github.com/alixahedi/I2C-LCD-STM32)  
2. [CMSIS DSP 라이브러리](https://github.com/ARM-software/CMSIS-DSP)
3. [CMSIS 6 라이브러리](https://github.com/ARM-software/CMSIS_6) 

**적용 방법:**  
각 라이브러리는 프로젝트에 맞게 설치하고 포함시켜 사용하세요.

CMSIS 라이브러리 [적용법](https://blog.naver.com/hiho0718/224008803960)

---

## 코드 설명

```c
arm_rfft_fast_f32(&arfi32, fft, fft, 0);
arm_cmplx_mag_f32(fft, fft, sample_size/2);
```

단순히 FFT 함수를 호출하고, 복소수의 크기를 구해 FFT를 마무리 합니다. 기본적인 동작입니다.

라이센스
없습니다. 자유롭게 쓰세요. 단, 라이브러리에는 라이센스가 있으니 꼭 확인하고 사용하세요.


# 📊 STM32 실시간 FFT 스펙트럼 분석기 (CMSIS-DSP)

## 1. 프로젝트 개요 (Overview)
STM32F103의 ADC와 **ARM CMSIS-DSP 라이브러리**를 활용하여 오디오/센서 신호를 실시간으로 주파수 도메인으로 변환(FFT)하고, 주요 주파수 성분을 I2C LCD에 출력하는 프로젝트입니다.
단순한 라이브러리 호출을 넘어, **ADC 샘플링 레이트와 주파수 분해능(Bin Resolution)의 관계를 분석**하여 정확한 주파수 값을 도출하는 데 초점을 맞췄습니다.

*   **기능:** 1024-Point Real-FFT 연산, 주요 주파수(Dominant Frequency) 탐지, LCD 시각화
*   **활용:** 진동 분석, 오디오 튜너, 신호 모니터링 시스템

---

## 2. 기술 스택 (Tech Stack)
*   **HW:** STM32F103C8T6, I2C LCD (16x2)
*   **SW:** C, STM32 HAL, CMSIS-DSP Library (v1.10.0 이상)
*   **Math:** Fast Fourier Transform (RFFT), Complex Magnitude

---

## 3. 요구사항

1. [I2C LCD 라이브러리](https://github.com/alixahedi/I2C-LCD-STM32)  
2. [CMSIS DSP 라이브러리](https://github.com/ARM-software/CMSIS-DSP)
3. [CMSIS 6 라이브러리](https://github.com/ARM-software/CMSIS_6) 

**적용 방법:**  
각 라이브러리는 프로젝트에 맞게 설치하고 포함시켜 사용하세요.

CMSIS 라이브러리 [적용법](https://blog.naver.com/hiho0718/224008803960)

---

## 4. 시스템 구성 및 성능 (Configuration & Performance)

### ⚙️ ADC 설정 및 샘플링 이론
정확한 주파수 분석을 위해 ADC 변환 속도를 정밀하게 계산하여 설정했습니다.

*   **ADC Clock:** 4 MHz (APB2 / 2, HSI 8MHz 사용함)
*   **Sampling Cycles:** 14 Cycles (1.5 Sampling + 12.5 Conversion)
*   **Sampling Rate (Fs):** 약 **285.7 KHz** (4,000,000 / 14)
*   **FFT Points (N):** 1024
*   **Frequency Resolution (Bin Size):** 약 **279 Hz** (Fs / N = 285,700 / 1024)

### 🔄 데이터 처리 흐름
1.  **ADC(DMA):** 1024개의 아날로그 샘플을 원형 버퍼(Circular Buffer)로 수집
2.  **RFFT 연산:** `arm_rfft_fast_f32` 함수를 사용하여 실수(Real) 데이터를 복소수(Complex)로 변환
3.  **Magnitude 계산:** 복소수 결과의 절대값(크기)을 계산하여 스펙트럼 도출
4.  **Output:** 버튼 입력에 따른 주파수 탐색 결과를 LCD에 출력

---

## 5. 핵심 구현 코드 (Core Implementation)

이 프로젝트는 실수 입력에 최적화된 **RFFT(Real-FFT)** 알고리즘을 사용하여 연산 속도를 높였습니다.

```c
// 1. RFFT 연산 수행 (실수 입력 -> 복소수 출력)
// RFFT는 입력 배열(fft_input)을 변환하여 출력 배열(fft_output)에 저장함
arm_rfft_fast_f32(&fft_handler, fft_input, fft_output, 0);

// 2. Magnitude(크기) 계산
// 복소수 형태의 FFT 결과에서 진폭(Magnitude)을 추출
// 출력 데이터의 절반(1024/2 = 512)까지만 유효함 (Nyquist 이론)
arm_cmplx_mag_f32(fft_output, fft_magnitude, FFT_LENGTH / 2);

```

---

## 6. 트러블 슈팅 (Troubleshooting) 🛠️
### Q. 주파수 오차 및 해상도(Resolution) 문제
1kHz 사인파를 입력했을 때, 정확히 1000Hz가 아닌 1116Hz 또는 837Hz 등으로 표시되는 현상. 원인은 FFT의 주파수 해상도(Bin Size)가 약 279Hz이기 때문에, 측정 결과는 279Hz의 정수배로만 표현됨 (Quantization Error).
### 해결:
*   이는 FFT 알고리즘의 수학적 특성임을 이해하고 분석함.
*   정밀도를 높이려면 샘플링 레이트를 낮추거나(Decimation), **샘플 수(N)를 늘려야 함(2048, 4096 등)**을 확인함.
  
### Q. CMSIS 라이브러리 링킹 에러
STM32CubeIDE에서 라이브러리 추가 시 Undefined reference 에러 발생.
### 해결:
* 프로젝트에서 중복 선언으로 인식되는 파일을 제거하여 해결. (상세 과정은 [블로그](https://blog.naver.com/hiho0718/224008803960) 참조)

---

## 📄 라이선스
MIT 라이선스 – 자세한 내용은 [`LICENSE`](LICENSE)를 참조하십시오.
