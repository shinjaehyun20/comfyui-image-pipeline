# 📐 ARCHITECTURE.md

**Environment / Project / GPU Architecture – Source of Truth**

## 1. 문서 목적

이 문서는 다음을 **고정**한다.

- 프로젝트와 환경(env)의 관계
- GPU / CPU 운용 원칙
- “어디에 무엇을 두는가”에 대한 최종 기준

> 이 문서에 없는 방식은 **비표준**이다.

---

## 2. 핵심 설계 원칙 (요약)

1. **프로젝트는 env를 소유하지 않는다**
2. **env는 역할 단위로만 존재한다**
3. **GPU/CUDA는 프로젝트 밖에서만 관리한다**
4. **실행은 항상 절대경로 python**
5. **GPU는 기본 OFF, 필요 시 ON**

---

## 3. 디렉토리 아키텍처 (확정)

```text
D:\
├─ env
│  ├─ python
│  │  ├─ py311-llm      # agent / LLM / API
│  │  ├─ py311-web      # web / UI
│  │  └─ py310-torch    # torch / GPU
│  └─ meta
│     ├─ env-map.json
│     └─ requirements
│
├─ projects
│  ├─ active
│  ├─ maintained
│  ├─ sandbox
│  └─ archive
│
├─ agents
│  └─ cline
│
└─ docs
   ├─ ARCHITECTURE.md
   └─ RUNBOOK.md
```

---

## 4. env 종류와 책임

### py311-llm

- agent / LLM / automation / API
- GPU ❌
- 예: agent-builder, pm-master-platform

### py311-web

- 웹 / UI / 게임
- GPU ❌
- 예: kids-game-collection

### py310-torch

- torch / 음성 / 영상 / GPU
- GPU ⭕
- 예: simswap, voice-mod-app

---

## 5. env 선택 메커니즘

### env-map.json (중앙 선언)

```json
{
  "simswap": {
    "python": "py310-torch",
    "cuda": "12.1"
  }
}
```

### 프로젝트는 `.env`로 참조만 한다

```env
PY_ENV=py310-torch
CUDA_VERSION=12.1
```

---

## 6. 실행 규칙 (절대)

```powershell
D:\env\python\py310-torch\Scripts\python.exe main.py
```

- venv activate ❌
- python 명령 ❌
- 상대경로 ❌

---

## 7. GPU / CPU 운용 정책

- 기본: CPU
- GPU는 실험/단발 용도
- 장시간/배치는 CPU 유지

```env
CUDA_VISIBLE_DEVICES=      # CPU
CUDA_VISIBLE_DEVICES=0     # GPU
```

조치(ComfyUI PoC 권고): 에이전트-툴 연동 관점에서, 에이전트는 로컬 이미지 생성 엔진(예: ComfyUI)을 "파일 생성 워커"로 간주해야 합니다. 즉, 에이전트는 프롬프트를 컴파일하여 ComfyUI용 workflow JSON을 생성하고 enqueue(트리거)만 수행하며, ComfyUI가 `output/`에 생성한 파일을 수집(Collector)이 탐지해 가져오는 흐름을 표준으로 삼습니다. 로컬 모델이 준비된 경우 에이전트는 Hugging Face 토큰을 자동으로 조회하거나 요구하지 않도록 설계해야 합니다.

---

## 8. 금지 사항

- 프로젝트별 torch 설치
- CUDA 재설정
- requirements.txt 프로젝트 단위 관리
- 코드 내 CUDA 하드코딩

---

## 9. 관련 문서

- [docs/RUNBOOK.md](docs/RUNBOOK.md)
- [docs/ONBOARDING.md](docs/ONBOARDING.md)
- [docs/GPU_UPDATE_PLAN.md](docs/GPU_UPDATE_PLAN.md)
