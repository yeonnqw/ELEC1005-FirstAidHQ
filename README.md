# Daemon Hunters — Community First Aid Response App
**ELEC1005 Assignment 2 — Lam Vo, Zhehao Lin, Zipeng Song**

## What is this?
A Microsoft Power Apps canvas app that connects community members, volunteers and coordinators to respond to first aid emergencies in real time — backed by Microsoft SharePoint as a live database.

## Why does it exist?
In a community emergency, fast coordination between reporters, volunteers and coordinators saves lives. This app replaces phone calls and manual coordination with a live digital system where:
- Anyone can report an emergency instantly
- Volunteers can see and accept nearby emergencies
- Coordinators can monitor all active incidents and available volunteers in one dashboard

## What can it do?
- Register as a Volunteer or Coordinator with name, email and role
- Log in and be automatically directed to the correct page based on your role
- **As a user (reporter):** Submit an emergency report with type, location and description
- **As a volunteer:** View assigned tasks, see active emergency reports, accept emergencies, mark as declined or resolved
- **As a coordinator:** View all unassigned, in progress and resolved incidents in real time, see available volunteers and their distance to each report

## Power Apps Link
https://make.powerapps.com/e/default-7be93ba7-4482-49d0-a512-7c6818096e33/canvas/?action=edit&app-id=%2Fproviders%2FMicrosoft.PowerApps%2Fapps%2F5e36fcb6-c1e4-487c-8d1b-fa417b66de9a

## SharePoint Site
https://unisydneyedu-my.sharepoint.com/:l:/g/personal/zlin0230_uni_sydney_edu_au/JAB4W-FSs76CRryNGXxc5M8NAVfA5NYHxikl6lxP-DADNto?e=Tqxjx3

https://unisydneyedu-my.sharepoint.com/:l:/g/personal/zlin0230_uni_sydney_edu_au/JABFHNDDNRYJQrad6e5tNcnUAb1_I0VNaFo3_QI0lNCHSEc?e=3IXqeA

## Quick Start
1. Open the Power Apps link above
2. If you are a new user click **Don't have an account? Register here**
3. Enter your Name, Email and select your role (Volunteer or Coordinator)
4. Click **Register** — you will be directed to your role's page automatically
5. If already registered enter your Name and Email and click **Login**

**As a user (reporter):**
- Fill in ReporterName, EmergencyType, Location and Description
- Click **Report Emergency** — the report is saved to SharePoint instantly

**As a volunteer:**
- View your assigned tasks at the top
- See all active emergency reports below
- Click **Accept** to take on an emergency
- Click **Resolved?** when the incident is handled
- Click **Decline** if you cannot respond

**As a coordinator:**
- See counts of Unassigned, In Progress and Resolved incidents at the top
- Browse all active incidents on the left panel
- See available volunteers and their distance to each report on the right panel

## SharePoint Lists
| List | Purpose |
|---|---|
| Volunteer&Coordinator login | Stores all registered users name, email, role, specialty, availability, location coordinates |
| CommunityFirstAid | Stores all emergency reports type, reporter, status, assigned volunteer, location, description |



## App Screens
| Screen | Who uses it | What it does |
|---|---|---|
| login page | Everyone | Enter name and email to log in — redirects based on role |
| Register | New users | Enter name, email and select role to create account |
| user's page | Public reporters | Submit emergency report form to SharePoint |
| volunteer's page | Volunteers | View assigned tasks and active emergencies, accept or resolve |
| coordinator's page | Coordinators | Monitor all incidents and available volunteers in real time |



## Common Confusion
- **"Login doesn't work"**
  → Make sure your name and email exactly match what you registered with — it is case sensitive

- **"I registered but got sent to the wrong page"**
  → Check that you selected the correct role (Volunteer or Coordinator) when registering

- **"No emergencies showing on volunteer page"**
  → Emergencies only appear if their status is Unassigned — check the CommunityFirstAid SharePoint list has data

- **"Available volunteers not showing on coordinator page"**
  → Volunteers only appear as available if their Availability field is set to Available in the SharePoint list

- **"Report Emergency button does nothing"**
  → Make sure all required fields are filled in — ReporterName and EmergencyType are required



## Accessibility
- All buttons have clear text labels — Login, Register, Accept, Decline, Resolved, Report Emergency
- Status counts shown as numbers with labels: Unassigned: 4, In Progress: 3, Resolved: 5
- Role based navigation automatically sends users to the correct screen — no manual navigation needed
- Form fields have clear placeholder labels for every input
- Notify() messages confirm actions when emergencies are submitted or status changes



## Observability — Power Apps Monitor Log

See `PowerAppsTraceEvents.json` in this repository for the full Monitor log.

### Key Observations

**1. SharePoint write (createRow) — 326ms**
Submitting an emergency report triggered a POST request to the CommunityFirstAid SharePoint list. The operation completed successfully with HTTP 201 in 326ms — fast enough for a live emergency reporting system.

**2. Delegation warning on CountRows**
The formula `CountRows(Filter(CommunityFirstAid, Status.Value = "Resolved"))` was not delegated to SharePoint. Power Apps scanned only 500 rows locally. For the current scale of the app this works correctly, but would return inaccurate counts if the list exceeded 500 rows. This could be fixed by using a server delegable formula.

**3. Efficient screen navigation**
5 user actions were recorded including 3 screen navigations. Navigating between screens did not trigger additional SharePoint network calls — confirming that data is loaded efficiently on app start.



## Technical Challenges

**Challenge 1 — Role based navigation**
The app needed to direct users to different screens based on their role after login. This required storing the role in a variable on login and using an If statement to navigate to the correct screen. Early versions navigated everyone to the same screen which required restructuring the login formula.

**Challenge 2 — SharePoint Choice column syntax**
Fields like EmergencyType and Status are Choice columns in SharePoint. Power Apps requires these to be written as `{Value: "text"}` in Patch formulas rather than plain strings. This caused type mismatch errors that were resolved by checking column types in SharePoint List Settings.

**Challenge 3 — Volunteer distance calculation**
The coordinator page shows volunteer distance to each report. This required storing latitude and longitude for both the volunteer and the incident and calculating distance using a formula. When GPS coordinates were unavailable the distance showed as unavailable.



## Reflection

### From Design to Reality
**What worked:**
- Role-based login and navigation to different screens worked correctly
- Emergency report form successfully submitted to SharePoint in real time
- Volunteer page correctly shows assigned tasks and active reports
- Coordinator dashboard shows live counts of incident statuses

**What changed:**
- Originally planned more detailed volunteer filtering — simplified to availability status
- GPS distance feature was added during development as an enhancement
- Register and Login were split into separate screens for clarity

### Engineering Lessons Learned
- Define SharePoint column types before building — Choice columns need different syntax
- Test login and navigation logic first before building content screens
- Role based apps need careful variable management to track current user state
- Keep forms simple — too many required fields increases report abandonment

### Real World Perspective
This project reflects real emergency dispatch software where speed and reliability are critical. The most surprising challenge was how much the data layer (SharePoint column types and formulas) affected the UI layer — a single wrong syntax broke the entire submit flow. In a real system this app would need offline capability and push notifications, which Power Apps currently does not fully support.



## Repository Contents
| File | Description |
|---|---|
| README.md | Project documentation |
| PowerAppsTraceEvents.json | Power Apps Monitor log from live testing |
| REFLECTION.md | Detailed engineering reflection |
| screenshots/ | Screenshots of all 5 app screens |
