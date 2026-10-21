# 현재 상태와 다음 할 일

> 새 작업을 시작할 때 이 문서를 먼저 읽는다. 결정과 그 이유는 [decisions.md](./decisions.md), 요구사항은 [01-requirements.md](./01-requirements.md), 대화 기록은 [conversation-log.md](./conversation-log.md)에 있다. 마지막 갱신: 2026-10-22.

## 저장소

| 저장소 | 내용 |
|---|---|
| [dessert-trend-docs](https://github.com/reverieCraftLab/dessert-trend-docs) | 요구사항, 결정 기록, 대화 기록(이 저장소) |
| [trendcanvas](https://github.com/reverieCraftLab/trendcanvas) | 백엔드. Spring Boot 4.1.0, Java 21, JPA, Flyway, PostgreSQL |
| [trendcanvas-web](https://github.com/reverieCraftLab/trendcanvas-web) | 프런트. Next.js 16, React 19, Tailwind 4 |

## 끝난 것

- 요구사항: FR-01~09, NFR-01~10, 데이터 5.1(`dessert`), 5.2(`search_log`). 결정 D-01~D-16.
- DB(Supabase `trendcanvas`): Flyway V1~V3 적용 완료. `dessert` 테이블과 시작용 디저트 10개, RLS 켜짐.
- 백엔드: `GET /api/desserts/home`(디저트 3개 무작위, 키워드만), `GET /api/images?keyword=&start=`(네이버 이미지 검색 호출, 응답 그대로 전달). 로컬 실행으로 확인함. 코드 [094c182](https://github.com/reverieCraftLab/trendcanvas/commit/094c182).

## 다음 할 일: 4단계 프런트 연결

작업 순서는 아래와 같고, 아직 시작하지 않았다.

1. 백엔드에 CORS 설정 추가(허용 주소는 환경 변수).
2. 프런트에 API 주소 환경 변수(`NEXT_PUBLIC_API_BASE_URL`)와 새 응답 타입을 만들고 이전 타입을 지운다.
3. 홈 화면: `/api/desserts/home`으로 디저트 3개를 받아 **디저트별 열 3개**를 그리고, 열마다 `/api/images`를 동시에 따로 호출한다. 열마다 로딩·성공·실패(503 `QUOTA_EXCEEDED`, 502 `UPSTREAM_ERROR`)를 따로 처리한다(NFR-09). 순위 영역은 제거하고 출처(네이버)와 원본 링크를 표시한다.
4. 스크롤 추가 로드(FR-02): 열 아래에 닿으면 `start`를 올려 다음 이미지를 가져온다.
5. 정리: `mock.json`과 이전 코드 삭제, 프런트의 `CLAUDE.md` 규칙 수정, 결정 기록 D-17 작성.

### 시작 전에 GT가 정할 것(아직 답하지 않음)

1. 브라우저가 Spring을 부르는 경로. Claude의 제안은 **A: 브라우저 → Spring 직접 호출**(응답 시간 NFR-08에 유리하고 단순함). B는 브라우저 → Next.js → Spring.
2. `next/image` 대신 일반 `<img>`를 쓸지. Claude의 제안은 일반 `<img>`다. `next/image`는 외부 이미지를 Next 서버가 내려받아 가공·저장하는 기능이라 네이버 약관(7.3③, NFR-05)에 걸릴 수 있다고 판단했다(법률 자문 아님).
3. 홈 제목 문구. 지금 "이번 주, 사람들이 찾는 디저트"는 랭킹을 암시한다(D-07). 임시 문구로 시작해도 된다.

### 현재 프런트가 새 설계와 충돌하는 부분

- 상단 순위 목록(막대 그래프): 랭킹을 보여주지 않기로 했다(D-07).
- 모든 디저트를 한 격자에 섞어 표시(라운드 로빈): 섞거나 순서를 바꾸면 안 되고 디저트별 열이어야 한다(FR-01, 특약 1.3).
- `next/image` 사용: 위의 약관 위험.
- 하단 "Photos from Unsplash"와 "예시 데이터" 문구: 출처 표기를 네이버로 바꾼다(NFR-05).
- 기존 디자인(폰트, 색, 매거진 느낌, 세로 격자 스타일)은 살린다.

## 그 뒤

- 5단계: 지도와 가게 검색(네이버 지도 + 지역 검색, FR-04~06, D-06, D-10, D-11).
- 6단계: 행동 기록(`search_log`, FR-08, D-09).
- 배포: EC2 t2.micro, 도메인 `reveriecraftlab.com`의 서브도메인과 HTTPS(NFR-07). 아직 EC2를 만들지 않았다.

## 미정·보류 항목

- 행동 기록 원본의 보관 기간과 통계 테이블 기준(D-09), 안내문 문구와 개인정보 처리방침 필요 여부(D-05).
- 지도 중심 좌표를 지역명으로 바꾸는 방식(Reverse Geocoding, D-06): MVP 화면 이후에 판단.
- 한도 소진 안내 문구: 사용자를 본 뒤 추가(D-11).
- 카카오 이미지 검색으로 대체하는 안(D-03): 사용자가 늘어난 뒤.
- 네이버 문의 여부(공유 캐시, 한도 증량, 혼합 배치).
- Supabase 무료 프로젝트는 사용이 없으면 일시 중지된다(2026-10-21에 한 번 중지되어 있었다). 대응은 미정.
- 서버 환경: 집 컴퓨터를 서버로 쓰는 안은 보류하고 EC2를 유지했다.
- 문서 전체 정합성 점검은 개발이 끝난 뒤에 한다. README의 "AI 리뷰"는 GT가 최종 내용을 마친 뒤에 쓰고, 약점도 포함한다.

## 로컬 실행 메모

- 백엔드 프로파일: 기본은 `local`(로컬 도커 PostgreSQL, `docker compose up -d`). `prd`는 Supabase에 붙으므로 마이그레이션을 적용할 때처럼 필요한 순간에만 쓴다.
- 환경 변수 이름: `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`(prd), `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`. **값은 문서, 코드, 채팅에 적지 않는다.** 터미널에서 `read -s`로 입력하거나 IntelliJ 실행 설정에만 둔다.
- IntelliJ가 만드는 `.mcp.json`은 커밋하지 않는다.
- 네이버 앱은 `tc-collector` 하나만 쓴다(약관 7.3⑤).

## 작업 방식

- 코드는 Claude가 만들고, GT가 IntelliJ에서 검토·실행하고 커밋·푸시한다. 문서 저장소는 Claude가 갱신하고 푸시한다.
- 코드와 결정에는 이유를 함께 설명한다(DB·트랜잭션 결정은 특히 자세히).
- GT가 정하지 않은 요구사항은 채우지 않는다. 초안과 이유를 보여주고 합의한 뒤 문서에 쓴다.
- 새 클래스를 추가하면 컴파일 통과만으로 끝내지 않고 앱을 띄워 확인한다(D-16의 의존성 주입 교훈).
