## Gradio PoC — Doc‑Writer → Developer 핸드오프 가이드

작성 목적

- Doc‑writer가 개발자로부터 받은 변경 사항(Gradio PoC)을 문서화하고, 개발자가 제공한 산출물을 기반으로 사용자 문서, README 스니펫, API 예제, 스크린샷 캡션을 빠르게 작성할 수 있도록 정확한 기술 사양과 검증 절차를 제공합니다.

요약 (한눈에)

- 추가된 기능: 로컬 SDXL용 Gradio PoC 런처
  - 최대 10행 입력(기본 3행 표시)
  - 상단 전역 컨트롤: Width, Height, Steps, Guidance
  - 전역 `Negative prompt` 텍스트박스 (모든 행에 동일 적용)
  - 각 행: `Title`, `Prompt`, 우측 `Thumbnail` + `Status`
  - 로컬 HTTP API: `POST /start`, `GET /status`, `POST /cancel`, `POST /shutdown` (127.0.0.1:5001)
  - 출력 폴더: `d:\tools\ComfyUI\output`
  - 아카이브: 중복 배치파일(`start_gradio.bat`, `stop_gradio.bat`, `restart_gradio.bat`, `start_comfy_simple.bat`, `start_comfy_runtime_error.txt`) → `d:\tools\ComfyUI\archive/`

변경된 파일(주요)

- `d:\tools\ComfyUI\gradio_app.py` — UI, API, job 로직, global negative prompt 전달 추가
- `d:\tools\ComfyUI\start_comfy.bat` — 새 시작 스크립트 (절대경로 Python 사용)
- `d:\tools\ComfyUI\README.md` — Archived files 섹션 추가
- `d:\tools\ComfyUI\archive/*` — 백업된 이전 배치 파일

UI 상세(개발자가 전달한 레이블/동작)

- 상단 라인:
  - `Width (px)` (Number, 기본 1200)
  - `Height (px)` (Number, 기본 800)
  - `Steps` (Slider, 1–200, 기본 20)
  - `Guidance` (Slider, 0.1–30.0, 기본 7.5)
  - `Negative prompt (global)` (Textbox, 2줄, 모든 행에 동일 적용)
  - `Cancel All` 버튼, `Generate All (queue)` 버튼
- 각 행 (최대 10):
  - `Title N` (선택)
  - `Prompt N` (텍스트박스, 3줄)
  - 오른쪽: `Thumbnail N` (이미지), `Status`(읽기전용)
  - 기본 표시 행 수: 3 (Add Row / Remove Row로 조절)

API 스펙 (핵심, doc에 그대로 사용 가능)

- POST /start
  - 요청 JSON 예:

    ```json
    {
      "item": {
        "title": "Sample",
        "prompt": "A serene mountain lake at sunrise, photorealistic",
        "negative_prompt": "lowres, blurry, watermark",
        "steps": 30,
        "guidance": 7.5,
        "width": 1200,
        "height": 800
      }
    }
    ```

  - 응답: `{ "job_id": "<UUID>" }`

- GET /status?job_id=<JOB_ID>
  - 응답 예: `{ "job_id":"...", "status":"DONE|RUNNING|QUEUED|ERROR|CANCELLED", "progress": 42, "path":"<path>","thumbnail":"data:image/png;base64,..." }`

- POST /cancel
  - 요청: `{ "job_id": "<JOB_ID>" }` → 큐된 작업은 `CANCELLED`, 실행중이면 `CANCELLING`으로 표기

- POST /shutdown
  - 안전한 종료 신호, 핸들러는 응답 후 백그라운드에서 프로세스 종료 수행

검증 / 재현 단계 (개발자가 문서화할 때 포함)

1. 서버 시작
   - 더블클릭: `d:\tools\ComfyUI\start_comfy.bat` 또는 CLI에서

     ```powershell
     D:\env\python\py310-torch\Scripts\python.exe d:\tools\ComfyUI\gradio_app.py
     ```

2. UI 테스트
   - 브라우저 접속: `http://localhost:7860`
   - 상단 `Negative prompt (global)`에 예시 입력
   - 한두 행에 Prompt 입력 → `Generate All (queue)` 클릭
   - `d:\tools\ComfyUI\output`에 생성된 파일 확인, UI 썸네일 확인
3. API 테스트
   - 위 `/start` JSON 으로 `curl` 또는 Postman 호출 → 반환된 `job_id`로 `/status` 확인
4. 종료 테스트
   - UI의 `Stop Server` 버튼 클릭 → Gradio와 Python 프로세스가 종료되는지 검증

스크린샷 요청 목록 (doc에 삽입할 이미지)

1. 전체 UI 캡처(상단 컨트롤 + 몇 개 행이 보이는 상태)
2. 상단 컨트롤 영역 확대(negative prompt 포함)
3. 행 하나에 prompt 입력 후 큐에 추가된 상태(썸네일/상태 포함)
4. 출력 폴더(`d:\tools\ComfyUI\output`)에서 생성된 이미지 파일과 워크플로우 정보(파일명 포맷 예시)
5. README에 추가된 `Archived files` 섹션 화면(선택)

PR/커밋 메시지 템플릿 (제안)

- 제목: `feat: add Gradio PoC launcher + global negative prompt`
- 본문: 간단 변경 사항 요약(위 요약 포함), 테스트 커맨드, 아카이브된 파일 목록, 스크린샷 위치

문서화 체크리스트 (doc-writer용)

- [ ] UI 캡션 4개 촬영 및 업로드
- [ ] README 스니펫(사용법) 삽입 및 링크 확인
- [ ] API 예제(JSON) 포함
- [ ] 변경 로그(아카이브 내역 포함) 추가
- [ ] PR 본문에 스크린샷과 검증 커맨드 포함

주의 / 알려둘 점

- 현재 구현은 `global negative prompt`만 지원합니다. 향후 각 행별 negative prompt가 필요하면 `gradio_app.py`의 UI에 각 행별 입력 필드를 추가하고 `create_job` 호출에 전달하면 됩니다.
- 파이프라인 호출은 단일 `StableDiffusionXLPipeline` 인스턴스에 대해 `pipeline_lock`로 직렬화되어 실행됩니다. 대규모 병렬 요청은 성능 저하나 OOM을 유발할 수 있습니다 — 문서에 경고 문구 추가 권장.

문의처(개발자 정보)

- 변경 작업자: 작업 내역에 포함된 커밋/패치 확인
- 위치: `d:\tools\ComfyUI\gradio_app.py`, `d:\tools\ComfyUI\start_comfy.bat`, `d:\tools\ComfyUI\README.md`

---
이 문서를 바탕으로 README 문단, 사용자 가이드, API 문서 초안을 생성하시면 됩니다. 원하시면 제가 README용 완성 문단(마크다운)과 API 섹션을 별도 파일로 생성해 드리겠습니다.
