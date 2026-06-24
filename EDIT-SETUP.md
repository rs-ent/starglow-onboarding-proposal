# 제안서 인라인 편집 + 실시간 공유 (Firebase Firestore)

**두 개의 파일로 분리되어 있습니다:**
- `site/index.html` — **원본 제안서**(고정, 안전). 사람들에게 보여주는 용. 편집 기능 없음.
- `site/edit.html` — **편집본**. 인라인 텍스트 편집 + Firestore 실시간 공유 + 접속자/잠금 + 이력 + 아티스트별 프리셋. 기존 `?edit` 카드선택 기능도 포함.

> ⚠️ `edit.html` 에서 고친 내용은 Firestore(`proposals/onboarding-live`)에 저장되지만, **원본 `index.html` 에는 자동 반영되지 않습니다**(원본은 고정). 편집 결과를 보여주려면 `edit.html` 링크를 공유하세요.

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
