# korean-web-typography

한국어를 표시하는 웹 UI를 위한 Claude Code 플러그인입니다. Claude가 한국어 웹 화면을 만들거나
고칠 때 다음 규칙을 따르게 합니다.

- 고정폭 글꼴 금지 (숫자 정렬은 `tabular-nums`로)
- `word-break: keep-all` + `overflow-wrap: anywhere`
- 양쪽 정렬 금지
- 줄 간격 조정 순서: 자간 → 어절 간격 → 행간
- 글꼴 선택: 한국어·영문만이면 Pretendard, 일본어·중국어까지면 Noto Sans CJK KR
- Pretendard는 같은 크기에서 작게 보이므로 가독성 기준으로 크기 보정
- 본문에 em dash(—), en dash(–), 대시로 쓴 마이너스 기호(-) 금지 (짧은 제목의 강조는 허용, 범위는 `~`)

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
