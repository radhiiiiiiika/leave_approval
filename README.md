# Leave Approval (Google Forms + Google Sheets + Apps Script)
> A low-code, multi-level leave request and approval system built using Google Forms, Google Sheets, Gmail, Google Calendar, and Google Apps Script.
> This was built to practice designing an approval hierarchy without spinning up a full backend server. Instead of managing leave requests over messy email threads, everything runs off a Google Form, a config tab in Sheets, and a small Apps Script Web App.

## What It Does
> Department-based approval hierarchy: An Approval Hierarchy tab maps each department (Engineering, Marketing, Operations, Sales, R&D) to a Level 1 Manager, a Level 2 Director, and a day threshold. Updating approvers or thresholds only requires editing the Sheet—no code changes or redeploys.
> Conditional Level 2 escalation: Short leaves (at or below the department's threshold) finish after Level 1. Longer leaves automatically generate a fresh token, send an intermediate status email to the employee (Level 1 Approved — Pending Level 2), and route the request to the Level 2 Director.
> Annual leave balance check: Each approval email and confirmation screen calculates how many approved days that employee email has already used in the calendar year out of a 15-day allowance, and shows a warning banner if approving the request would exceed it.
> Approve & reject confirmation screens: Clicking Approve opens a quick confirmation page with an optional note box; clicking Reject requires a reason. Notes from both L1 and L2 are saved to Decision Comment and included in the employee's email.
> Automatic Google Calendar OOO events: Once a request gets final approval, CalendarApp adds an all-day OOO: <Name> (<Leave Type>) event spanning the approved dates.
48-hour pending reminders: A daily trigger (sendPendingReminders) checks for any Pending L1 or Pending L2 rows untouched for over 48 hours and nudges the current approver (throttled to once every 24 hours).
> Live Sheets dashboard: A Dashboard tab tracks total requests, status counts, escalations, average duration, approved days by department, and a status breakdown chart using built-in formulas.

## Things I Paid Attention To While Building
> Single-use tokens & locking: Every approval link carries a random UUID (Utilities.getUuid()) stored in the Sheet. Once acted on, the token is cleared (or rotated for L2), so old email links show an "Already Processed" page. LockService.getScriptLock() wraps doGet() so two quick clicks don't overwrite each other.
> Two-step confirmation instead of instant GET actions: Making both Approve and Reject open a small confirmation form first prevents accidental clicks (and automated email link scanners) from mutating statuses.
> Google Calendar exclusive end dates: CalendarApp.createAllDayEvent() treats the end date as exclusive, so the script adds + 1 day to the form's End Date to make multi-day leaves span the full inclusive range.
> Testing multi-role flows on one Gmail account: Used Gmail + aliases (+employee, +manager, +director) to test all three roles end-to-end without needing a paid Workspace org.
Spreadsheet Structure

## The Google Sheet (Leave Approval Tracker) has 3 tabs:
### Form responses 1
> Columns A–H (from Google Form): Timestamp, Employee Name, Employee Email, Department, Leave Type, Start Date, End Date, Reason for leave
> Columns I–T (managed by Apps Script): Request ID, Days, L1 Approver, L1 Status, L2 Approver, L2 Status, Final Status, Action Token, Last Updated, Decision Comment, Calendar Status, Last Reminded
### Approval Hierarchy
> Columns A–D: Department, L1 Manager Email, L2 Director Email, L2 Threshold (Days)
### Dashboard
> Formula-driven KPI counts (COUNTA, COUNTIF, AVERAGE), department summary (SUMIFS), and a Doughnut chart of request statuses.
