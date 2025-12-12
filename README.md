# Super Amazing Project (SAP) 🛰️  
_Base Infra & GPU Cluster for Cloud-Native Research_

> **SAP**는 Honam Univ 온프렘 환경에서 **Kubernetes + GPU + Observability**를 중심으로  
> ROS2, Quantum Simulation 등 여러 연구 프로젝트를 올리기 위한 공통 인프라 프로젝트입니다.

---

## 0. Links

- 📌 **To Do List**: https://www.notion.so/To-Do-List-2b285d6a017e80ea891bee4ef09e37ef?pvs=21
- 🌱 **Extended Projects**
  - [`SAP-ROS2: Cloud-Native Robotics on Kubernetes`](https://www.notion.so/SAP-ROS2-Cloud-Native-Robotics-on-Kubernetes-2b185d6a017e80ae9044edc482ad41e5?pvs=21)
  - [`SAP-Qubit: V100-based Quantum Simulation Lab`](https://www.notion.so/SAP-Qubit-V100-based-Quantum-Simulation-Lab-2b185d6a017e805ba7eefa0353d92c65?pvs=21)

---

## 1. Overview

### 1-1. 목표

- V100 GPU를 포함한 **온프렘 Kubernetes 클러스터 구축**
- **Observability 스택**: Grafana + Prometheus + Node Exporter + NVIDIA DCGM Exporter
- 학교 네트워크 제약(외부 네트워크 / VLAN 제약)으로 인해
  - **Ceph 미사용**
  - 단일 노드 또는 단순 스토리지(Local NVMe / NFS / hostPath) 기반 설계

### 1-2. Scope

- 공통 베이스 인프라:
  - OS 설치 및 공통 유틸
  - K8s 클러스터 구성(Control / Worker / GPU 노드)
  - Observability / Alerting 기본 구성
- 위 인프라 위에 올라갈 프로젝트:
  - **SAP-ROS2**: Cloud-native Robotics + SLAM/딥러닝
  - **SAP-Qubit**: V100 기반 양자 회로 시뮬레이션 및 벤치마크

---

## 2. Architecture (공통)

- **OS**: Ubuntu 24.04.2 LTS (Noble Numbat)
- **Control Plane**: x 노드 (CPU Only)  
- **Worker**: x 노드 (CPU / 범용 워커)
- **GPU Node**: V100 장착 노드 (worker-4, worker-5)
- **Storage**:
  - 로컬 NVMe (etcd / 중요 데이터용)
  - HDD / SSD (워크로드 / 로그)
  - NFS / hostPath 기반 간단 스토리지
- **Observability**:
  - Prometheus
  - Grafana
  - Node Exporter
  - nvidia-dcgm-exporter (GPU 메트릭용)
- **형상 관리**:
  - GitHub repo: `sap-infra` (이 README가 위치한 레포)

> 상세 네트워크/토폴로지는 추후 `docs/architecture.md`로 분리 예정.

---

## 3. Node Inventory & Performance Baseline

### 3-1. Physical Node Spec

> Role 컬럼은 실제 구성에 맞게 `[TODO]`로 채워 넣을 예정입니다.

| Node ID   | Model / Type        | CPU (Model, Cores)                  | RAM           | Storage (OS / Data)              | GPU         | Network       | Role           |
|----------|----------------------|-------------------------------------|---------------|----------------------------------|------------|---------------|----------------|
| master-1 | PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 16GB     | 500GB HDD                        | None       | 172.30.0.221  | [TODO]         |
| master-2 | PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 16GB     | 500GB HDD                        | None       | 172.30.0.222  | [TODO]         |
| master-3 | PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 16GB     | 500GB HDD                        | None       | 172.30.0.223  | [TODO]         |
| worker-1 | PowerEdge R630       | Intel Xeon E5-2637 v4 (16 cores)    | DDR4 16GB * 2 | 2TB HDD * 2 (RAID 0)             | G200eR2    | 172.30.0.224  | [TODO]         |
| worker-2 | HP Z-820             | Intel Xeon E5-2680 (32 cores)       | DDR3 4GB * 16 | 2TB, 500GB, 250GB HDD * 1        | GT 625 OEM | 172.30.0.225  | [TODO]         |
| worker-3 | PowerEdge R440       | Intel Xeon Silver 4210 (20 cores)   | DDR4 16GB * 2 | 480GB SSD * 2 (RAID 0)           | G200eW3    | 172.30.0.226  | [TODO]         |
| worker-4 | PowerEdge R740       | Intel Xeon Gold 6150 (72 cores)     | DDR4 32GB * 4 | 4TB HDD                          | V100 * 2   | 172.30.0.227  | GPU Worker     |
| worker-5 | PowerEdge R740       | Intel Xeon Gold 6150 (72 cores)     | DDR4 32GB * 4 | 4TB HDD                          | V100 * 2   | 172.30.0.228  | GPU Worker     |
| control-1| PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 8GB      | 500GB HDD + 1TB NVMe SSD         | None       | 172.30.0.231  | Control Plane? |
| control-2| PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 8GB      | 500GB HDD + 1TB NVMe SSD         | None       | 172.30.0.232  | Control Plane? |
| control-3| PowerEdge T110 II    | Intel Xeon E3-1220 V2 (4 cores)     | DDR3 8GB      | 500GB HDD + 1TB NVMe SSD         | None       | 172.30.0.233  | Control Plane? |

> ⚠️ 실제 K8s Control Plane / Worker / Storage 역할은 운영 구조 확정 후 업데이트 예정.

---

### 3-2. GPU Node Detail (V100)

**GPU Nodes**: `worker-4`, `worker-5`

- **GPU**: NVIDIA Tesla V100 16GB × 2 (per node)
- **주요 활용**
  - `SAP-Qubit`
    - 양자 회로 시뮬레이션
    - 대규모 행렬 연산, 벤치마크
  - `SAP-ROS2`
    - 딥러닝 / 강화학습 로직 학습 및 추론
- **대표 스펙 (데이터시트 기준)**
  - FP32 연산 성능: ~14 TFLOPS
  - 메모리 대역폭: ~900 GB/s
  - GPU 메모리: 16GB
- **모니터링 항목**
  - GPU Utilization (%)
  - Memory Used (GB)
  - Power Draw (W)
  - Temperature (°C)
  - (옵션) ECC Error Count

---

### 3-3. Baseline Benchmark Plan

> 실제 측정 결과는 실험 후 테이블에 채워 넣고, `evidence/` 폴더에 로그/스크린샷 보관 예정.

#### 측정 도구 제안

- **CPU**
  - `sysbench`
  - `stress-ng`
- **Disk**
  - `fio` (NVMe / HDD read/write IOPS, MB/s)
- **Network**
  - `iperf3` (노드 간 대역폭, RTT)
- **GPU**
  - `nvidia-smi`
  - (옵션) `dcgmi`
  - 자체 Python/NumPy/CuPy 벤치마크 스크립트

#### CPU Benchmarks

| Node ID  | Tool / Test   | Threads | Score / Time | 메모      |
|----------|---------------|---------|--------------|-----------|
| control-1| sysbench cpu  | [TODO]  | [TODO]       | CP        |
| worker-1 | sysbench cpu  | [TODO]  | [TODO]       |           |
| worker-2 | sysbench cpu  | [TODO]  | [TODO]       |           |
| worker-3 | sysbench cpu  | [TODO]  | [TODO]       |           |
| worker-4 | sysbench cpu  | [TODO]  | [TODO]       | GPU Node  |
| worker-5 | sysbench cpu  | [TODO]  | [TODO]       | GPU Node  |

#### Disk Benchmarks

| Node ID   | Device      | Test (read/write) | IOPS / MB/s | 메모                 |
|-----------|-------------|-------------------|-------------|----------------------|
| worker-3  | 480GB SSD   | [TODO]            | [TODO]      | K8s etcd/OS 후보    |
| storage-1 | 1TB NVMe    | [TODO]            | [TODO]      | 데이터/로그         |
| storage-2 | 1TB NVMe    | [TODO]            | [TODO]      |                      |
| storage-3 | 1TB NVMe    | [TODO]            | [TODO]      |                      |

#### Network Benchmarks

| From → To             | Tool   | Bandwidth (Gbps) | RTT(ms) | 메모              |
|-----------------------|--------|------------------|---------|-------------------|
| control-1 → worker-4  | iperf3 | [TODO]           | [TODO]  | Control ↔ GPU     |
| control-1 → worker-5  | iperf3 | [TODO]           | [TODO]  | Control ↔ GPU     |
| worker-3 → worker-4   | iperf3 | [TODO]           | [TODO]  | Worker ↔ GPU      |
| worker-3 → storage-1  | iperf3 | [TODO]           | [TODO]  | Worker ↔ Storage  |

#### GPU Benchmarks (V100)

| Node ID  | Test Name           | Size / Shots | CPU Time(s) | GPU Time(s) | Speedup(x) |
|----------|---------------------|--------------|-------------|-------------|-----------|
| worker-4 | Matrix Mul 4096×4096| [TODO]       | [TODO]      | [TODO]      | [TODO]    |
| worker-4 | Bell State 10k shots| [TODO]       | [TODO]      | [TODO]      | [TODO]    |
| worker-5 | Grover (N = …)      | [TODO]       | [TODO]      | [TODO]      | [TODO]    |

---

## 4. 운영 / 관리 도구

### 4-1. OS / 시스템 레벨

- SSH, `tmux`
- `htop`, `nvtop`
- 기본 유틸: `curl`, `wget`, `git`, `vim`, `jq` 등

### 4-2. Kubernetes 관리

- `kubectl`
- `k9s`
- (옵션) Helm, kustomize

### 4-3. Observability

- Grafana
- Prometheus
- Alertmanager (옵션, 추후 알림 연계)
- Node Exporter
- NVIDIA DCGM Exporter

### 4-4. 형상 관리

- Git / GitHub
  - Repository: `sap-infra`
  - 향후 구성:
    - `cluster/` : K8s 매니페스트 / Helm values
    - `ansible/` : 초기 배포 스크립트
    - `benchmarks/` : 벤치마크 스크립트
    - `docs/` : 아키텍처 / 운영 문서

### 4-5. 자동화 / 워크플로우 (n8n)

- **n8n (self-hosted) 활용 계획**
  - Prometheus Alert → n8n → Discord / Slack / Telegram 알림
  - 실험 결과(JSON/CSV) → n8n → Google Sheets / Notion 자동 기록
  - 주기적 헬스체크
    - 노드 상태 점검
    - GPU 온도 / 전력 사용량 체크
    - 결과를 요약 리포트로 Slack/메일 발송

---

## 5. Infra Checklist (전체)

- [x] 모든 노드 BIOS / RAID / 펌웨어 점검
- [x] OS 설치 및 공통 유틸 설치
- [x] Kubernetes 클러스터 구성 (컨트롤 / 워커 분리)
- [x] GPU Node 드라이버 + CUDA 설치
- [x] NVIDIA Device Plugin / GPU Operator 설정
- [ ] Prometheus + Grafana 설치 및 노드 메트릭 수집
- [ ] Baseline Benchmark 실행 (CPU / Disk / Net / GPU)
- [ ] 결과를 위 테이블에 기록 & 스크린샷 저장
- [ ] GitHub `sap-infra`에 매니페스트 + 스크립트 정리
- [ ] `SAP-ROS2`, `SAP-Qubit`과 연계 테스트

---

## 6. 참고 / 메모

- 각종 Ubuntu 관련 명령어 정리
  - https://www.notion.so/Ubuntu-2b185d6a017e80089886eaa684a07684?pvs=21
- 관련 기술/자료 모음
  - https://www.notion.so/2b285d6a017e80538123d951aa48f9b9?pvs=21

---
