# 🧠 GPU 공식 지원 업데이트 플랜

## 현재 상태

- GPU: RTX 5070 Ti (cc 12.x)
- PyTorch 안정판: 공식 지원 ❌
- cu121 빌드: 동작하나 경고 있음

---

## 단기 전략 (현재)

- CPU 기본
- GPU는 실험용
- py310-torch 유지

PoC 검증 메모: 로컬 SDXL PoC에서 RTX 5070 Ti 환경(cu128 호환)으로 성능 개선을 확인했습니다. 전역 권고에는 검증된 조합(`py310-torch` + `cu128` 호환 torch 빌드)을 `verified` 메모로 추가하고, 문서에 명시된 특정 마이너 버전 대신 `12.x`와 같은 범용 표기를 사용해 혼선을 줄이세요.

---

## 중기 전략 (Nightly)

- Nightly 릴리스 노트 확인
- `sm_120` 포함 시 테스트

```powershell
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu121
```

---

## 장기 전략 (정식 지원)

- 릴리스 노트에 `cc 12.x / sm_120` 명시 시
- 안정판 업그레이드
- RUNBOOK / ARCHITECTURE 갱신

---

## 업데이트 체크리스트

- [ ] torch.cuda.is_available()
- [ ] 경고 제거 여부
- [ ] simswap 스모크 테스트
- [ ] voice-mod-app 스모크 테스트

---

## 한 줄 기준

> **GPU는 “지원되면 올리고, 그 전까지는 참는다.”**

---

## 참조 문서

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [docs/RUNBOOK.md](docs/RUNBOOK.md)
