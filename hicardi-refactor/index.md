---
layout: project
title: "Hicardi Refactor"
subtitle: "레거시 Java 기반 Android 앱을 Kotlin · MVI/Clean Architecture · 멀티모듈로 재설계하고, build-logic/CI·CD로 빌드·배포 시간을 정량 개선한 사례"
period: "2025.09 – 2025.12"
role: "Android Developer (Refactor / Build & CI·CD Owner)"
featured: true
tags:
  - Android
  - Kotlin
  - MVI
  - Clean Architecture
  - Multi-module
  - build-logic
  - Gradle(KTS)
  - GitHub Actions
  - JUnit5
metrics:
  - { value: "78%", label: "빌드 시간 단축" }
  - { value: "66%", label: "배포 소요 시간 감소" }
  - { value: "60%", label: "테스트 커버리지" }
---

## 1. Project Overview
기능 확장이 반복되면서 레거시 구조의 변경 영향도가 커졌고, 빌드·배포 시간이 늘어나 개발 리드타임이 병목이 되었습니다.  
단순한 기능 추가가 아니라 **아키텍처 안정화와 개발 생산성 회복을 동시에 달성해야 하는 상황**이었습니다.

---

## 2. Background & Problem
- **레거시(Java) 구조**: UI·도메인·데이터 계층의 책임 경계가 불명확
- **모듈 간 의존성 관리 부재**: 느슨해진 의존성으로 빌드 시간이 지속적으로 증가
- **테스트 기반 부족**: 리팩토링 시 안정성을 보장하기 어려움
- **수동 배포 프로세스**: 배포 리드타임이 길고 예측이 어려움

---

## 3. Technical Strategy

### A. Architecture: MVI + Clean Architecture
- **Unidirectional Data Flow (UDF)**:
  - **How**: 사용자의 의도(Intent)를 `Sealed Interface`로 정의하고, Reducer를 통해 불변 상태(Immutable State)만 UI로 발행
  - **Why**: 의료 데이터 특성상 UI와 실제 데이터의 불일치가 치명적이므로, 상태 변경의 원천(Source of Truth)을 하나로 통제하여 사이드 이펙트를 차단

- **UseCase Separation**:
  - **How**: 비즈니스 로직을 ViewModel에서 분리하여 순수 Kotlin 모듈(Domain Layer)로 격리, 안드로이드 의존성 제거

### B. Modularization & Build Logic
- **Multi-module Strategy**:
  - **How**: `feature`(화면), `core`(공통 로직), `domain`(비즈니스) 계층으로 모듈을 분리하고, `implementation` 의존성을 통해 컴파일 파이프라인 최적화
  - **Why**: 모놀리식 구조에서는 코드 한 줄 수정에도 전체 재빌드가 발생했으나, 모듈화를 통해 변경된 모듈만 빌드(Incremental Build)하여 생산성 극대화

- **Convention Plugins (build-logic)**:
  - **How**: `buildSrc` 대신 `included build` 방식을 도입하여 `AndroidHiltPlugin`, `JvmLibraryPlugin` 등 반복되는 Gradle 설정을 플러그인으로 모듈화
  - **Why**: 다수 모듈의 중복 설정을 중앙에서 관리함으로써 라이브러리 버전 불일치를 방지하고 빌드 스크립트 가독성 향상

### C. CI·CD: GitHub Actions
- **Pipeline Optimization**:
  - **How**: `Gradle Build Cache` 및 `Docker Layer Caching`을 적용하여 중복되는 빌드 작업을 생략하도록 파이프라인 최적화
  - **Why**: 배포 시간을 30분에서 10분으로 단축시킴으로써 QA 피드백 주기를 일 1회에서 4회 이상으로 증대 (Business Agility 확보)

<div class="mermaid">
flowchart LR
    subgraph CI [CI: Continuous Integration]
        direction LR
        Push[Git Push] --> Build[Gradle Build]
        Build --> Test[Unit Test]
    end
    subgraph CD [CD: Continuous Deployment]
        direction LR
        Artifact[Build APK] --> ApiCall[API Call: Internal Deploy]
    end
    Test --> Artifact
    style Build fill:#e1f5fe
    style Test fill:#fff9c4
    style Artifact fill:#e8f5e9
    style ApiCall fill:#e8f5e9
</div>

---

## 4. Key Results
- **빌드 시간 78% 단축** (약 7m → 1.5m)
- **배포 소요 시간 66% 감소** (약 30m → 10m)
- **JUnit 기반 테스트 커버리지 60% 달성**
- 리팩토링 후 기능 추가 시 변경 영향 범위를 명확히 예측 가능하도록 구조 개선

---

## 5. Evidence & Benchmarks
면접관님께 신뢰도 높은 데이터를 제공하기 위해 **Gradle Build Scan**과 **CI 로그**를 기반으로 측정한 상세 지표입니다.

| 측정 항목 (Metric) | Legacy (Monolithic) | Refactored (Multi-module) | 개선율 |
| :--- | :--- | :--- | :--- |
| **Clean Build** | 평균 7분 20초 | **평균 1분 35초** | ▼ 78% |
| **Incremental Build** | 평균 45초 | **평균 8초** | ▼ 82% |
| **CI Pipeline** | 30분 (Test 생략 포함) | **10분 (Test 전체 수행)** | ▼ 66% |

> **Note**: Incremental Build는 `configuration-cache`가 적용된 상태에서 단일 모듈 수정 시의 측정값입니다.

---

## 6. Deep Dive & Trade-offs
- MVI 도입: 초기 학습 곡선은 높았으나, 장기적인 유지보수성과 상태 관리의 안정성을 우선시함
- **순환 참조(Circular Dependency) 해결**:
  - 모듈 분리 중 발생한 상호 참조 문제를 해결하기 위해, 공통 로직을 `core:model`로 추상화하고 의존성 역전 원칙(DIP)을 적용하여 순환 참조 고리를 끊음
- **Trade-off**: 초기 설정 비용(Boilerplate)은 증가했으나, 향후 팀 규모 확대 시 발생할 코드 충돌과 빌드 시간 증가 문제를 선제적으로 방어