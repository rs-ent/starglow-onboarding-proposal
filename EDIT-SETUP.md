# 제안서 인라인 편집 + 실시간 공유 (Firebase Firestore)

## 구조 (아티스트/버전별 다중 제안서)

- `site/index.html` — **원본(마스터)**. 내용 고정. 우측 상단 **「＋ 복제하기」**(아티스트·버전 입력 → 원본 복사본 생성) + **「☰ 목록」** 버튼.
- `site/edit.html` — **편집 앱** (해시 라우팅):
  - `edit.html#list` (또는 해시 없음) → **목록 페이지**: 모든 아티스트/버전 카드 + 새로 만들기 + 삭제
  - `edit.html#{아티스트}/{버전}` → 그 제안서 **편집기** (인라인 편집·실시간 공유·접속자/잠금·이력·프리셋·복제·`?edit` 카드선택)

각 아티스트/버전은 Firestore 문서 `proposals/{아티스트__버전}` 로 **독립 저장**(하위 presence/history/presets 포함). 처음 그 주소를 열면 원본 내용으로 자동 생성됩니다.

예) `…/edit.html#bts/v1`, `…/edit.html#newjeans/2026`

## 동작
- 상단 가운데 툴바의 **✎ 편집**을 켜면 표지·카드·표·리스트 텍스트를 클릭해 바로 수정 → 800ms 뒤 자동 저장 → 모두에게 동기화.
- **접속자 아바타**(툴바 왼쪽) + **칸별 잠금**: 누가 한 칸을 편집 중이면 다른 사람 화면에선 그 칸이 잠김.
- **이력 / ↶ 되돌리기**: 버전 스냅샷 확인·복원.
- 저장 위치: Firestore `proposals/onboarding-live` 문서 (+ 하위 `presence`, `history`).

## Firestore 보안 규칙 (필수)
Firebase 콘솔 → Firestore Database → **규칙** 탭에 붙여넣고 **게시**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /proposals/{docId} {
      allow read, write: if true;
      match /{sub=**} { allow read, write: if true; }   // presence/history 필수
    }
    match /{document=**} { allow read, write: if false; }
  }
}
```

> `match /{sub=**}` 가 없으면 접속자 표시·변경 이력이 동작하지 않습니다.

## Firebase config
`site/index.html` 의 `FB_CONFIG` 에 이미 프로젝트(`yves-8cb44`) 값이 들어 있습니다.
다른 프로젝트로 바꾸려면 그 값만 교체하세요. (apiKey 등은 공개값 — 비밀 아님)

## 한계
- 동시에 같은 칸을 편집하면 마지막 저장이 이깁니다(잠금이 전파되기 전 순간). 변경 이력으로 복원 가능.
- 표지/요약/마무리 등 모든 텍스트가 편집 대상. 레퍼런스 사진·기존 ?edit 체크박스는 편집 대상 아님.
