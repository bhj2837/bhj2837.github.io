---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

## 이헌준

고려대학교 컴퓨터학과.
쓸모가 있든 없든 궁금한 건 일단 끝까지 만들어보는 편입니다.
그 과정에서 틀렸던 것들을 여기에 정리합니다.

[rex2837@korea.ac.kr](mailto:rex2837@korea.ac.kr) · [github.com/bhj2837](https://github.com/bhj2837)

## Tech

| | |
|---|---|
| **Language** | Java, Python, C/C++, JavaScript |
| **Frontend** | Next.js 14 (App Router), Vue 3, TailwindCSS |
| **Backend** | Django 5 / DRF, Crow (C++), Paper API |
| **ML/NLP** | PyTorch, HuggingFace Transformers |
| **Infra** | GCP Compute Engine, Vercel, Railway |

## Projects

### LearningPath — AI 학습 로드맵 생성 플랫폼

`Next.js 14` `Django 5` `DRF` `Claude API`

목표·기간·현재 수준을 입력하면 주차별 커리큘럼과 추천 자료, 체크리스트를 생성합니다.

만들면서 제일 오래 붙잡았던 건 AI 쪽이 아니라 **권한**이었습니다. ViewSet 마다
`get_queryset()` 을 사용자 기준으로 좁혀서, 남의 리소스 ID 로 조회하면 403 이 아니라
**404** 가 나가도록 했습니다. 존재 여부 자체를 흘리지 않기 위해서입니다. 남의 로드맵을
FK 로 지정해 쓰기를 시도하면 400. 이 규칙이 깨지지 않도록 IDOR·입력 검증·인증·저장 회귀를
포함해 테스트 34개를 붙여뒀습니다. 로드맵 → 주차 → 체크리스트로 중첩 직렬화가 되다 보니
N+1 이 나서, 같은 자리에서 `prefetch_related` 로 묶었습니다.

로드맵 생성 API 는 호출당 Claude API 비용이 발생해서 시간당 10회로 스로틀을 걸었습니다.

[저장소 →](https://github.com/bhj2837/learningpath)

### SharedFate — 운명 공유 마인크래프트 플러그인

`Java 21` `Paper API` `Gradle` `GCP`

여러 명이 데미지·인벤토리·죽음 카운트를 공유하는 Paper 1.21.x 서버 플러그인.
실제로 GCP 에 서버를 띄워 지인들과 플레이하면서 밸런스를 고쳤습니다.

"전부 공유"가 답이 아니었습니다. 인벤토리를 통째로 공유했더니 아무도 자기 장비를
챙길 수 없어서, **슬롯 9–35 만 공유하고 핫바·갑옷·오프핸드는 개인 소유**로 남겼습니다.

기억에 남는 버그 두 개:

- 죽음 카운트가 한 번도 오르지 않았습니다. 동시 사망 판정 가드가 항상 참이었는데,
  `lastCountedAt` 초기값이 `Long.MIN_VALUE` 라 `now - lastCountedAt` 이 **오버플로**하고
  있었습니다. 초기값을 0 으로 바꿔 해결.
- OP 만 죽음 집계에서 빠졌습니다. "OP 는 모든 권한을 가진다"는 말은 사실
  **plugin.yml 에 선언되지 않은 권한**에만 해당합니다. 면제 권한을
  `default: false` 로 명시 선언하니 아무에게도 부여되지 않았습니다.

[저장소 →](https://github.com/bhj2837/sharedfate)

### schedule-engine — C++ 일정 최적화 서버

`C++17` `Crow` `Asio` `CMake`

EDF 는 마감일만 봅니다. 그래서 우선순위를 시간으로 환산해 섞었습니다.

```
score = deadline - (priority × 3600)
```

우선순위 한 단계가 마감을 한 시간 당기는 효과를 냅니다. 같은 마감이면 중요한 일이
앞으로 오고, 마감 차이가 크면 우선순위가 뒤집지 못합니다.

Crow 로 REST API 와 대시보드 정적 파일을 **하나의 exe 에서 같이 서빙**하게 만들어서,
빌드 결과물만 옮기면 바로 돌아갑니다. 결과는 간트 차트로 그리고 마감을 넘기는
태스크는 따로 표시합니다.

[저장소 →](https://github.com/bhj2837/schedule-engine)

### mini-shell — C 로 만든 UNIX 셸

`C` `POSIX`

파이프, 입출력·추가 리다이렉션, 백그라운드 실행, `cd -`, `export`, 시그널 처리를
직접 구현했습니다. `fork` / `execve` / `dup2` 를 손으로 쓰면서 알게 된 건,
파이프라인이 결국 **자식마다 fd 를 갈아끼우는 일**이라는 것과, `Ctrl+C` 가 셸까지
같이 죽이지 않게 하려면 포그라운드 프로세스 그룹을 따로 관리해야 한다는 것이었습니다.

[저장소 →](https://github.com/bhj2837/mini-shell)

### korean-dialogue-generation — 한국어 멀티턴 대화 생성

`PyTorch` `HuggingFace` `KoBART`

KoBART(`gogamza/kobart-base-v2`)를 AI Hub 한국어 SNS 대화 데이터로 파인튜닝했습니다.

멀티턴을 한 줄로 이어 붙이면 모델이 누가 한 말인지 구분하지 못합니다.
`<P01>`, `<P02>` 스페셜 토큰을 토크나이저에 추가해 발화자를 표시하는 방식으로 풀었습니다.
SNS 데이터라 전처리 비중이 컸습니다 — 초성 반복 정규화, 너무 짧은 대화 필터링 등.

자연어처리 수업 최종 프로젝트 (2024).

### daily-brief — 개인 대시보드

`Vue 3` `Vercel`

날씨·일정·뉴스를 한 화면에 모았습니다. 이전에 만들어둔 weather 프로젝트를
컴포넌트 단위로 떼어 `/weather` 라우트로 재사용했습니다.

[데모 →](https://daily-brief-neon-three.vercel.app) · [저장소 →](https://github.com/bhj2837/daily-brief)

## 그 밖에

- **Sign-Language-Translation** — 수어 영상 분류 (COSE471 데이터 과학)
- **COSE474 딥러닝** — 강의 과제 및 실습 노트북

## 이 블로그

알고리즘 문제를 풀고 나서 *왜 그 풀이가 되는지* 를 정리해 두는 공간입니다.
답만 적어두면 한 달 뒤에 다시 못 푸는 걸 몇 번 겪고 나서 접근 과정을 남기기로 했습니다.
