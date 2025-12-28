# GPU / Python Environment RUNBOOK

## 목적

- 프로젝트별 환경 중복 제거
- GPU/CPU 전환 표준화
- 재설정 없는 장기 운용

---

## 환경 구조

D:\env
├─ python
│  ├─ py311-llm      # agent / LLM
│  ├─ py311-web      # web / ui
│  └─ py310-torch    # GPU / torch
└─ meta
   └─ env-map.json

---

## 환경 선택 방식

모든 프로젝트는 `.env`로 환경을 선택한다.

```env
PY_ENV=py310-torch
CUDA_VERSION=12.1
```

---

## 실행 규칙 (절대 원칙)

- venv activate ❌
- 항상 절대경로 python 사용 ⭕
- 예:

```powershell
D:\env\python\py310-torch\Scripts\python.exe main.py
```

운영 권고(ComfyUI PoC): 에이전트-툴 실행 관점에서 모든 실행은 절대경로 인터프리터로 수행해야 하며(예: `D:\env\python\py310-torch\Scripts\python.exe`), 에이전트는 로컬 모델 보유 환경에서는 Hugging Face 토큰을 자동 조회하지 않도록 구성해야 합니다. GPU 활성화는 수동 환경변수(`CUDA_VISIBLE_DEVICES`)로 제어하며, 운영팀 승인이 필요합니다.

---

## GPU / CPU 전환

### CPU (기본)

```env
CUDA_VISIBLE_DEVICES=
```

### GPU (실험)

```env
CUDA_VISIBLE_DEVICES=0
```

또는 PowerShell:

```powershell
$env:CUDA_VISIBLE_DEVICES=""
$env:CUDA_VISIBLE_DEVICES=0
```

---

## GPU 주의사항

- RTX 5070 Ti (cc 12.x)
- 현재 PyTorch 안정판 공식 지원 ❌
- CUDA 인식은 되나 커널 경고 발생 가능
- 운영 기본값은 CPU

---

## 프로젝트별 실행 예시

### simswap

```powershell
D:\env\python\py310-torch\Scripts\python.exe main.py
```

### voice-mod-app

```powershell
D:\env\python\py310-torch\Scripts\python.exe app.py
```

---

## 금지 사항

- 프로젝트별 torch 설치
- CUDA 재설정
- requirements 중복 관리

---

## 업데이트 기준

- PyTorch 릴리스 노트에 `sm_120` 명시 시
- torch / torchvision / torchaudio 업그레이드

```powershell
# 예시 업그레이드 (cu121)
pip install --upgrade torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

---

## 참고 문서

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [docs/ONBOARDING.md](docs/ONBOARDING.md)
- [docs/GPU_UPDATE_PLAN.md](docs/GPU_UPDATE_PLAN.md)
