# SSU EE Coffee Order System

숭실대학교 전기공학부 학부회의 커피 주문 웹 시스템입니다.
사용자는 배원규 (Dr. BAE) 교수님이며, 이 프로젝트는 학부 교수님들이 매주 수요일 학부회의 전 커피 주문을 취합하는 용도입니다.

## Quick Facts

- **저장소**: `baelab-create/ee.ssu.coffee` (GitHub)
- **배포 URL**: https://baelab-create.github.io/ee.ssu.coffee/
- **호스팅**: GitHub Pages
- **로컬 개발 서버**: 없음 (단일 HTML 파일이라 브라우저로 바로 열면 됨)
- **주요 파일**: 루트의 `index.html` 하나에 모든 CSS·JS·이미지가 내장됨

## Tech Stack

- **Frontend**: Vanilla HTML/CSS/JavaScript (ES modules, 단일 파일)
- **Backend**: Firebase Realtime Database (Spark 무료 플랜)
- **Firebase 프로젝트**: `ssu-ee-coffee`
- **데이터베이스 URL**: `https://ssu-ee-coffee-default-rtdb.asia-southeast1.firebasedatabase.app`
- **Firebase SDK**: `firebase-app.js` + `firebase-database.js` v10.14.1 (CDN)
- **폰트**: Google Fonts (Playfair Display, Noto Serif KR, Noto Sans KR)
- **이미지**: 히어로 배너 · 파비콘 모두 Base64 데이터 URI로 HTML 내장 (외부 파일 없음)
- **OG 썸네일**: `og-image.jpg`만 저장소 루트에 별도 존재 (메신저 크롤러용)

## 아키텍처 핵심

### 데이터 모델

```
Firebase Realtime DB
└─ orders
   └─ YYYY-MM-DD (회의일 기준 키)
      └─ pushId
         ├─ person: string (주문자 이름)
         ├─ menu: string (예: "카페 아메리카노")
         ├─ temp: "ICE" | "HOT"
         ├─ size: "Tall" | "Grande" | "Venti"
         ├─ note: string (요청사항, optional)
         └─ timestamp: number (Date.now())
```

### "회의일 기준" 창구 로직 ⭐ (중요)

**절대 자정 기준으로 리셋하지 말 것.** 과거 자정 리셋 방식은 화요일 밤에 미리 주문한 것이 수요일에 사라지는 심각한 UX 문제를 일으켰다.

현재 로직 (`getMeetingDate` 함수):
- 회의는 매주 **수요일 16:30**
- 오늘이 수요일이고 16:30 이전 → 회의일 = 오늘
- 오늘이 수요일이고 16:30 이후 → 회의일 = 다음 주 수요일
- 다른 요일 → 회의일 = 다가올 다음 수요일

이 로직으로 모든 저장/조회가 회의일 키(`YYYY-MM-DD`)에 통일된다. 화요일 저녁 미리 주문부터 수요일 회의 시작까지 모두 동일한 창구.

상수 정의 위치:
```javascript
const MEETING_WEEKDAY = 3;   // Date.getDay(): 0=일, 3=수
const MEETING_HOUR = 16;
const MEETING_MINUTE = 30;
```

### 실시간 동기화

- 여러 교수님이 각자 폰/컴퓨터로 접속하면 `onValue` 리스너로 실시간 공유
- 동일 회의일 키를 가진 모든 클라이언트가 즉시 동기화됨
- online/offline 이벤트 + visibilitychange 이벤트로 자동 재구독

### 이메일 발송

- 하단 액션 바의 **"이슬기 조교에게 메일"** 버튼이 `mailto:` 링크 실행
- 수신자: `wgbae@ssu.ac.kr` (⚠️ 실제는 이슬기 조교 주소여야 하지만 현재는 테스트용으로 배원규 교수 본인 주소)
- 본문: 회의일, 주문자별 상세 + 시각([HH:MM]), 스타벅스 주문용 집계 (수량 내림차순)
- 서명: `SSU Electrical Eng.`

## UI/UX 원칙

- **모바일 우선**: 회의실에서 폰으로 빠르게 주문
- **미학**: 스타벅스 브랜드 컬러 (다크그린 `#1e3932` + 골드 `#cba258`)
- **타이포**: 영문은 Playfair Display 이탤릭, 한글 헤딩은 Noto Serif KR 900
- **히어로 배너**: 니콜라 테슬라가 스타벅스 컵을 든 세피아 톤 이미지 (교수님 창작 자산)
  - 상단에 다크그린 그라디언트 오버레이로 텍스트 가독성 확보
- **헤더 하단 뱃지**: 좌측 = 🟢 실시간 연결됨, 우측 = ☕ N잔

## 하지 말아야 할 것

- ❌ **자정 기준 리셋 로직으로 되돌리지 말 것** (위 회의일 로직 유지)
- ❌ **user-scalable=no 추가하지 말 것** (시력 약한 교수님 접근성)
- ❌ **이미지를 외부 파일로 분리하지 말 것** (GitHub 업로드 시 경로 문제로 두 번 실패한 이력 있음. Base64 내장이 안전함)
- ❌ **`import` 문을 스크립트 중간에 두지 말 것** (ES 모듈은 최상단 필수. 실제로 이 문제로 메뉴 렌더링 실패한 이력 있음)
- ❌ **Firebase API 키를 숨기려 하지 말 것** (프론트엔드 노출은 안전. 보안은 Firebase 규칙으로 처리)
- ❌ **NEW 태그를 오래 방치하지 말 것** (2024년산 메뉴가 2년째 NEW로 남아있던 이력)

## Firebase 보안 규칙 (참고)

```json
{
  "rules": {
    "orders": {
      "$date": {
        ".read": true,
        ".write": true,
        ".validate": "$date.matches(/^[0-9]{4}-[0-9]{2}-[0-9]{2}$/)"
      }
    }
  }
}
```

날짜 형식(`YYYY-MM-DD`)이 아닌 경로는 쓰기 거부. 규칙 만료 시 데이터베이스가 잠기므로 주기적 확인 필요.

## 배포 워크플로우

1. `index.html` 수정
2. GitHub 저장소에서 `index.html` 덮어쓰기 업로드 → Commit
3. 1~2분 후 자동 재배포
4. 캐시 우회를 위해 URL 끝에 `?v=NN` 붙여서 확인 (`?v=100`, `?v=101`, ...)
5. 카카오톡 링크 미리보기가 안 바뀌면 Facebook Debugger에서 Scrape Again

## 코딩 스타일 지침

- **한글 주석 OK**: 배원규 교수님이 한국인이라 한글 주석 환영
- **명시적 에러 처리**: try-catch를 통한 방어 코딩 선호
- **접근성 배려**: aria-label, semantic HTML, 확대 허용
- **가독성 > 최적화**: 단일 파일 앱이므로 미니파이 필요 없음
