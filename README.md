# GPU Instance

OpenStack 환경에서 GPU 인스턴스를 생성하고, 호스트/인스턴스 단위의 GPU 모니터링을 구현한 프로젝트입니다.

## 배경

OpenStack에서 GPU 인스턴스를 생성하려면 GPU 할당 방식(Passthrough / MIG)과
OS 버전, OpenStack 버전에 따라 Flavor metadata 구성 방식이 달라집니다.
이를 수동으로 맞추는 과정에서 발생하는 실수를 줄이고,
다양한 GPU 호스트 상태에서 인스턴스를 일관되게 생성하기 위해 개발했습니다.

## 주요 기능

### GPU 인스턴스 생성

GPU 호스트 상태와 환경에 따라 적합한 Flavor metadata를 자동으로 구성합니다.

**지원 호스트 모드**
1. Passthrough + MIG mode disabled : GPU 전체를 VM에 직접 할당
2. Passthrough + MIG mode enabled : Passthrough 호스트에서 MIG 모드 활성화
3. MIG : MIG로 분할된 GPU 슬라이스를 VM에 할당

**Flavor metadata 구성 방식**
OS 버전 및 OpenStack 버전에 따라 아래 방식 중 적합한 방식을 자동 선택합니다.

1. Passthrough
- whiteList
- DeviceSpec

2. Mig
- mdev
- deviceSpec
   
## OS / OpenStack 버전별 지원 현황

**Ubuntu 22.04 + Yoga**
- Passthrough: WhiteList
- MIG: 사용 불가
  
**SUSE9 + Caracal**
- Passthrough: DeviceSpec
- MIG: mdev

**SUSE9 + Epoxy**
- Passthrough: DeviceSpec
- MIG: mdev

**Ubuntu 24.04 + Caracal**
- Passthrough: DeviceSpec
- MIG: 사용 불가

**Ubuntu 24.04 + Epoxy**
- Passthrough: DeviceSpec
- MIG: DeviceSpec

### 인스턴스 생성 가능 여부 사전 검증

인스턴스 생성 전 아래 항목을 자동으로 검증합니다.

- **nova.conf 설정 검증** — Nova API를 통해 각 컴퓨트 호스트의 nova.conf를 조회하고, GPU 인스턴스 생성에 필요한 설정이 올바르게 적용되어 있는지 확인
- **자원 가용성 검증** — Placement API를 활용해 해당 호스트에 요청한 GPU 자원이 남아있는지 판단 (검토 중)


### GPU 모니터링 (DCGM Exporter)
dcgm-exporter를 활용해 호스트와 인스턴스 양쪽에서 GPU 메트릭을 수집합니다.

- **호스트 단위 모니터링** — 물리 GPU 전체 메트릭 수집
- **인스턴스 단위 모니터링** — qemu-guest-agent를 통해 VM 내부 GPU 메트릭 수집

## 테스트 완료 환경
- NVIDIA A100
- NVIDIA H100
- NVIDIA B300

## 환경

- OS: Ubuntu 22.04
- 인프라: OpenStack
- 모니터링: DCGM Exporter



==============================================
# GPU Instance 기능 관련 개념 정리
- SR-IOV 개념 (PF / VF 구조)
- IOMMU / VFIO 개념
- Passthrough vs MIG 차이
- MIG 개념 및 GI/CI 구조
- mdev 동작 방식
- SR-IOV → IOMMU/VFIO → mdev → Nova PCI 관리 흐름

- nova.conf PCI 설정 (whitelist / device_spec) 차이
- GPU 자원이 Placement에 어떻게 등록되는지
- dcgm-exporter 동작 방식
- qemu-guest-agent 동작 방식 및 호스트-게스트 통신 구조







