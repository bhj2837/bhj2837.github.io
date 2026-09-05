---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# 이헌준 (Heonjun Lee)

고려대학교 컴퓨터학과 재학 중입니다.
웹 서비스부터 시스템 프로그래밍까지, 만들어보고 싶은 건 일단 만들어보는 편입니다.

- **Email** — [rex2837@korea.ac.kr](mailto:rex2837@korea.ac.kr)
- **GitHub** — [github.com/bhj2837](https://github.com/bhj2837)
- **Blog** — 이 사이트. 알고리즘 문제 풀이 정리를 올립니다.

---

## 기술 스택

| 구분 | 사용 경험 |
|------|-----------|
| Language | Java, Python, C/C++, JavaScript |
| Frontend | Next.js 14, Vue 3, TailwindCSS |
| Backend | Django 5, Crow (C++), Paper/Bukkit API |
| ML/NLP | PyTorch, HuggingFace Transformers (KoBART) |
| Infra | GCP Compute Engine, Vercel, Railway, Git |

---

## 프로젝트

### LearningPath — AI 학습 로드맵 생성 플랫폼
[저장소 →](https://github.com/bhj2837/learningpath)

목표와 기간을 입력하면 Claude API가 주차별 커리큘럼, 추천 자료, 실습 프로젝트를 자동
생성합니다. Next.js 14 프론트엔드와 Django 5 REST 백엔드를 분리해 각각 Vercel / Railway 에
배포했습니다.

`Next.js 14` `Django 5` `Claude API` `TailwindCSS`

### SharedFate — 협동 마인크래프트 서버 플러그인
[저장소 →](https://github.com/bhj2837/sharedfate)

여러 플레이어가 데미지·인벤토리·죽음 카운트를 **공유**하는 Paper 1.21.x 서버 플러그인.
직접 GCP 에 서버를 띄워 실제로 지인들과 플레이하며 밸런스를 조정했습니다.
엔더드래곤 처치 후 통계 기반 시상식 콘텐츠까지 구현했습니다.

`Java 21` `Paper API` `Gradle` `GCP`

### schedule-engine — C++ 작업 스케줄링 최적화 엔진
[저장소 →](https://github.com/bhj2837/schedule-engine)

EDF(Earliest Deadline First)와 우선순위 가중치를 결합한 스케줄러를 C++17 로 구현하고,
Crow 프레임워크로 REST API 와 간트 차트 대시보드를 **단일 실행 파일**에서 함께 서빙합니다.

`C++17` `Crow` `REST API`

### mini-shell — UNIX 셸 구현
[저장소 →](https://github.com/bhj2837/mini-shell)

C 로 작성한 최소 기능 셸. 파이프라인, 입출력·추가 리다이렉션, 시그널 처리, 내장 명령어를
직접 구현하며 `fork`/`execve`/`dup2` 와 프로세스 그룹 동작을 익혔습니다.

`C` `POSIX` `System Programming`

### korean-dialogue-generation — 한국어 대화 생성 모델
[저장소 →](https://github.com/bhj2837/korean-dialogue-generation)

KoBART(`gogamza/kobart-base-v2`)를 파인튜닝한 한국어 멀티턴 대화 생성 모델.
`<P01>`, `<P02>` 화자 토큰을 추가해 발화 주체를 구분하도록 했습니다. (자연어처리 최종 프로젝트)

`PyTorch` `HuggingFace` `KoBART`

### daily-brief — 개인 대시보드
[데모 →](https://daily-brief-neon-three.vercel.app) · [저장소 →](https://github.com/bhj2837/daily-brief)

날씨·일정·뉴스를 한 화면에 모은 Vue 3 대시보드. 이전에 만든 weather 모듈을 컴포넌트로
재사용해 `/weather` 라우트로 붙였습니다. (SK AX SKALA Vue.js 종합과제)

`Vue 3` `Vercel`

---

## 학교 프로젝트

- **Sign-Language-Translation** — 수어 영상 분류 (COSE471 데이터 과학)
- **COSE474 딥러닝** — 강의 과제 및 실습 노트북

---

## 이 블로그

알고리즘 문제를 풀고 나서 *왜 그 풀이가 되는지*를 정리해 두는 공간입니다.
답만 적어두면 한 달 뒤에 다시 못 푸는 걸 여러 번 겪어서, 접근 과정을 남기기로 했습니다.
