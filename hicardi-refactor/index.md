---
layout: project
title: "Hicardi Android 앱 레거시 리팩토링 및 아키텍처 고도화"
subtitle: "레거시 Java 기반 Android 앱을 Kotlin · MVI/Clean Architecture · 멀티모듈로 재설계하고, build-logic/CI·CD로 빌드·배포 시간을 정량 개선한 사례"
period: "2025.09 – 2025.12"
role: "Android Developer"
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
초기 스타트업 단계에서 빠르게 개발된 단일 모듈(Monolithic) 구조로, 비즈니스 로직과 UI 코드가 뒤섞여 있어 유지보수가 한계에 다다른 상황이었습니다.  
테스트 코드 부재로 인해 수정 시마다 전체 수동 테스트가 강제되었고, **APK 추출부터 사내 배포 게시판 업로드까지 이어지는 반복적인 수동 배포 과정**이 개발 생산성을 심각하게 저해하고 있었습니다.

---

## 2. Background & Problem
- **스파게티 코드 및 구조 부재**: 단일 모듈 내에 모든 파일이 혼재되어 있고, 패키지 구조가 정리되지 않아(A 기능 코드가 B 패키지에 존재 등) 유지보수 난이도 극심
- **테스트 환경 전무**: 자동화된 테스트 코드가 없어, 작은 변경에도 앱 전체를 수동으로 검증해야 하는 리스크 존재
- **비효율적인 수동 배포**: APK 추출 후 사내 배포 사이트에 접속해 정보를 입력하고 파일을 첨부하는 반복 작업으로 개발 리소스 낭비

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
  - **How**: Google의 'Now in Android' 아키텍처를 참고하여 `app` → `feature` → `core`의 3단계 계층 구조로 재편. 기능 단위는 `feature` 모듈로, 공통 비즈니스 로직과 데이터 모델은 `core` 모듈로 분리하여 응집도를 높임
  - **Why**: 기능별 격리를 통해 코드 결합도를 낮추고, 변경된 모듈만 빌드하는 증분 빌드(Incremental Build) 환경을 구축하여 생산성 극대화

<div class="mermaid">
graph TD
    app[":app"]
    feature[":feature"]
    core[":core"]

    app --> feature
    feature --> core
    
    style app fill:#dbeafe,stroke:#2563eb,stroke-width:2px
    style feature fill:#d1fae5,stroke:#059669,stroke-width:2px
    style core fill:#f3f4f6,stroke:#4b5563,stroke-width:2px
</div>

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
- 리팩토링 후 기능 추가 시 변경 영향 범위를 명확히 예측 가능하도록 구조 개선 및 배포 자동화 달성

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
  - 기능(Feature) 모듈 간에 서로를 참조하는 순환 의존성 문제를 해결하기 위해, 두 모듈이 공통으로 사용하는 데이터 모델을 하위 계층인 `core:model` 모듈로 **추출(Extract)**하여 의존성 방향을 단방향(`feature` → `core`)으로 정리함
- **Trade-off**: 초기 설정 비용(Boilerplate)은 증가했으나, 향후 팀 규모 확대 시 발생할 코드 충돌과 빌드 시간 증가 문제를 선제적으로 방어