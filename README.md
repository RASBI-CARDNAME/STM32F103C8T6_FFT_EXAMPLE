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
