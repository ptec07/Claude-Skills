# Google Calendar API Reference

## Authentication

### OAuth 2.0 Scopes

| Scope | Description | Use Case |
|-------|-------------|----------|
| calendar | Full access to all calendars | Complete calendar management |
| calendar.events | Read/write access to events | Event CRUD without calendar settings |
| calendar.readonly | Read-only access | Display and reporting only |

### Token Management

| File | Purpose |
|------|---------|
| credentials.json | OAuth client credentials from Google Cloud Console |
| token.json | User access and refresh tokens (auto-generated) |

---

## Events API

### events.list

List events from a calendar.

Parameters:
- calendarId (string, required): Calendar identifier
- timeMin (datetime): Lower bound (RFC3339 format)
- timeMax (datetime): Upper bound (RFC3339 format)
- maxResults (integer): Maximum entries (default: 250)
- singleEvents (boolean): Expand recurring events
- orderBy (string): startTime or updated

### events.get

Get a single event.

Parameters:
- calendarId (string, required): Calendar identifier
- eventId (string, required): Event identifier

### events.insert

Create a new event.

Parameters:
- calendarId (string, required): Calendar identifier
- body (object, required): Event resource

Event Resource Fields:
- summary (string, required): Event title
- start (object, required): Start time
- end (object, required): End time
- description (string): Event description
- location (string): Event location

### events.update

Update an existing event (full replacement).

Parameters:
- calendarId (string, required): Calendar identifier
- eventId (string, required): Event identifier
- body (object, required): Complete event resource

### events.patch

Partially update an event.

Parameters:
- calendarId (string, required): Calendar identifier
- eventId (string, required): Event identifier
- body (object, required): Fields to update

### events.delete

Delete an event.

Parameters:
- calendarId (string, required): Calendar identifier
- eventId (string, required): Event identifier

---

## Error Codes

| Status | Description | Action |
|--------|-------------|--------|
| 400 | Bad Request | Check request format |
| 401 | Unauthorized | Refresh or re-authenticate |
| 403 | Forbidden | Request additional scopes |
| 404 | Not Found | Verify IDs |
| 429 | Rate Limited | Implement backoff |

---

## Rate Limits

- Queries per day: 1,000,000
- Queries per 100 seconds per user: 500

---

## Date/Time Formats

RFC3339 Examples:
- DateTime with timezone: 2026-01-15T14:00:00+09:00
- DateTime UTC: 2026-01-15T05:00:00Z
- Date only: 2026-01-15

Common Timezones:
- Asia/Seoul (UTC+9)
- America/New_York (EST/EDT)
- Europe/London (GMT/BST)
- UTC

---

## Dependencies

Required Packages:
- google-auth >= 2.0.0
- google-auth-oauthlib >= 1.0.0
- google-auth-httplib2 >= 0.1.0
- google-api-python-client >= 2.0.0

Installation:
pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| GOOGLE_CALENDAR_CREDENTIALS_PATH | credentials.json | OAuth credentials |
| GOOGLE_CALENDAR_TOKEN_PATH | token.json | Token storage |
| GOOGLE_CALENDAR_DEFAULT_TIMEZONE | UTC | Default timezone |

---

## Resources

- API Documentation: https://developers.google.com/calendar/api
- Python Client: https://github.com/googleapis/google-api-python-client
- OAuth 2.0 Guide: https://developers.google.com/identity/protocols/oauth2
