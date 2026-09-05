---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# 이헌준

**고려대학교 컴퓨터학과**

풀스택 웹부터 C/C++ 시스템 프로그래밍, 딥러닝 모델까지 한 층에만 머물지 않고
만들어 왔습니다. 만드는 것보다 **왜 그렇게 만들었는지 남기는 일**을 더 중요하게 생각해서,
설계 판단과 틀렸던 부분을 기록으로 남기는 습관을 들이고 있습니다.

| | |
|---|---|
| **Email** | [rex2837@korea.ac.kr](mailto:rex2837@korea.ac.kr) |
| **GitHub** | [github.com/bhj2837](https://github.com/bhj2837) |
| **Blog** | [bhj2837.github.io](https://bhj2837.github.io) |

---

## Skills

| 구분 | 내용 |
|---|---|
| **Language** | Java, Python, C, C++17, JavaScript |
| **Frontend** | Next.js 14 (App Router), Vue 3 (Composition API), TailwindCSS, Vite |
| **Backend** | Django 5 · Django REST Framework, Crow (C++) |
| **ML / NLP** | PyTorch, HuggingFace Transformers, LSTM, KoBART |
| **Infra / Tools** | GCP Compute Engine, Vercel, Railway, Docker, CMake, Git |

---

## Projects

### LearningPath

> 목표와 기간만 입력하면 주차별 학습 로드맵을 생성하는 AI 학습 플랫폼
{: .prompt-info }

| | |
|---|---|
| **기간** | 2026.02 – 2026.09 |
| **구성** | 개인 프로젝트 (기획 · 프론트엔드 · 백엔드 · 배포 전 과정) |
| **스택** | Next.js 14, Django 5, DRF, Claude API, PostgreSQL, TailwindCSS |
| **링크** | [저장소](https://github.com/bhj2837/learningpath) |

**배경** — 무엇을 공부할지는 정했는데 어떤 순서로 얼마나 해야 하는지 몰라서 시작을
못 미루는 경우가 많습니다. 목표·카테고리·기간·현재 수준을 입력하면 Claude API 가 주차별
커리큘럼, 추천 자료, 실습 프로젝트, 체크리스트를 설계하도록 만들었습니다.

**가장 오래 붙잡은 문제 — 권한**
AI 응답 품질보다 **소유권 검증**에 시간을 더 썼습니다. 로그인만 되어 있으면 URL 의 ID 만
바꿔서 남의 로드맵을 볼 수 있는 IDOR 이 가장 흔한 실수라고 판단했습니다.

- 모든 ViewSet 의 `get_queryset()` 을 요청자 기준으로 좁혀, 애초에 남의 레코드가
  쿼리셋에 들어오지 않게 했습니다.
- 그 결과 남의 리소스 ID 로 조회하면 `403` 이 아니라 **`404`** 가 나갑니다.
  403 은 "그 리소스는 존재한다"는 정보를 흘리기 때문입니다.
- 남의 로드맵을 FK 로 지정해 쓰기를 시도하면 `400`.
- 이 규칙이 나중에 깨지지 않도록 IDOR · 입력 검증 · 인증 · 저장 회귀를 포함해
  **테스트 34개**를 붙였습니다.

**그 외 다듬은 것**

- 로드맵 → 주차 → 체크리스트로 중첩 직렬화가 되면서 N+1 쿼리가 발생해
  `prefetch_related("weeks__checklists")` 로 묶었습니다.
- 로드맵 생성은 호출당 Claude API 비용이 나가므로 **시간당 10회** 스로틀,
  회원가입 · 로그인은 시간당 20회로 제한했습니다.
- 프론트(Vercel)와 백엔드(Railway)를 분리 배포하고 `/healthz/` 로 DB 연결까지
  확인하는 헬스체크를 뒀습니다.

---

### schedule-engine

> EDF 에 우선순위를 섞은 C++ 일정 최적화 서버. API 와 대시보드를 exe 하나로 배포
{: .prompt-info }

| | |
|---|---|
| **기간** | 2026.02 |
| **구성** | 개인 프로젝트 |
| **스택** | C++17, Crow, standalone Asio, CMake, MSVC |
| **링크** | [저장소](https://github.com/bhj2837/schedule-engine) |

**설계 판단** — EDF(Earliest Deadline First)는 마감일만 봅니다. 그래서 마감이 한참 남은
중요한 일이 계속 뒤로 밀립니다. 우선순위를 **시간 단위로 환산**해서 하나의 점수로 합쳤습니다.

```cpp
score = deadline - (priority * 3600);   // 낮을수록 먼저
```

우선순위 한 단계가 마감을 한 시간 당기는 효과를 냅니다. 덕분에 마감이 같으면 중요한 일이
앞으로 오고, 마감 차이가 크면 우선순위가 그것을 뒤집지 못합니다. 가중치를 곱셈이 아니라
**같은 차원(초)으로 변환해 뺄셈**으로 처리한 게 핵심이었습니다.

**구현** — Crow 로 REST API 를 세우고, 대시보드 HTML/JS 를 같은 바이너리에서 정적 서빙해
**빌드 결과 exe 하나만 옮기면 바로 동작**하도록 만들었습니다. 계산된 순서는 브라우저에서
간트 차트로 그리고, 마감을 넘기는 태스크는 자동 감지해 따로 표시합니다.

---

### mini-shell

> C 로 바닥부터 구현한 UNIX 셸
{: .prompt-info }

| | |
|---|---|
| **기간** | 2026.03 |
| **구성** | 개인 프로젝트 |
| **스택** | C, POSIX API, Make |
| **링크** | [저장소](https://github.com/bhj2837/mini-shell) |

파서 · 실행기 · 내장 명령어를 파일 단위로 나눠 직접 작성했습니다.

| 기능 | 예시 |
|---|---|
| 파이프라인 | `ls \| grep .c \| wc -l` |
| 리다이렉션 | `> file`, `>> file`, `< file` |
| 백그라운드 실행 | `sleep 5 &` |
| 내장 명령어 | `cd` (`cd -` 포함), `pwd`, `echo`, `export`, `exit` |
| 시그널 | `Ctrl+C` 가 셸이 아니라 실행 중인 명령만 종료 |

책으로 볼 때는 추상적이던 것들이 손으로 짜면서 구체화됐습니다. 파이프라인은 결국
**자식 프로세스마다 `dup2` 로 fd 를 갈아끼우는 일**이었고, `Ctrl+C` 가 셸까지 죽이지
않게 하려면 포그라운드 작업을 별도로 다뤄야 한다는 것을 실패하면서 알았습니다.

---

### daily-brief

> 뉴스 · 날씨 · 마켓을 한 화면에 모은 Vue 3 정보 포털
{: .prompt-info }

| | |
|---|---|
| **기간** | 2026.08 |
| **구성** | SK AX SKALA — Frontend Framework (Vue.js) 종합과제 |
| **스택** | Vue 3, Vue Router, Vite, ESLint · Prettier |
| **링크** | [데모](https://daily-brief-neon-three.vercel.app) · [저장소](https://github.com/bhj2837/daily-brief) |

에디토리얼(신문) 레이아웃에 라이트/다크 테마를 얹고, 뉴스 · 날씨 · 환율 · 코인 · 증시 ·
끝말잇기 게임 · 게시판까지 **15개 라우트**를 구성했습니다. 전 화면 Lazy Loading,
글쓰기 · 수정에는 `router.beforeEach` 인증 가드를 적용했습니다.

**신경 쓴 부분 — 심사자가 키 없이도 볼 수 있게**
외부 API 7종을 붙이면서, API 키가 없으면 앱이 깨지는 게 아니라 **Mock 데이터로 자동
폴백**하도록 만들었습니다. 키가 필요 없는 Open-Meteo · Frankfurter · CoinGecko ·
Hacker News 등은 배포 후에도 항상 실데이터로 동작하고, 증시(Finnhub) · 종합뉴스(GNews)는
키를 넣으면 자동으로 실시간으로 전환됩니다.

이전에 만든 `skala-weather` 를 버리지 않고 컴포넌트 단위로 떼어 `/weather` 모듈로
재사용했습니다.

---

### korean-dialogue-generation

> 화자를 구분하는 한국어 멀티턴 대화 생성 모델
{: .prompt-info }

| | |
|---|---|
| **기간** | 2024 |
| **구성** | 자연어처리 최종 프로젝트 |
| **스택** | PyTorch, HuggingFace Transformers, KoBART |
| **링크** | [저장소](https://github.com/bhj2837/korean-dialogue-generation) |

KoBART(`gogamza/kobart-base-v2`)를 AI Hub 한국어 SNS 대화 데이터로 파인튜닝했습니다.

**문제** — 멀티턴 대화를 한 줄로 이어 붙이면 모델이 어디까지가 누구의 발화인지 구분하지
못합니다. `<P01>`, `<P02>` 스페셜 토큰을 토크나이저에 추가하고 임베딩을 확장해
발화자를 명시하는 방식으로 context-to-response 구조를 만들었습니다.

**전처리** — SNS 데이터라 모델링만큼 정제가 중요했습니다. 초성 반복 정규화,
지나치게 짧은 대화 필터링 등을 거쳐 학습 데이터를 정리했습니다.

---

## Coursework

| 과목 | 프로젝트 |
|---|---|
| **COSE471** 데이터 과학 | [Sign-Language-Translation](https://github.com/bhj2837/Sign-Language-Translation) — LSTM 으로 수어 시퀀스를 학습하고, 웹캠 입력을 받아 **실시간으로 단어를 추론**하는 데모까지 구현 |
| **COSE474** 딥러닝 | [강의 실습 및 최종 프로젝트](https://github.com/bhj2837/20242R0136COSE47402) |
| **자연어처리** | 위 korean-dialogue-generation |

---

## Writing

이 블로그에는 **알고리즘 설계 기법**을 교재 순서대로 정리하고 있습니다.
문제 풀이의 답이 아니라, *왜 그 접근이 나오는지* 를 남기는 게 목적입니다.
답만 적어두면 한 달 뒤에 다시 못 푸는 걸 몇 번 겪고 나서 방식을 바꿨습니다.

- 완전 탐색 (Brute Force) · 완전 검색
- 감소 정복 (Decrease and Conquer) — 4편
- 분할 정복 (Divide and Conquer) — 2편
- 변환 정복 (Transform and Conquer) — 3편

[전체 글 보기 →](https://bhj2837.github.io)
