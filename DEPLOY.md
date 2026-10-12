# 유씨네 집들이 — 배포 안내

손님에게 보낼 링크를 만드는 순서입니다. **1번(Firebase)을 먼저 하고 2번(공개)을 하세요.**
1번을 건너뛰면 손님이 찜을 해도 **각자 자기 핸드폰에만 저장되어** 선물이 겹칩니다.

---

## 1. Firebase 연결 — 찜을 모두가 같이 보려면 꼭 필요해요

### 1-1. 프로젝트 만들기
1. <https://console.firebase.google.com> 접속 → **프로젝트 추가**
2. 이름은 아무거나 (예: `yoo-housewarming`). Google 애널리틱스는 **사용 안 함**으로 두세요.

### 1-2. 웹 앱 등록하고 설정값 복사
1. 프로젝트 개요 화면에서 **웹(`</>`) 아이콘** 클릭
2. 앱 닉네임 입력 → **앱 등록** (호스팅 설정은 체크 안 해도 됩니다)
3. 화면에 나오는 `firebaseConfig` 중괄호 안의 내용을 복사

### 1-3. 익명 로그인 켜기
**빌드 → Authentication → 시작하기 → Sign-in method → 익명 → 사용 설정**

> 손님에게 로그인을 시키지 않으면서도 "누가 찜했는지" 구분하려고 씁니다.

### 1-4. Firestore 만들기
**빌드 → Firestore Database → 데이터베이스 만들기**
- 위치: `asia-northeast3 (서울)`
- 모드: 프로덕션 모드로 시작 (규칙은 아래에서 바꿉니다)

### 1-5. 보안 규칙 붙여넣기
Firestore의 **규칙** 탭에 아래를 통째로 붙여넣고 **게시**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /claims/{giftId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
                    && request.resource.data.uid == request.auth.uid
                    && request.resource.data.name is string
                    && request.resource.data.name.size() > 0
                    && request.resource.data.name.size() <= 20;
      allow delete: if request.auth != null && resource.data.uid == request.auth.uid;
      allow update: if false;
    }
  }
}
```

이 규칙이 하는 일:
- 먼저 찜한 사람이 임자 — 이미 찜된 선물은 **덮어쓸 수 없습니다**
- 찜 취소는 **본인 것만** 가능
- 남의 이름으로 찜할 수 없음

### 1-6. 허용 도메인에 배포 주소 추가
**Authentication → 설정 → 승인된 도메인 → 도메인 추가** 에 `rohsom.github.io` 추가.
(이걸 안 하면 손님 화면에서 로그인이 막혀 체험 모드로 떨어집니다.)

### 1-7. `index.html`에 설정값 붙여넣기
파일 위쪽의 이 줄을 찾아서

```js
const FIREBASE_CONFIG = null;
```

이렇게 바꿉니다 (1-2에서 복사한 값):

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "yoo-housewarming.firebaseapp.com",
  projectId: "yoo-housewarming",
  storageBucket: "yoo-housewarming.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```

> 이 값들은 공개돼도 괜찮습니다. 웹용 Firebase 설정은 원래 브라우저에 노출되는 값이고,
> 실제 보호는 위의 **보안 규칙**이 합니다.

---

## 2. 링크 공개하기 (GitHub Pages)

1. <https://github.com/rohsom/rohty/settings/pages> 접속
2. **Source**: `Deploy from a branch`
3. **Branch**: `housewarming-site` / `/ (root)` → **Save**
4. 1~2분 뒤 주소가 열립니다 → **<https://rohsom.github.io/rohty/>**

> `main` 브랜치에는 다른 프로젝트가 들어 있어서 건드리지 않았습니다.

---

## 3. 보내기 전 확인 (휴대폰에서 2분)

- [ ] 주소를 열면 **화면 위 노란 경고줄이 없어야** 합니다 (있으면 Firebase 미연결)
- [ ] 문 두 번 두드리기 → "똑" 소리
- [ ] 이름 입력 → 중앙 인사 → 초대장 → 집으로 들어가기 버튼
- [ ] 미니게임 3판 → 열쇠 3개 → 위시리스트 열림
- [ ] **다른 기기(또는 시크릿 창)에서 접속해 같은 선물이 "이미 찜됨"으로 보이는지** ← 제일 중요
- [ ] 카톡에 링크를 붙여 미리보기 카드가 뜨는지

---

## 4. 집주인 보기 — 누가 뭘 찜했는지

주소 뒤에 `?host` 를 붙이면 집주인 전용 화면이 열립니다.

**<https://rohsom.github.io/rohty/?host>**

- 찜 완료 / 남은 선물 / 찜된 금액 요약
- 찜된 선물마다 **누가, 언제** 찜했는지
- 남은 선물을 우선순위별로 정리
- 손님이 찜하면 **새로고침 없이 바로** 반영
- 「목록 복사하기」로 전체 현황을 텍스트로 복사

> 손님에게는 `?host` 가 **없는** 주소를 보내세요.
> 잘못 찜한 걸 지우려면 Firebase 콘솔 → Firestore → `claims` 에서 해당 문서를 삭제하면 됩니다.

---

## 자주 고치는 것들

| 고치고 싶은 것 | 위치 (`index.html`) |
| --- | --- |
| 사이트 제목·인사말·초대장 문구 | 맨 위 `SITE` |
| 랜덤 쿠폰 | `COUPONS` |
| 위시리스트 (이름·가격·링크·한마디·놓인 자리) | `GIFTS` |
| 열쇠 몇 개로 열지 | `KEYS_NEEDED` |
| 집주인 퀴즈 문제 | `QUIZ_POOL` |

초대장 문구에서 `*별표*`로 감싼 부분은 형광펜으로 강조됩니다.
