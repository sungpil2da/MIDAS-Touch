[Uploading README.md…]()
# MIDAS TOUCH — AI 배포 거버넌스 감사 도구

M·I·D·A·S 5개 축 · 24개 통제로 조직의 AI 배포를 감사하고 **적합 / 조건부 적합 / 부적합** 의견을 산출하는 단일 파일 웹앱입니다. 감사 대장은 **Firebase Realtime Database로 실시간 동기화**되며, 미설정·오프라인 시 자동으로 브라우저 `localStorage`로 폴백합니다.

- 빌드 도구 없음(순수 HTML/JS) · Netlify 정적 호스팅 · Firebase RTDB 실시간 동기화
- 진입 파일: `index.html`

---

## 1. Firebase 설정 (실시간 동기화)

1. [Firebase 콘솔](https://console.firebase.google.com)에서 **프로젝트 생성**.
2. 좌측 **빌드 → Realtime Database → 데이터베이스 만들기** (위치 선택, 우선 **테스트 모드**로 시작).
3. **프로젝트 설정(톱니) → 내 앱 → 웹 앱(</>) 추가** → 표시되는 `firebaseConfig` 값을 복사.
4. `index.html` 상단 스크립트의 `FIREBASE_CONFIG`를 붙여넣은 값으로 교체:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "midas-audit.firebaseapp.com",
  databaseURL: "https://midas-audit-default-rtdb.firebaseio.com",
  projectId: "midas-audit",
  appId: "1:...:web:..."
};
```

> `databaseURL`이 반드시 있어야 실시간 동기화가 켜집니다. 값이 `YOUR_...` 그대로면 앱은 로컬 모드로 동작합니다.
> 감사 대장 화면 우측 상단 배지로 상태 확인: **● 실시간 동기화 / ○ 오프라인 / △ 연결 오류 / ○ 로컬 모드**.

### 데이터 구조
```
audits/
  {timestamp_id}/  { id, ts, auditee, system, domain, tier, op, overall, major, minor, obs, lead, signer, findings[] }
```

### 보안 규칙 (Realtime Database → 규칙)

**빠른 시작(데모용, 누구나 읽기/쓰기 — 공개 주의):**
```json
{
  "rules": {
    "audits": { ".read": true, ".write": true }
  }
}
```

**권장(로그인 사용자만):** — 이 규칙은 인증이 필요하므로, 아래 "익명 인증" 옵션을 함께 적용해야 앱이 동작합니다.
```json
{
  "rules": {
    "audits": {
      ".read": "auth != null",
      ".write": "auth != null",
      "$id": { ".validate": "newData.hasChildren(['id','ts','op'])" }
    }
  }
}
```

> `apiKey`는 비밀값이 아니라 프로젝트 식별자입니다(웹앱에 공개돼도 정상). 실제 데이터 보호는 **RTDB 보안 규칙**으로 합니다. 공개 데모가 아니라면 반드시 규칙을 잠그세요.

---

## 2. GitHub 업로드

```bash
git init
git add index.html README.md
git commit -m "MIDAS TOUCH audit tool"
git branch -M main
git remote add origin https://github.com/<사용자명>/midas-audit.git
git push -u origin main
```

---

## 3. Netlify 배포

**A. GitHub 연동(권장, 자동 배포)**
1. [Netlify](https://app.netlify.com) → **Add new site → Import an existing project → GitHub** → 위 저장소 선택.
2. 빌드 설정: **Build command 비움**, **Publish directory `.`(루트)**. (정적 사이트라 빌드 불필요.)
3. Deploy. 이후 `git push`마다 자동 재배포됩니다.

**B. 드래그 앤 드롭(가장 빠름)**
- Netlify 대시보드에 `index.html`이 든 폴더를 그대로 끌어다 놓으면 즉시 배포됩니다.

배포 후 `https://<사이트명>.netlify.app` 로 접속. 여러 감사인이 각자 브라우저에서 열어도 **감사 대장이 실시간으로 공유**됩니다.

---

## 4. (선택) 익명 인증으로 규칙 잠그기

위 "권장" 규칙(`auth != null`)을 쓰려면 로그인이 필요합니다. 가장 간단한 방법은 **익명 인증**입니다.

1. Firebase 콘솔 → **Authentication → 로그인 방법 → 익명 사용 설정**.
2. `index.html`의 Firebase 스크립트 두 줄 아래에 auth-compat를 추가:
   ```html
   <script src="https://www.gstatic.com/firebasejs/12.15.0/firebase-auth-compat.js"></script>
   ```
3. `initSync()`의 `firebase.initializeApp(...)` 직후에 익명 로그인 호출:
   ```js
   firebase.auth().signInAnonymously().catch(function(){ setSync("error"); });
   ```
   (RTDB 리스너는 로그인 완료 후 규칙을 통과하게 됩니다.)

> 사내 배포라면 익명 인증 대신 Google/이메일 로그인 + 도메인 제한, 또는 Netlify 접근 제어를 권장합니다. Auth를 쓰면 Firebase 콘솔 **Authentication → 설정 → 승인된 도메인**에 Netlify 도메인을 추가하세요.

---

## 동작 요약
- Firebase 미설정/스크립트 로드 실패/오프라인 → 자동으로 `localStorage` 저장(단일 사용자, 오프라인 사용 가능).
- Firebase 설정 → `audits` 노드 실시간 구독. 한 감사인이 "감사 확정·기록"하면 모든 접속자의 대장이 즉시 갱신됩니다.
- "전체 삭제"는 연결 시 서버 데이터를, 오프라인 시 로컬 데이터를 지웁니다.

## 라이선스
TEAM Pactolus · 내부 사용.
