# Google Calendar Examples

## Authentication Examples

### Initial Setup and Authorization

```python
import os
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = ["https://www.googleapis.com/auth/calendar"]

def get_calendar_service():
    """Initialize and return authenticated Calendar service."""
    creds = None
    
    if os.path.exists("token.json"):
        creds = Credentials.from_authorized_user_file("token.json", SCOPES)
    
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                "credentials.json", SCOPES
            )
            creds = flow.run_local_server(port=8080)
        
        with open("token.json", "w") as token:
            token.write(creds.to_json())
    
    return build("calendar", "v3", credentials=creds)
```

---

## Read Examples

### List Today Events

```python
from datetime import datetime, timedelta

def get_today_events(service):
    """Get all events for today."""
    now = datetime.utcnow()
    start_of_day = now.replace(hour=0, minute=0, second=0, microsecond=0)
    end_of_day = start_of_day + timedelta(days=1)
    
    events_result = service.events().list(
        calendarId="primary",
        timeMin=start_of_day.isoformat() + "Z",
        timeMax=end_of_day.isoformat() + "Z",
        maxResults=50,
        singleEvents=True,
        orderBy="startTime"
    ).execute()
    
    return events_result.get("items", [])
```

### List Events in Date Range

```python
def get_events_in_range(service, start_date, end_date):
    """Get events between two dates."""
    events_result = service.events().list(
        calendarId="primary",
        timeMin=start_date.isoformat() + "Z",
        timeMax=end_date.isoformat() + "Z",
        maxResults=100,
        singleEvents=True,
        orderBy="startTime"
    ).execute()
    
    return events_result.get("items", [])
```

### Get Single Event

```python
def get_event(service, event_id, calendar_id="primary"):
    """Get a specific event by ID."""
    return service.events().get(
        calendarId=calendar_id,
        eventId=event_id
    ).execute()
```

---

## Create Examples

### Create Simple Event

```python
def create_event(service, summary, start_time, end_time, 
                 description=None, location=None, timezone="Asia/Seoul"):
    """Create a new calendar event."""
    event = {
        "summary": summary,
        "start": {"dateTime": start_time.isoformat(), "timeZone": timezone},
        "end": {"dateTime": end_time.isoformat(), "timeZone": timezone},
    }
    
    if description:
        event["description"] = description
    if location:
        event["location"] = location
    
    return service.events().insert(
        calendarId="primary",
        body=event
    ).execute()
```

### Create All-Day Event

```python
def create_all_day_event(service, summary, date, description=None):
    """Create an all-day event."""
    event = {
        "summary": summary,
        "start": {"date": date.strftime("%Y-%m-%d")},
        "end": {"date": (date + timedelta(days=1)).strftime("%Y-%m-%d")},
    }
    
    if description:
        event["description"] = description
    
    return service.events().insert(calendarId="primary", body=event).execute()
```

---

## Update Examples

### Update Event

```python
def update_event(service, event_id, **updates):
    """Update an existing event."""
    event = service.events().get(calendarId="primary", eventId=event_id).execute()
    
    for key in ["summary", "description", "location"]:
        if key in updates:
            event[key] = updates[key]
    
    return service.events().update(
        calendarId="primary", eventId=event_id, body=event
    ).execute()
```

### Partial Update (Patch)

```python
def patch_event(service, event_id, **updates):
    """Partially update an event."""
    body = {k: v for k, v in updates.items() if k in ["summary", "description", "location"]}
    
    return service.events().patch(
        calendarId="primary", eventId=event_id, body=body
    ).execute()
```

---

## Delete Examples

### Delete Event

```python
def delete_event(service, event_id, calendar_id="primary"):
    """Delete an event by ID."""
    service.events().delete(calendarId=calendar_id, eventId=event_id).execute()
```

---

## Error Handling

### Safe API Call Decorator

```python
from googleapiclient.errors import HttpError

def safe_api_call(func):
    """Decorator for handling Calendar API errors."""
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except HttpError as e:
            status = e.resp.status
            messages = {
                401: "Authentication error. Please re-authorize.",
                403: "Permission denied.",
                404: "Event not found.",
                429: "Rate limit exceeded."
            }
            print(messages.get(status, f"API error: {e}"))
            return None
    return wrapper
```

---

## Complete Example: CalendarManager Class

```python
from datetime import datetime, timedelta

class CalendarManager:
    """Complete calendar management wrapper."""
    
    def __init__(self, service):
        self.service = service
        self.calendar_id = "primary"
    
    def list_events(self, days=7):
        now = datetime.utcnow()
        end = now + timedelta(days=days)
        
        result = self.service.events().list(
            calendarId=self.calendar_id,
            timeMin=now.isoformat() + "Z",
            timeMax=end.isoformat() + "Z",
            maxResults=50,
            singleEvents=True,
            orderBy="startTime"
        ).execute()
        
        return result.get("items", [])
    
    def create(self, summary, start, end, **kwargs):
        event = {
            "summary": summary,
            "start": {"dateTime": start.isoformat(), "timeZone": "Asia/Seoul"},
            "end": {"dateTime": end.isoformat(), "timeZone": "Asia/Seoul"},
        }
        event.update(kwargs)
        return self.service.events().insert(calendarId=self.calendar_id, body=event).execute()
    
    def update(self, event_id, **updates):
        event = self.service.events().get(calendarId=self.calendar_id, eventId=event_id).execute()
        event.update(updates)
        return self.service.events().update(calendarId=self.calendar_id, eventId=event_id, body=event).execute()
    
    def delete(self, event_id):
        self.service.events().delete(calendarId=self.calendar_id, eventId=event_id).execute()
```
