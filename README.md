# Google Calendar Integration Skill

Claude Code용 Google Calendar API 통합 스킬입니다. OAuth 2.0 인증을 통해 캘린더 이벤트의 생성, 조회, 수정, 삭제(CRUD) 작업을 수행할 수 있습니다.

## 개요

이 스킬은 Claude Code에서 Google Calendar를 프로그래밍 방식으로 제어할 수 있도록 합니다.

**주요 기능:**
- OAuth 2.0 인증 및 토큰 자동 갱신
- 이벤트 CRUD 작업 (생성/조회/수정/삭제)
- 날짜 범위 기반 이벤트 조회
- 종일 이벤트 지원
- API 에러 핸들링

## 사전 요구사항

### Google Cloud Console 설정

1. [Google Cloud Console](https://console.cloud.google.com)에서 프로젝트 생성
2. **APIs & Services**에서 Google Calendar API 활성화
3. OAuth 2.0 자격 증명 생성 (데스크톱 애플리케이션 유형)
4. `credentials.json` 다운로드 후 프로젝트 루트에 배치
5. 승인된 리디렉션 URI에 `http://localhost:8080` 추가

### Python 의존성 설치

```bash
pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
```

## 파일 구조

```
maro-google-callendar-skill/
├── SKILL.md        # 스킬 정의 및 구현 가이드
├── examples.md     # Python 코드 예제
├── reference.md    # API 레퍼런스
└── README.md       # 이 파일
```

## 사용 방법

### 1. 인증 설정

```python
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = ["https://www.googleapis.com/auth/calendar"]

def get_calendar_service():
    creds = None
    if os.path.exists("token.json"):
        creds = Credentials.from_authorized_user_file("token.json", SCOPES)

    if not creds or not creds.valid:
        flow = InstalledAppFlow.from_client_secrets_file("credentials.json", SCOPES)
        creds = flow.run_local_server(port=8080)
        with open("token.json", "w") as token:
            token.write(creds.to_json())

    return build("calendar", "v3", credentials=creds)
```

### 2. 이벤트 조회

```python
# 오늘 이벤트 조회
events = service.events().list(
    calendarId="primary",
    timeMin=start_of_day.isoformat() + "Z",
    timeMax=end_of_day.isoformat() + "Z",
    singleEvents=True,
    orderBy="startTime"
).execute()
```

### 3. 이벤트 생성

```python
event = {
    "summary": "회의",
    "start": {"dateTime": "2026-01-15T14:00:00", "timeZone": "Asia/Seoul"},
    "end": {"dateTime": "2026-01-15T15:00:00", "timeZone": "Asia/Seoul"},
}
service.events().insert(calendarId="primary", body=event).execute()
```

## API 스코프

| 스코프 | 설명 |
|--------|------|
| `calendar` | 전체 캘린더 접근 권한 |
| `calendar.events` | 이벤트 읽기/쓰기 권한 |
| `calendar.readonly` | 읽기 전용 접근 |

## 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `GOOGLE_CALENDAR_CREDENTIALS_PATH` | credentials.json | OAuth 자격 증명 경로 |
| `GOOGLE_CALENDAR_TOKEN_PATH` | token.json | 토큰 저장 경로 |
| `GOOGLE_CALENDAR_DEFAULT_TIMEZONE` | UTC | 기본 시간대 |

## 보안 고려사항

- `token.json`과 `credentials.json`을 `.gitignore`에 추가하세요
- 공개 저장소에 자격 증명 파일을 커밋하지 마세요
- 필요한 최소한의 스코프만 요청하세요

## 참고 자료

- [Google Calendar API 문서](https://developers.google.com/calendar/api)
- [Python Client 라이브러리](https://github.com/googleapis/google-api-python-client)
- [OAuth 2.0 가이드](https://developers.google.com/identity/protocols/oauth2)

## 라이선스

MoAI-ADK Team

---

**버전:** 1.0.0
**최종 업데이트:** 2026-01-13
