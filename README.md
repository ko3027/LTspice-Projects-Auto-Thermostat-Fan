# LTspice Automatic Temperature Control Fan

> NTC Thermistor, Comparator, NE555, MOSFET 및 Fan 모델을 이용한  
> **폐루프 자동 온도 제어 시스템 설계 및 LTspice 시뮬레이션 프로젝트**

---

## Project Overview

본 프로젝트는 열원의 온도가 일정 수준 이상으로 상승하면 냉각팬을 자동으로 작동시키고,
온도가 충분히 감소하면 팬을 정지시키는 **자동 온도 제어 시스템**을 LTspice로 설계한 프로젝트입니다.

단순한 ON/OFF 회로에 그치지 않고,

**온도 감지 → 임계값 판단 → PWM 생성 → MOSFET 스위칭 → 팬 구동 → 냉각 → 다시 온도 변화**

로 이어지는 폐루프(Closed-loop) 구조를 구현하였습니다.

프로젝트 과정에서는 NTC Thermistor의 온도 특성, Comparator의 히스테리시스,
NE555 PWM 발생, MOSFET 스위칭, 유도성 부하 보호 및 열 등가회로 모델링을 단계적으로 학습하고 검증하였습니다.

---

## System Architecture

```mermaid
flowchart LR
    A["Heat Source"] --> B["NTC Thermistor"]
    B --> C["LT1016 Comparator"]
    C --> D["NE555 PWM"]
    D --> E["NMOS"]
    E --> F["5V Fan Model"]
    F --> G["Cooling"]
    G --> A


전체 신호 흐름은 다음과 같습니다.


열원 온도
   ↓
NTC Thermistor
   ↓
온도 → 센서 전압 변환
   ↓
LT1016 Comparator
   ↓
히스테리시스 기반 ON/OFF 판단
   ↓
NE555 PWM
   ↓
NMOS Gate
   ↓
5V Fan Model
   ↓
RPM 신호
   ↓
냉각량 변화
   ↓
열원 온도에 Feedback

Design Target

초기 설계 목표는 다음과 같습니다.

약 50℃에서 팬 작동
약 45℃까지 냉각되면 팬 정지
임계온도 부근의 반복적인 ON/OFF 현상 방지
MOSFET을 이용한 팬 전력 스위칭
유도성 부하의 역기전력 보호
온도와 팬 동작이 서로 영향을 주는 폐루프 구현
LTspice Transient Simulation을 통한 전체 시스템 검증
Final Simulation Result

최종 회로의 Comparator 기준전압과 NTC 특성을 계산한 결과
다음과 같은 온도 임계값을 얻었습니다.

Parameter	Result
Fan ON Temperature	약 49.3℃
Fan OFF Temperature	약 45.5℃
Hysteresis Width	약 3.8℃
Temperature Range	약 45~49℃
Comparator Output	약 0~3.7V
NE555 Output	약 0~5V
MOSFET Gate	약 0~5V

시뮬레이션에서는 팬이 정지하면 열원 온도가 상승하고,
상한 임계온도에 도달하면 팬이 동작하여 온도가 감소하는 것을 확인하였습니다.

온도가 하한 임계값까지 감소하면 팬이 다시 정지하고,
이 과정이 반복되면서 열원 온도가 설정 범위 안에서 유지됩니다.

Key Circuit Blocks
Block	Component	Function
Thermal Model	B_TH, C_TH	가열·자연 방열·팬 냉각 및 열관성 모델링
Temperature Sensor	NTC Thermistor	온도 변화 → 저항 변화
Voltage Divider	R3 + NTC	NTC 저항 → 센서 전압 변환
Reference Voltage	R6 10kΩ + R9 3.9kΩ	Comparator 기준전압 생성
Hysteresis	R5 68kΩ	Fan ON/OFF 임계온도 분리
Comparator	LT1016	센서전압과 기준전압 비교
PWM Generator	NE555	MOSFET Gate 구동 신호 생성
Gate Resistor	R2 100Ω	Gate 충·방전 전류 제한
Power Switch	NMOS	Fan 전류 Low-side Switching
Protection	1N5819	유도성 부하 역기전력 억제
Fan	FAN5V.lib	5V Fan 및 RPM 신호 모델
Thermal Model

LTspice는 기본적으로 전압과 전류를 계산하는 회로 시뮬레이터이기 때문에,
본 프로젝트에서는 온도를 다음과 같이 전압으로 대응시켰습니다.

1 V = 1 ℃

따라서,

V(TEMP_MON) = 48 V

는 실제 회로에 48V가 인가된다는 의미가 아니라
열 모델에서 48℃를 나타내는 계산용 값입니다.

사용한 주요 열 모델 파라미터는 다음과 같습니다.

.param Tamb=25
.param Rth=5
.param Cth=0.5
.param Pheat=6
.param Pcool=15

C_TH TEMP_MON 0 {Cth} IC=40

B_TH 0 TEMP_MON I={
 Pheat
 -(V(TEMP_MON)-Tamb)/Rth
 -Pcool*if(V(RPM)>0.5,1,0)
}

팬이 작동하지 않을 경우 열원의 평형온도는

Teq = Tamb + Pheat × Rth
    = 25 + 6 × 5
    = 55℃

이므로 약 55℃를 향해 상승합니다.

팬이 작동하면 냉각항 Pcool이 추가되고 온도가 감소합니다.

Temperature Sensing

NTC Thermistor에는 Beta 모델을 적용하였습니다.

.param Beta=3950
.param R25=10000

NTC의 저항은 온도가 상승할수록 감소합니다.

Temperature ↑
      ↓
NTC Resistance ↓
      ↓
Sensor Voltage ↓
      ↓
Comparator State Change

이를 통해 실제 온도 변화가 Comparator에서 비교 가능한 전압 신호로 변환됩니다.

Comparator & Hysteresis

하나의 임계온도만 사용하면 온도가 경계점 주변에서 미세하게 변할 때
팬이 매우 빠르게 ON/OFF를 반복하는 Chattering이 발생할 수 있습니다.

이를 방지하기 위해 LT1016 출력 일부를
R5 68kΩ을 통해 기준전압 측으로 Positive Feedback 하였습니다.

그 결과 두 개의 서로 다른 임계온도가 형성됩니다.

Temperature Rising
        ↓
     49.3℃
        ↓
      FAN ON
        ↓
     Cooling
        ↓
     45.5℃
        ↓
      FAN OFF

이를 통해 약 3.8℃의 Hysteresis Band를 구현하였습니다.

Power Switching Stage

Comparator의 출력으로 팬을 직접 구동하지 않고
NMOS를 Low-side Switch로 사용하였습니다.

NE555 PWM
    ↓
Gate Resistor
    ↓
NMOS Gate
    ↓
NMOS Switching
    ↓
Fan Current

MOSFET Gate 앞에는 100Ω 저항을 사용하여
Gate 충·방전 과정의 순간 전류를 제한하였습니다.

또한 팬은 유도성 부하이므로 MOSFET OFF 시 발생하는 역기전력으로부터
스위칭 소자를 보호하기 위해 1N5819 Schottky Diode를 사용하였습니다.

Engineering Challenges

프로젝트를 진행하면서 단순히 최종 회로를 구성하는 것보다
문제의 원인을 분석하고 회로를 단계적으로 개선하는 과정을 중요하게 다루었습니다.

대표적인 시행착오는 다음과 같습니다.

01. Comparator 단일 임계값 문제

하나의 기준온도만 사용할 경우 임계점 주변에서
Fan ON/OFF 상태가 불안정해질 수 있음을 확인하였습니다.

→ Comparator Positive Feedback을 이용한 Hysteresis 적용

02. LTspice Reserved Keyword 충돌

NTC 온도 계산에 Temp라는 변수명을 사용했으나
LTspice 내부 예약어와 충돌하여 Simulation이 실행되지 않았습니다.

→ 사용자 정의 변수명을 변경하여 해결

03. RL Load와 실제 Motor Model의 차이

초기에는 Fan을 단순 RL 직렬회로로 모델링하였습니다.

하지만 RL 모델만으로는

회전속도
역기전력
기동 특성
기계적 관성

등 실제 Motor 특성을 표현하기 어렵다는 한계를 확인하였습니다.

→ Fan Macro Model을 이용하는 방향으로 개선

04. MOSFET Source Resistance 문제

Source와 Ground 사이에 큰 저항을 삽입하면 Source 전압이 상승하여
VGS가 감소하고 MOSFET이 충분히 ON되지 않을 수 있음을 확인하였습니다.

→ Low-side Switching 구조를 재검토

05. Flyback Diode 누락

유도성 부하를 MOSFET으로 Switching할 때
MOSFET OFF 순간 높은 역전압이 발생할 수 있음을 학습하였습니다.

→ 1N5819 Flyback Diode 추가

자세한 과정은 아래 시행착오 문서에 정리하였습니다.

시행착오 기록

What I Learned

이 프로젝트를 통해 다음 내용을 학습하고 실제 회로에 적용하였습니다.

Analog Circuit
Voltage Divider
Reference Voltage
Comparator
Positive Feedback
Hysteresis
NTC Thermistor
Power Electronics
NMOS Low-side Switching
VGS / VDS / IDS
Gate Resistance
Inductive Load
Back EMF
Flyback Diode
Circuit Simulation
LTspice Transient Analysis
Behavioral Source
Parameter Modeling
NTC Beta Equation
Thermal Equivalent Circuit
Macro Model
Waveform Analysis
Engineering Process
문제 재현
원인 분석
가설 수립
회로 수정
재시뮬레이션
결과 비교
설계 한계 분석
Documentation

프로젝트의 상세 설계 과정은 아래 문서에서 확인할 수 있습니다.

Document	Description
01. 소자 공부	프로젝트에 사용된 주요 소자 및 회로 개념
02. 동작 원리	전체 회로 및 각 블록의 동작 원리
03. 피드백 전 설계	초기 회로와 폐루프 적용 전 설계
04. 시행착오	설계 과정의 문제와 개선 과정
05. AI 활용 및 회로 검증	AI를 활용한 문제 분석과 회로 검증 과정
Simulation	최종 회로 및 시뮬레이션 상세 분석
Final Result	최종 설계 결과
Project Focus

본 프로젝트의 목적은 단순히 팬이 동작하는 회로를 만드는 것이 아니라,

센서 신호를 아날로그 회로로 처리하고, 제어 신호를 생성하여 MOSFET으로 전력 부하를 구동한 뒤 그 결과를 다시 센서 입력에 반영하는 전체 신호 흐름을 이해하는 것

에 있습니다.

이를 통해 Analog Circuit, Power Semiconductor Switching,
Sensor Interface 및 Circuit Simulation 사이의 연관성을 학습하였습니다.

Tools
LTspice
Markdown
GitHub
Circuit Simulation
AI-assisted Debugging & Verification
