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
