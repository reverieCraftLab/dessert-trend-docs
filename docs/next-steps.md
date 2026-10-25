# 현재 상태와 다음 할 일

> 새 작업을 시작할 때 이 문서를 먼저 읽는다. 결정과 그 이유는 [decisions.md](./decisions.md), 요구사항은 [01-requirements.md](./01-requirements.md), 대화 기록은 [conversation-log.md](./conversation-log.md)에 있다. 마지막 갱신: 2026-10-24.

## 저장소

| 저장소 | 내용 |
|---|---|
| [dessert-trend-docs](https://github.com/reverieCraftLab/dessert-trend-docs) | 요구사항, 결정 기록, 대화 기록(이 저장소) |
| [trendcanvas](https://github.com/reverieCraftLab/trendcanvas) | 백엔드. Spring Boot 4.1.0, Java 21, JPA, Flyway, PostgreSQL |
| [trendcanvas-web](https://github.com/reverieCraftLab/trendcanvas-web) | 프런트. Next.js 16, React 19, Tailwind 4 |

## 끝난 것

- 요구사항: FR-01~09, NFR-01~10, 데이터 5.1(`dessert`), 5.2(`search_log`). 결정 D-01~D-18.
- DB(Supabase `trendcanvas`): Flyway V1~V3 적용 완료. `dessert` 테이블과 시작용 디저트 10개, RLS 켜짐.
- 백엔드: `GET /api/desserts/home`(디저트 10개 무작위, 키워드만), `GET /api/images?keyword=&start=`(네이버 이미지 검색을 10장씩 호출, 검색어는 키워드 그대로, 응답을 그대로 전달), CORS(GET만, 허용 주소는 환경 변수). 코드 [094c182](https://github.com/reverieCraftLab/trendcanvas/commit/094c182), [9789792](https://github.com/reverieCraftLab/trendcanvas/commit/9789792).
- 프런트(`trendcanvas-web`): 홈 화면 연결 완료([2da5905](https://github.com/reverieCraftLab/trendcanvas-web/commit/2da5905)), 이후 ㄱ자 묶음 화면으로 개편([68b51b6](https://github.com/reverieCraftLab/trendcanvas-web/commit/68b51b6)). 디저트 한 개가 ㄱ자 한 겹, 스크롤하면 묶음을 아래로 이어 붙임(무한 스크롤), 검색창(`Search`, 돋보기), 하단 네이버 출처 한 줄. `next/image`는 쓰지 않고 일반 `<img>`. 로컬에서 데이터 수신을 확인했다. 결정은 [D-17](./decisions.md).

## 다음 할 일

1. **확인하지 못한 화면 테스트**: 실패 흐름(백엔드 중지, 키 오류 때 영역별 실패와 자동 재시도 없음), Network 탭에서 `/_next/image` 요청이 없고 썸네일을 브라우저가 `search.pstatic.net`에서 직접 받는지, 모바일 폭, 이미지 선명도. 결과는 D-17에 반영한다.
2. **네이버 문의 결정**: 디저트 2장씩 번갈아 보여주는 섞인 배치가 가능한지, 원본 링크를 이미지 클릭으로 대신해도 되는지. 답을 받기 전에는 디저트별로 이어진 열을 유지한다.
3. **`<img>` 대 `<Image unoptimized>`**: 화면에서 썸네일 선명도를 보고 판단한다(서버가 가공하지 않는 점은 같다).
4. 키워드 품질 조정: 접미사는 비웠다(D-17). 문제 되는 키워드(한정선, 아그작, 크로플, 소금빵, 버터떡, 토라쿠 푸딩)는 후보 검색어를 시험한 뒤 GT가 DB의 `dessert.keyword`를 SQL로 고친다.

## 그 뒤

- 5단계: 지도와 가게 검색(네이버 지도 + 지역 검색, FR-04~06, D-06, D-10, D-11).
- 6단계: 행동 기록(`search_log`, FR-08, D-09).
- 배포: EC2 t2.micro, 도메인 `reveriecraftlab.com`의 서브도메인과 HTTPS(NFR-07). 아직 EC2를 만들지 않았다.

## 미정·보류 항목

- 행동 기록 원본의 보관 기간과 통계 테이블 기준(D-09), 안내문 문구와 개인정보 처리방침 필요 여부(D-05).
- 행동 기록 저장소는 PostgreSQL 유지, 시계열 DB는 쓰지 않는다(D-18). 통계 테이블 기준에는 증가 속도와 기기당 중복 제거가 후보로 적혀 있다(미결).
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
