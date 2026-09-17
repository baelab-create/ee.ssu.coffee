# 인수인계 (Handoff)

## 프로젝트 개요

배원규 교수님(SSU 전기공학부)의 **학부회의 커피 주문 시스템**입니다.
매주 수요일 오후 4시 30분에 학부회의가 열리고, 회의 1시간 전에 커피를 주문합니다.
여러 교수님들이 각자 자신의 폰으로 접속해서 주문을 넣으면 실시간으로 공유되고,
회의 시작 전 대표 1명이 이메일 발송 버튼으로 이슬기 조교에게 전달합니다.

## 개발 히스토리 요약

### Phase 1: 초기 구축
- 단일 HTML 파일로 스타벅스 TOP 10 메뉴 주문 페이지 제작
- localStorage 기반으로 시작 → Firebase Realtime DB로 전환
- GitHub Pages 배포 (`baelab-create/ee.ssu.coffee`)
- 반응형 모바일 최적화 (바텀 시트 모달, 고정 액션 바 등)

### Phase 2: 시각화 강화
- 니콜라 테슬라가 스타벅스 컵을 든 히어로 배너 이미지 추가
- Open Graph 메타 태그로 카카오톡·슬랙 링크 미리보기 지원
- 파비콘 · 히어로 배너 · 모바일용 저해상도 이미지 각각 준비
- **최종적으로 모든 이미지를 Base64로 HTML에 내장**해서 배포 단순화
  - 이유: GitHub Pages에 폴더 구조로 업로드하면 경로 문제가 반복됨
  - OG 이미지만 외부 파일 (`og-image.jpg`)로 저장소 루트에 별도 배치

### Phase 3: UX 개선
- 헤더 하단에 좌측=연결상태, 우측=주문잔수 배치
- 이메일 본문에 회의일·회의시각·주문시각 표시
- 스타벅스 주문용 집계는 수량 내림차순 정렬
- 주문 리스트 UI에도 시각 뱃지 표시

### Phase 4: 자정 리셋 문제 해결 ⭐
**이전 문제**: 화요일 저녁 미리 주문한 것이 수요일 자정을 지나며 사라짐.
**해결**: "회의일 기준 창구(Meeting Day Window)" 로직 도입.

핵심 함수 `getMeetingDate(now)`가 다음 회의일을 반환:
- 오늘이 수요일 16:30 이전 → 오늘
- 오늘이 수요일 16:30 이후 → 다음 주 수요일
- 다른 요일 → 다가올 다음 수요일

이 키(`YYYY-MM-DD`)로 Firebase에 저장/조회하므로 자정을 넘어도 데이터 유지됨.

### Phase 5: ES 모듈 버그 수정
**증상**: 배포 후 TOP 10 메뉴가 표시되지 않음.
**원인**: `<script type="module">` 안에서 `import` 문이 스크립트 최상단이 아니라 12줄 뒤에 있었음. 일부 엄격한 파서에서 스크립트 전체 실행 중단.
**해결**: `import`를 스크립트 첫 번째 코드 라인으로 이동. `DOMContentLoaded` 대기 + `init` 에러 방어 추가.

## 현재 파일 구조

저장소 루트:
```
ee.ssu.coffee/
├── index.html          ← 메인 앱 (모든 코드, 히어로 배너·파비콘 내장)
├── og-image.jpg        ← 카카오톡·페이스북 링크 썸네일 (1200×630)
├── favicon.png         ← 이전 파일 (더 이상 참조되지 않음)
├── hero-banner.jpg     ← 이전 파일 (더 이상 참조되지 않음)
├── hero-mobile.jpg     ← 이전 파일 (더 이상 참조되지 않음)
└── ee-ssu-coffee-v2.zip ← 이전 배포 잔재 (삭제 가능)
```

> 정리하고 싶으면 `og-image.jpg`와 `index.html` 외의 파일은 모두 삭제 가능합니다.

## Firebase 설정 (전체 공개 가능)

