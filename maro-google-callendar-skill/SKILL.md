---
name: "google-calendar"
description: "Google Calendar integration skill for event management including create, read, update, and delete operations with OAuth 2.0 authentication"
version: 1.0.0
category: "integration"
modularized: false
user-invocable: true
tags: ['google', 'calendar', 'events', 'scheduling', 'oauth']
updated: 2026-01-13
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - WebFetch
status: "active"
author: "MoAI-ADK Team"
---

# Google Calendar Integration Skill

## Quick Reference

Google Calendar API Integration - Comprehensive patterns for managing calendar events including authentication, CRUD operations, and error handling.

Core Capabilities:

- Authentication: OAuth 2.0 flow with token refresh
- Event Management: Create, read, update, delete calendar events
- Query Events: List events with date range filters
- Error Handling: Graceful API error management

When to Use:

- Managing Google Calendar events programmatically
- Scheduling automation workflows
- Calendar data synchronization
- Event reminder integrations

---

## Prerequisites

### Google Cloud Console Setup

Before using this skill, configure Google Cloud Console:

1. Create a project at https://console.cloud.google.com
2. Enable the Google Calendar API in APIs and Services
3. Create OAuth 2.0 credentials (Desktop application type)
4. Download credentials.json and place in project root
5. Add authorized redirect URIs including http://localhost:8080

Required Scopes:

- https://www.googleapis.com/auth/calendar (full access)
- https://www.googleapis.com/auth/calendar.events (events only)
- https://www.googleapis.com/auth/calendar.readonly (read-only)

---

## Implementation Guide

### Authentication Setup

OAuth 2.0 Token Management:

Create a GoogleCalendarAuth class that handles OAuth 2.0 authentication. Initialize with credentials_path pointing to credentials.json and token_path for storing tokens. The SCOPES list should include calendar API scope. Implement get_credentials method that loads existing token from file, refreshes if expired using google.auth.transport.requests.Request, or initiates new OAuth flow using InstalledAppFlow if no valid token exists. Save updated tokens after refresh or new authorization.

Token Refresh Logic:

The get_credentials method should check if token.json exists and load Credentials from file. If credentials are invalid but have a refresh_token, call credentials.refresh. If no valid credentials exist, run InstalledAppFlow.from_client_secrets_file with local server on port 8080. Always save credentials to token.json after obtaining them.

### Calendar Service Initialization

Service Builder:

Create a CalendarService class that wraps the Google Calendar API. Initialize by calling GoogleCalendarAuth.get_credentials to obtain valid credentials. Build the service object using googleapiclient.discovery.build with serviceName calendar, version v3, and credentials parameter. Store the service instance for subsequent API calls.

---

## Event Operations

### List Events (Read)

Query Events by Date Range:

Implement list_events method that accepts calendar_id (default primary), time_min, time_max, and max_results parameters. Convert datetime objects to RFC3339 format with isoformat and Z suffix for UTC. Call service.events.list with calendarId, timeMin, timeMax, maxResults, singleEvents True, and orderBy startTime. Execute the request and return items list from response.

Today's Events Query:

Create get_today_events helper that calculates today's start as midnight and end as 23:59:59. Pass these bounds to list_events with calendar_id primary.

### Create Event (Create)

Basic Event Creation:

Implement create_event method accepting summary, start_datetime, end_datetime, and optional description and location. Build event body dictionary with summary, description, location, and start/end objects containing dateTime in ISO format and timeZone. Call service.events.insert with calendarId primary and body parameter. Return the created event response.

All-Day Event Creation:

For all-day events, use date key instead of dateTime in start and end objects. Format should be YYYY-MM-DD string without time component.

### Update Event (Update)

Event Modification:

Implement update_event method accepting event_id and keyword arguments for fields to update. First retrieve existing event using service.events.get with calendarId and eventId. Merge updates into event dictionary. Call service.events.update with calendarId, eventId, and updated body. Return the modified event response.

Partial Update:

For partial updates, use service.events.patch instead of update. This only modifies specified fields without requiring the full event body.

### Delete Event (Delete)

Event Deletion:

Implement delete_event method accepting calendar_id and event_id. Call service.events.delete with calendarId and eventId parameters. Execute the request. The method returns None on success. Handle HttpError for non-existent events gracefully.

---

## Error Handling

### Common Error Patterns

API Error Handler:

Create a decorator handle_calendar_errors that wraps API calls in try-except block. Catch googleapiclient.errors.HttpError and extract status_code and reason from response. Map common status codes to user-friendly messages: 401 for authentication errors requiring re-authorization, 403 for permission denied, 404 for event not found, 429 for rate limiting. Re-raise with CalendarAPIError containing context.

Token Expiration Handling:

On 401 errors, check if refresh_token exists. If so, attempt automatic token refresh and retry the operation once. If refresh fails or no refresh_token, prompt user to re-authorize by deleting token.json and running auth flow again.

---

## Usage Patterns

### Daily Workflow Integration

Morning Briefing:

Create a morning_briefing function that lists today's events and formats them as a readable summary. Include event time, title, and location. Sort by start time and present in chronological order.

### Event Creation Workflow

Quick Event Creation:

Implement quick_create method accepting natural language string. Parse using dateutil.parser or similar library to extract date, time, and description. Map parsed components to create_event parameters.

---

## Configuration

### Environment Variables

Required Configuration:

- GOOGLE_CALENDAR_CREDENTIALS_PATH: Path to credentials.json file
- GOOGLE_CALENDAR_TOKEN_PATH: Path for token storage (default: token.json)
- GOOGLE_CALENDAR_DEFAULT_TIMEZONE: Default timezone (default: UTC)

### Credentials File Structure

The credentials.json file from Google Cloud Console contains installed object with client_id, project_id, auth_uri, token_uri, auth_provider_x509_cert_url, client_secret, and redirect_uris array.

---

## Security Considerations

### Token Storage

Store tokens securely outside of version control. Add token.json to .gitignore. Consider encrypting token files at rest for production deployments. Never commit credentials.json to public repositories.

### Scope Minimization

Request only necessary scopes. Use calendar.events scope instead of full calendar scope when only event operations are needed. Use calendar.readonly for read-only integrations.

---

## Dependencies

Required Python Packages:

- google-auth >= 2.0.0
- google-auth-oauthlib >= 1.0.0
- google-auth-httplib2 >= 0.1.0
- google-api-python-client >= 2.0.0

Install Command:

pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
