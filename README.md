# Subtitle Player

> 스트리밍 영상 위에 자막 파일을 플로팅 오버레이로 띄우는 Android 앱

자막은 인터넷에 있고 영상은 OTT/스트리밍에 있는데, 둘을 연결하는 모바일 도구가 없었다. PC 크롬 확장 외에는 경쟁 도구 부재. 5일 만에 V0→V5까지 6번 반복 검증해 원스토어 출시 완료.

<p>
  <img src="https://img.shields.io/badge/version-5.0-FE6951" />
  <img src="https://img.shields.io/badge/platform-Android%208.0+-1A1A1A" />
  <img src="https://img.shields.io/badge/language-Kotlin-7F52FF" />
  <img src="https://img.shields.io/badge/status-released-success" />
</p>

---

## 핵심 기능

| 기능 | 설명 |
|------|------|
| **플로팅 오버레이** | 시스템 전역에서 다른 앱 위에 자막 표시 (`SYSTEM_ALERT_WINDOW`) |
| **3초 카운트다운** | 유저가 "1→0" 순간 영상 ▶ 탭 → 자막·영상 동기화 |
| **자막 라이브러리** | 최근 자막 10개 자동 저장, 시점·싱크 함께 복원 |
| **시간 조정** | −/+ 버튼 (0.1초 / 1초) 또는 시/분/초 직접 입력 |
| **다중 포맷** | SMI / SRT / VTT / ASS (텍스트) |
| **인코딩 자동 감지** | UTF-8, EUC-KR |

---

## 왜 카운트다운인가

유튜브 등 외부 영상 앱은 API로 제어할 수 없다. 앱이 영상의 재생 타이밍을 알 수 없다는 뜻이다.

**해결: 유저가 동기화 시점을 직접 만든다.**

```
1. 오버레이 ▶ 탭
2. 3 → 2 → 1 → 0 카운트다운
3. 유저가 "1→0" 순간 영상 ▶ 탭
4. 자막과 영상이 동시에 0초부터 재생
```

기술적 한계를 감추지 않고, 유저를 동등한 파트너로 다루는 UX 설계.

---

## 유저 플로우

```
앱 실행
  │
  └─[최초 실행?]─Yes→ 온보딩 4화면 ──┐
        │                              │
        No ─────────────────────→ 홈 화면
                                      │
                              [라이브러리?]
                                ├─empty→ + 안내
                                └─items→ 리스트
                                      │
                          ┌── + 탭 ───┴── 항목 탭 ──┐
                          ↓                          ↓
                    [권한 검사]               [URI 권한 유효?]
                          ↓                          ↓
                    파일 선택 (SAF)         파싱 + 시점 복원
                          ↓                          │
                    검증 (5MB / 포맷)                │
                          ↓                          │
                    라이브러리 추가 ──────────────────┤
                                                     ↓
                                           오버레이 시작
                                                     ↓
                                     유튜브 이동 → ▶ 탭
                                                     ↓
                                       3·2·1 카운트다운
                                                     ↓
                                           자막 동기 재생
```

---

## V5 신규 (V4 대비)

**추가**
- 자막 라이브러리 — 홈 리스트, 저장·복원, URI 영속화
- 시간 조정 UI 리뉴얼 — `[−] [HH:MM:SS.d] [+]` 한 줄
- 커스텀 시간 입력 다이얼로그 — 시/분/초 직접 입력
- 꾹 누름 가속 + 시계 깜빡임 예고
- 좌상단 큰 카운트다운 — Pretendard Black 150sp
- 온보딩 4화면 — 최초 1회
- '홈으로' 버튼 — V4 '자막 바꾸기' 대체

**삭제**
- `FilePickerActivity` — 모든 자막 선택이 라이브러리 경유로 통합
- 싱크 오프셋 30초 제한
- 우하단 자막 영역 카운트다운 숫자

---

## 기술 스택