프론트엔드에서 노출되는 값이므로 공개 저장소에 그대로 있어도 안전합니다. 보안은 Realtime DB의 규칙(Security Rules)으로 처리됩니다.

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyAiplueyqN1d2bxm2swdfdzgDDnIyvlPhU",
  authDomain: "ssu-ee-coffee.firebaseapp.com",
  databaseURL: "https://ssu-ee-coffee-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "ssu-ee-coffee",
  storageBucket: "ssu-ee-coffee.firebasestorage.app",
  messagingSenderId: "111668112163",
  appId: "1:111668112163:web:46bd3d14dd7f19492fba96",
  measurementId: "G-2QK742L0PT"
};
```

## 알려진 미해결 과제 / 개선 아이디어

### 우선순위 높음
1. **Firebase 보안 규칙 상태 확인 미완료**
   - 처음에 테스트 모드로 시작했는데 30일 후 만료되는지 확인 안 됨
   - Firebase Console → Realtime Database → 규칙 탭 확인 필요
   - `CLAUDE.md`에 있는 JSON 규칙으로 재설정 필요할 수 있음

2. **이슬기 조교 실제 이메일 주소로 변경**
   - 현재 `ASSISTANT_EMAIL = 'wgbae@ssu.ac.kr'` (테스트용, 배원규 교수 본인 주소)
   - 실사용 시 이슬기 조교 실제 주소로 교체 필요

### 우선순위 낮음
3. **저장소 정리**: 사용되지 않는 이미지·zip 파일 삭제
4. **README.md 없음**: GitHub 저장소에 README가 없음
5. **오래된 데이터 자동 정리**: 현재는 Firebase Console에서 수동 삭제 필요
6. **회의 시간 변경 시 코드 수정 필요**: `MEETING_WEEKDAY`, `MEETING_HOUR`, `MEETING_MINUTE` 상수 하드코딩

### 검토 필요
7. **접근성 감사**: aria-label 추가 여부
8. **파일 크기 최적화**: Base64 이미지가 파일 크기의 대부분(약 400KB)을 차지. 필요하면 이미지 재압축 검토
9. **Firebase SDK 최신화**: 현재 10.14.1, 향후 11.x/12.x로 마이그레이션 검토

## 배포 방법

1. `index.html` 수정
2. GitHub 저장소 `baelab-create/ee.ssu.coffee` 접속
3. 기존 `index.html` 클릭 → 연필 아이콘 또는 Upload files로 덮어쓰기
4. Commit changes
5. 1~2분 대기 → `https://baelab-create.github.io/ee.ssu.coffee/?v=NN` 접속 (`NN`은 캐시 우회용)

## 테스트 시나리오

배포 후 아래를 확인:

1. ✅ 우측 상단(또는 헤더 하단 좌측) 🟢 "실시간 연결됨" 뱃지
2. ✅ TOP 10 메뉴가 모두 표시됨 (2×5 그리드)
3. ✅ 오늘이 수요일 아니면 헤더에 "📅 회의일: 다음 수요일 날짜" 표시
4. ✅ 이름 입력 → 메뉴 클릭 → 옵션 선택 → 주문 추가 → Firebase Console에서 데이터 확인
5. ✅ 다른 기기에서 같은 URL 접속 → 주문 리스트 자동 공유 확인
6. ✅ 미리보기 → 이메일 내용 확인
7. ✅ 카카오톡에 URL 공유 → Tesla 이미지 썸네일 표시

## 자주 하는 실수 피하기 (반복 방지)

- ⚠️ `import`를 스크립트 중간에 두면 브라우저에 따라 파싱 실패. **항상 최상단**.
- ⚠️ 이미지 파일을 외부 폴더로 분리하면 GitHub 업로드 시 폴더 구조가 유지되지 않을 수 있음. **Base64 내장 권장**.
- ⚠️ OG 이미지만은 반드시 외부 URL이어야 함 (메신저 크롤러가 외부 URL만 인식).
- ⚠️ 자정 기준 리셋 로직으로 되돌리지 말 것. 회의일 기준 창구가 맞음.
- ⚠️ 카카오톡은 URL당 24시간 캐싱. 새 썸네일 테스트 시 `?v=NN` 붙이거나 Facebook Debugger 사용.
