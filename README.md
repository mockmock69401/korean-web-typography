# korean-web-typography

한국어를 표시하는 웹 UI를 위한 Claude Code 플러그인입니다. Claude가 한국어 웹 화면을 만들거나 고칠 때 다음 규칙을 따르게 합니다.


- 다른 디자인 스킬과 겹치면 한국어 텍스트의 글꼴·크기·줄바꿈·줄 간격은 이 스킬을 따름
- 고정폭 글꼴 금지 (숫자 정렬은 `tabular-nums`로)
- 숫자·영문에 별도 영문 글꼴을 섞지 않고 한 가지 글꼴로 통일
- `word-break: keep-all` + `overflow-wrap: anywhere`
- 양쪽 정렬 금지
- 줄 간격 조정 순서: 자간 → 어절 간격 → 행간
- 제목: 자간 -0.02~-0.03em, 한 줄이면 행간 1.15, 줄바꿈되면 띄어쓰기 폭으로 계산, `text-wrap: balance`
- 글꼴 선택: 한국어·영문만이면 Pretendard, 일본어·중국어까지면 Noto Sans CJK KR
- Pretendard는 같은 크기에서 작게 보이므로 가독성 기준으로 크기 보정
- 작은 글씨 하한: 만들기 전에 큰 글씨 우선인지 묻고, 우선이면 14~16px, 아니면 12~13px
- 본문에서 em dash(—), en dash(–) 사용 자제 (짧은 제목의 강조는 허용, 범위는 `~`)
- 날짜는 한국식 표기: `2026년 10월 16일(금)`, `10월 16일(금)` (브라우저 기본 날짜 입력칸의 표기에 맡기지 않음)
- 카드(상자) 처리는 미디어(이미지·영상·지도), 입력 컴포넌트, 버튼에만 권장. 상품·숙소·검색 결과 같은 콘텐츠 항목은 상자로 감싸지 않고, 미디어만 상자로 두고 글은 페이지 바탕 위에 둠. 같은 정보는 한 번만 씀
- 나란히 놓인 항목은 같은 요소의 높이를 맞춤: 일부 항목에만 있는 배지가 아래 줄을 밀지 않게 하고, 가격 줄은 아래에 고정
- 가격 같은 강조 숫자는 항목 제목보다 한 단계 큰 토큰으로 키우고 단위는 작은 글씨로
- 강조한 숫자는 그 항목을 담은 섹션 제목보다 작게. 값이 없을 때(`0원`)는 강조하지 않고 안내 문구로
- 입력칸·선택 상자·버튼을 카드처럼 테두리로 두르지 않아도 됨: 경계선 없이 배경색만 다르게 하는 방식 허용 (포커스 링, 오류 표시는 따로 챙김)
- 입력칸 라벨은 유동적으로: 입력한 값만으로 뜻이 통하면 placeholder가 라벨을 대신하고, 왼쪽 아이콘(날짜 칸의 달력 등)으로 칸의 종류를 계속 보여 주거나, 값만으로 모호하면 떠 있는 라벨(floating label) 사용. 화면에서 라벨을 빼도 스크린리더용 이름은 유지
- 표는 기본 구성요소가 아님: 꼭 실어야 할 내용이 행과 열의 비교일 때만 사용하고, 표를 넣으려고 섹션을 만들지 않음
- 넓은 행의 빈 공간은 선이나 그림으로 채우지 않고 열 너비로 정리
- 가로 스크롤 영역(날짜 칩 등)은 여백선에서 시작하고 항목 단위로 멈춤. 선택 항목은 가운데가 아니라 시작 위치로 스크롤
- 줄 앞에 오는 배지는 패딩의 절반만큼 바깥으로 내어, 상자와 글자가 본문 시작선 양쪽에 고르게 걸치게 함
- 글자 크기는 h1~h6, 본문, 작은 글씨 8단계만 사용하고 제목은 계층에 맞춰 선택 (맞는 크기가 없으면 임의로 조정하지 않고 가장 가까운 단계 사용)
- 텍스트 링크는 호버(키보드 포커스 포함)할 때만 밑줄 표시, 링크 색은 링크에만 사용
- 완료 전 브라우저에서 실제 최소 글자 크기와 글꼴 로딩, 375px에서 오른쪽으로 넘친 요소가 없는지 확인

## 샘플 사이트

```
Model: Opus 5.5
제약조건: 상호명은 '구름다리', 강조색은 #0F6B5C로 적용함
```

|스킬 사용|토큰 사용량|소요시간|
|------|---|---|
|[No Skills](https://mockmock69401.github.io/korean-web-typography/samples/a.html)|139.6k|10m 47s|
|[korean-web-typography](https://mockmock69401.github.io/korean-web-typography/samples/b.html)|109.9k|6m 20s|
|[taste-skills](https://mockmock69401.github.io/korean-web-typography/samples/c.html)|176.2k|12m 1s|
|[taste-skills + korean-web-typography](https://mockmock69401.github.io/korean-web-typography/samples/d.html)|189.3k|13m 42s|
<br/>


## 설치

Claude Code에서:

```
/plugin marketplace add mockmock69401/korean-web-typography
/plugin install korean-web-typography@mockmock69401
```

터미널에서:

```bash
claude plugin marketplace add mockmock69401/korean-web-typography
claude plugin install korean-web-typography@mockmock69401
```

설치 후 Claude Code 세션을 새로 열거나 `/reload-plugins`를 실행하면 적용됩니다.

## 업데이트

```bash
claude plugin marketplace update mockmock69401
claude plugin update korean-web-typography@mockmock69401
```

업데이트는 Claude Code를 다시 시작하면 적용됩니다.

## 수동 설치

마켓플레이스 없이 쓰려면 저장소를 스킬 폴더에 바로 받아도 됩니다.

```bash
git clone https://github.com/mockmock69401/korean-web-typography.git ~/.claude/skills/korean-web-typography
```
