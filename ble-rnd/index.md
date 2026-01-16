---
layout: project
title: "BLE National R&D Project"
subtitle: "BLE 연결/명령 신뢰성을 높이기 위한 검증·상태관리 설계를 적용한 사례"
period: "2021.01 – 2023.12"
role: "Android Developer (BLE Protocol / Device Control)"
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
BLE 특성상 환경에 따라 품질 편차가 커서, 데이터 무결성과 응답 순서를 보장하지 못하면 실사용에서 오동작이 발생했습니다. 검증 계층과 상태 흐름을 강화해 신뢰성을 끌어올렸습니다.

---

## 2. Background & Problem
- **불안정한 연결 품질**: BLE 특성상 환경에 따라 연결/응답 품질 편차가 큼
- **데이터 무결성 이슈**: 패킷 유실/중복/순서 뒤틀림으로 상태 오염 가능성 존재
- **디버깅 난이도**: 콜백 중심 흐름으로 인해 이슈 재현 및 추적이 어려움

---

## 3. Technical Strategy

### A. 통신 신뢰성 확보 (Reliability)
- **Checksum & Packet Validation**:
  - **How**: 패킷 파싱 시 `CRC-16` 알고리즘을 적용해 수신된 바이트 배열의 무결성을 검증하고, 검증 실패 시 즉시 폐기(Drop)
  - **Why**: 무선 환경(BLE)의 노이즈로 인한 비트 플립(Bit Flip)이 제어 로직에 반영되는 치명적 오동작을 원천 차단

- **Timestamp & Sequence**:
  - **How**: 송신 측 타임스탬프를 패킷에 포함하고, 수신 측에서 `LastProcessedTime`과 비교하여 오래된 패킷(Out-of-order)이나 중복 패킷을 필터링
  - **Why**: BLE 재전송 메커니즘으로 인해 동일 명령이 중복 실행되거나 순서가 뒤바뀌는 문제를 해결

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
  - **How**: Kotlin `Sealed Class`로 연결 상태(Disconnected, Scanning, Connecting, Ready)를 정의하고, `StateFlow`를 통해 UI와 로직에 전파
  - **Why**: 기존의 중첩된 콜백(Callback Hell) 구조에서는 예외 처리(연결 끊김 등)가 누락되기 쉬우나, FSM은 모든 상태 전이를 명시적으로 강제하여 안정성 확보

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
- 데이터 오염/오동작 리스크 감소
- 실사용 환경에서 연결/명령 흐름 안정화(정성)
- 5개 협력 기관과의 인터페이스 표준화 및 일정 준수

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
- 검증/상태 관리 로직 추가로 복잡도는 증가했지만, 현장 안정성을 우선
- **KC 인증(EMC/ESD) 대응**:
  - 정전기(ESD) 테스트 중 BLE 스택이 죽는 현상이 발생했을 때, 앱 레벨에서 `BluetoothAdapter` 상태를 감지하여 자동으로 서비스를 재시작하는 **Self-Healing** 로직 구현
  - 이를 통해 하드웨어적 불안정 상황에서도 사용자 개입 없이 복구되는 시스템 구축
