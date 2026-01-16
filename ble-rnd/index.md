---
layout: project
title: "BLE National R&D Project"
subtitle: "BLE 연결/명령 신뢰성을 높이기 위한 검증·상태관리 설계를 적용한 사례"
period: "2021.01 – 2023.12"
featured: false
tags:
  - Android
  - Kotlin
  - MVVM
  - Clean Architecture
  - BLE
  - NFC
metrics:
  - { value: "Checksum", label: "무결성 검증" }
  - { value: "Timestamp", label: "중복/순서 관리" }
  - { value: "Stability", label: "실사용 안정화" }
---

## 1. Project Overview
BLE 통신은 주변 환경에 따라 품질 편차가 크기 때문에, 데이터 무결성과 응답 순서가 보장되지 않을 경우 오동작 위험이 높습니다. 이를 해결하기 위해 검증 계층을 도입하고 상태 흐름을 체계화하여 통신 신뢰성을 확보했습니다.

---

## 2. Background & Problem
- **불안정한 연결 품질**: 무선 환경 특성상 연결 및 응답 품질의 편차가 심해 안정적인 통신 보장이 어려움
- **데이터 무결성 훼손**: 패킷 유실, 중복 수신, 순서 뒤바뀜 등으로 인한 데이터 및 상태 오염 발생
- **높은 디버깅 난이도**: 비동기 콜백 중심의 복잡한 흐름으로 인해 문제 재현과 원인 추적이 난해함

---

## 3. Technical Strategy

### A. 통신 신뢰성 확보 (Reliability)
- **Checksum & Packet Validation**:
  - **How**: 수신된 바이트 배열 파싱 시 `CRC-16` 알고리즘으로 무결성을 검증하며, 검증 실패 패킷은 즉시 폐기(Drop)하여 로직 오염 방지
  - **Why**: 무선 통신 노이즈로 발생할 수 있는 비트 플립(Bit Flip)이 비즈니스 로직에 영향을 주지 않도록 원천 차단

- **Timestamp & Sequence**:
  - **How**: 송신 시 타임스탬프를 패킷에 포함시키고, 수신 측에서 `LastProcessedTime`과 대조하여 순서가 뒤바뀌거나 중복된 패킷을 필터링
  - **Why**: BLE 재전송 특성으로 인해 발생할 수 있는 명령 중복 실행 및 순서 역전 문제를 구조적으로 해결

<div class="mermaid">
sequenceDiagram
    participant App
    participant Device
    App->>Device: Request Command
    Device-->>App: Response Packet
    Note over App: 1. Check CRC-16
    alt CRC Valid
        Note over App: 2. Check Timestamp
        App->>UI: Update State
    else CRC Invalid
        App->>App: Drop Packet
    end
</div>

### B. 상태 기반 흐름 제어 (State Machine)
- **Finite State Machine (FSM)**:
  - **How**: 연결 상태(Disconnected, Scanning, Connecting, Ready)를 Kotlin `Sealed Class`로 정의하고, `StateFlow`를 통해 UI 및 로직에 일관되게 전파
  - **Why**: 중첩된 콜백(Callback Hell) 구조에서 발생하기 쉬운 예외 처리 누락을 방지하고, FSM을 통해 모든 상태 전이를 명시적으로 제어하여 안정성 확보

<div class="mermaid">
stateDiagram-v2
    [*] --> Disconnected
    Disconnected --> Scanning : User Action / Auto Retry
    Scanning --> Connecting : Device Found
    Connecting --> Ready : Connection Success
    Connecting --> Disconnected : Connection Fail
    Ready --> Disconnected : Error / User Action
</div>

---

## 4. Key Results
- 데이터 오염 및 기기 오동작 리스크 획기적 감소
- 실제 현장 환경에서의 연결 및 명령 처리 흐름 안정화 달성
- 5개 협력 기관과의 인터페이스 표준화 주도 및 개발 일정 준수

---

## 5. Engineering Evidence
단순한 연결 구현을 넘어, **데이터 무결성을 보장하기 위해 설계한 패킷 구조**입니다.

**[Packet Structure Design]**
```text
| STX (1B) | LEN (1B) | CMD (1B) | PAYLOAD (N) | TIMESTAMP (4B) | CRC16 (2B) | ETX (1B) |
```
- **CRC16**: Payload 변조 여부를 하드웨어 레벨이 아닌 앱 레벨에서 2차 검증
- **TIMESTAMP**: `System.currentTimeMillis()`를 4바이트로 압축하여 패킷 순서 보장

> **Measurement**: 1주일간 50대 디바이스의 수신 로그를 수집하여 분석한 결과, CRC 에러로 버려지는 패킷 비율이 약 3~5%로 측정되었습니다. 이를 방어하지 않았다면 데이터 오염으로 이어졌을 것입니다.

---

## 6. Deep Dive & Trade-offs
- 검증 및 상태 관리 로직 도입으로 코드 복잡도는 다소 증가했으나, 현장 운영의 안정성을 최우선 가치로 둠
- **KC 인증(EMC/ESD) 대응**:
  - 정전기(ESD) 테스트 중 BLE 스택이 중단되는 현상 발생 시, 앱 레벨에서 `BluetoothAdapter` 상태를 감지하고 서비스를 자동 재시작하는 **Self-Healing** 메커니즘 구현
  - 하드웨어적 불안정 상황에서도 사용자 개입 없이 시스템이 스스로 복구되는 복원력(Resilience) 확보
