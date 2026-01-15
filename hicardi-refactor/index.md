---
layout: project
title: "Hicardi Refactor"
subtitle: "레거시 Java → Kotlin, MVI/Clean, 멀티모듈 + CI/CD로 빌드·배포 효율을 정량 개선한 사례"
period: "2025.09 – 2025.12"
role: "Android Developer (Refactor / Build & CI/CD Owner)"
featured: true
tags:
  - Android
  - Kotlin
  - MVI
  - Clean Architecture
  - Multi-module
  - build-logic
  - GitHub Actions
metrics:
  - { value: "78%", label: "빌드 시간 단축" }
  - { value: "66%", label: "배포 소요 시간 감소" }
  - { value: "60%", label: "테스트 커버리지" }
---

## Overview
레거시 구조가 커지면서 변경 영향도가 커지고, 빌드/배포 시간이 늘어나 생산성이 떨어지는 문제가 있었습니다. 구조 개편과 빌드 시스템 정비를 동시에 진행해 “속도”와 “유지보수성”을 함께 끌어올렸습니다.

## Problem
- 레거시 구조로 변경 영향도가 큼 (수정 → 사이드이펙트 증가)
- 빌드/배포 시간이 길어 개발 리드타임이 증가
- 테스트 기반이 약해 리팩토링 리스크가 큼

## Solution
- MVI + Clean Architecture로 상태/이벤트/도메인 경계를 명확히 분리
- 멀티모듈 분리로 의존성 경계와 코드 소유권을 명확히 함
- build-logic로 Gradle 설정을 단일화해 중복 제거 및 빌드 성능 최적화
- GitHub Actions 기반 CI/CD로 빌드/테스트/배포를 자동화

## Result
- 빌드 시간 78% 단축
- 배포 소요 시간 66% 감소
- 테스트 커버리지 60% 달성

## Key Decisions / Trade-offs
- MVI 도입: 초기 학습 비용은 증가했지만, 복잡한 상태를 안정적으로 제어
- 멀티모듈: 설정 복잡도는 늘었지만, 장기적으로 확장성과 빌드 효율 확보

## Interview-ready
- MVI를 선택한 이유와 MVVM 대비 운영에서의 장점은?
- 멀티모듈 분리 기준과 순환 의존성 방지 전략은?
- build-logic 도입 시 발생한 충돌(플러그인/버전) 해결 방식은?
- 빌드 78% 개선을 어떤 방식으로 측정/검증했나?