- **Language**: Kotlin
- **Min SDK**: 26 (Android 8.0)
- **Target SDK**: 35 (Android 15)
- **권한**: `SYSTEM_ALERT_WINDOW`, `FOREGROUND_SERVICE`
- **의존성**: `androidx.recyclerview:recyclerview:1.3.2`

### 아키텍처

```
OnboardingActivity ─→ MainActivity ─→ PermissionGuideActivity
                          │
                          ↓
                SubtitleLibraryManager
                (SharedPreferences + JSON)
                          │
                          ↓
                    OverlayService
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
    조작 윈도우      자막 윈도우      카운트다운 윈도우
```

### 주요 파일

| 파일 | 역할 |
|------|------|
| `OnboardingActivity.kt` | 4화면 온보딩 (최초 1회) |
| `MainActivity.kt` | 홈 + 라이브러리 리스트 |
| `PermissionGuideActivity.kt` | 권한 안내 풀스크린 |
| `OverlayService.kt` | 오버레이 메인 로직 + 카운트다운 |
| `SubtitleLibraryManager.kt` | 라이브러리 관리 (object) |
| `LibraryItem.kt` | 데이터 클래스 |
| `SubtitleListAdapter.kt` | RecyclerView 어댑터 |

---

## 디자인 시스템

### 이중 톤 구조

| 영역 | 톤 | 목적 |
|------|------|------|
| 외부 (스토어 / 아이콘 / 마케팅) | 카툰 컬러 | 브랜드 인지 |
| 내부 UI | 미니멀 다크 · 화이트 | 기능 집중 |

### 팔레트

**내부 UI**

| 용도 | 값 |
|------|------|
| 상단바 배경 | `#1A1A1A` |
| 화이트 영역 | `#FFFFFF` |
| 리스트 글씨 | `#111111` |
| 메타 글씨 | `#999999` |
| 구분선 | `#E5E5E5` |
| 오버레이 구분선 | `#33FFFFFF` |

**브랜드**

| 용도 | 값 |
|------|------|
| 마스코트 빨강 | `#FE6951` |
| 앱 아이콘 배경 민트 | `#B6EBF1` |

### 폰트

- **Roboto** — 영문·숫자 (UI 기본)
- **Noto Sans KR** — 한글
- **Pretendard Black** — 카운트다운 전용 (150sp)

---

## 글로벌 확장 (V6)

V5는 한국 출시에 특화돼 있었다. V6는 6개국 시장 진입을 위해 무국가적 UX로 재설계.

→ [V6 진행도 상세보기](./V6_progress.md)

| 항목 | 변경 |
|------|------|
| **오버레이 메뉴** | 한국어 버튼 → 아이콘화 (⏮ 📋 ← ⏻) |
| **자막 포맷** | SMI 추가 → SRT · VTT · ASS(텍스트) 지원 |
| **다국어** | 한국어 / 영어 / 러시아어 / 포르투갈어 / 인도네시아어 / 베트남어 |

### 타겟 시장 선정 근거

Sensor Tower 크롤링 (51개 데이터셋, 18개 컬럼 직접 설계) + 자막 파일 소비 문화 교차 분석.

- Android 강세 + OTT 미성숙 + 자막 파일 직접 소유 문화 = **러시아 · 브라질 · 인도네시아 · 베트남**

### 출시 로드맵

- ✅ 원스토어 (한국, 2026.04)
- 🔄 Google Play Store (DUNS 발급 완료, 조직 인증 진행 중)
- 🔄 Galaxy Store (사업자 셀러 전환 후)

---

## 출시 정보

- **스토어**: [원스토어](https://m.onestore.co.kr/) — 검색: `Subtitle Player`
- **패키지명**: `com.mitchellmoon.subtitleplayer`
- **카테고리**: 생활/위치 > 생산성
- **최소 사양**: Android 8.0+
- **개인정보처리방침**: https://mitchellmoon.github.io/subtitle-player/privacy-policy.html

---

## 라이선스 / 문의

- **개발자**: Mitchell Moon (mitchmoon)
- **사업자**: MONG.AI
- **문의**: minmoon33@naver.com
