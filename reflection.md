# Reflection — Daemon Hunters
**ELEC1005 Assignment 2**
**Team: Lam Vo, Zhehao Lin, Zipeng Song**

## 1. From Design to Reality

### What worked as expected
The core concept of the app translated well from design to implementation. 
Role-based navigation worked correctly after login, the app automatically 
directed users to the correct screen based on their role (Volunteer or 
Coordinator). The emergency report form submitted data to SharePoint reliably, 
and the coordinator dashboard displayed live incident counts and available 
volunteer information as intended.

The two-list SharePoint structure (Volunteer&Coordinator login and 
CommunityFirstAid) proved to be a clean and effective data model. Separating 
user data from incident data made queries simpler and kept the app fast.

### What didn't work as expected
The original design assumed that all form fields would be plain text. In 
reality, SharePoint Choice columns (EmergencyType, Status, Availability) 
required a completely different syntax in Power Apps `{Value: "text"}` 
instead of plain strings. This caused type mismatch errors across multiple 
screens and took significant time to debug.

GPS location capture worked inconsistently. On some devices the coordinates 
were captured correctly, but on others the latitude and longitude showed as 
blank, which caused the volunteer distance calculation on the coordinator page 
to display "unavailable".

### Changes made from original design
- The Register screen was originally combined with the Login screen but was 
  separated into two screens for clarity and better user flow
- Volunteer filtering was simplified originally planned to filter by 
  specialty and location, but was simplified to filter by availability status
- A GPS location feature was added during development as an enhancement that 
  was not in the original wireframes
- Zipeng Song was removed from active development responsibilities partway 
  through the project, requiring Lam Vo and Zhehao Lin to redistribute tasks

## 2. Technical Challenges

### Challenge 1: SharePoint Choice column syntax
**Problem:** Power Apps Patch formulas require Choice columns to use record 
syntax `{Value: "text"}` rather than plain strings. Fields like EmergencyType, 
Status and Availability are all Choice columns in SharePoint. Initially all 
Patch formulas used plain strings which caused "type does not match expected 
type Record" errors across the report form and status update buttons.

**Resolution:** Checked each column type in SharePoint List Settings and 
identified all Choice columns. Updated every Patch formula to wrap Choice 
values in `{Value: "..."}` syntax. Also discovered that reading Choice column 
values requires `.Value` suffix — for example `ThisItem.Status.Value` instead 
of `ThisItem.Status`.

**Lesson learned:** Always check SharePoint column types before writing Power 
Apps formulas. Plain text columns and Choice columns look identical in the 
SharePoint list view but behave completely differently in formulas.

### Challenge 2: Role-based navigation after login
**Problem:** The app needed to send users to different screens based on their 
role after logging in. Early versions used a simple Navigate formula that sent 
everyone to the same screen. The challenge was reading the user's role from 
SharePoint and making a navigation decision before the screen loaded.

**Resolution:** Used a LookUp formula to find the current user in the 
Volunteer&Coordinator login list by matching their name and email, then stored 
their role in a variable called `currentRole`. The login button OnSelect 
formula then used an If statement to navigate to the correct screen based on 
that variable.

**Lesson learned:** Variables must be set before navigation happens. The order 
of operations in Power Apps OnSelect formulas matters — Set must come before 
Navigate.

### Challenge 3: GPS coordinates and distance calculation
**Problem:** The coordinator page was designed to show volunteer distance to 
each emergency report. This required GPS coordinates for both the volunteer 
and the incident location. On many devices `Location.Latitude` and 
`Location.Longitude` returned blank values because the browser did not have 
location permissions enabled.

**Resolution:** Added a fallback message "Current distance to Report: 
unavailable" when coordinates were blank. Volunteers were also given the 
option to manually enter their location during registration.

**Lesson learned:** GPS and device permissions cannot be assumed in a web app. 
Always build fallback states for features that depend on device hardware.

## 3. Testing and Observability

### What testing revealed
Testing the login flow revealed that the name and email matching was 
case-sensitive, which meant users who entered their name slightly differently 
than they registered could not log in. This was an unexpected usability issue 
that required communicating exact registration details to users.

Testing the report form showed that submitting without an EmergencyType 
selected caused a silent failure — the form appeared to submit but no row was 
created in SharePoint. This was fixed by adding a validation check before 
the Patch formula.

### What Monitor showed
The Power Apps Monitor log (PowerAppsTraceEvents.json) revealed:

- **createRow completed in 326ms** the emergency report submission to 
  SharePoint was fast and returned HTTP 201 successfully
- **Delegation warning on CountRows** the formula 
  `CountRows(Filter(CommunityFirstAid, Status.Value = "Resolved"))` was not 
  delegated to SharePoint, meaning only 500 rows were scanned locally. For 
  the current scale of the app this works correctly but would fail on larger 
  datasets
- **Navigation was efficient** moving between screens did not trigger 
  additional SharePoint network calls, confirming that data loaded on app 
  start was reused correctly

### Bugs found during testing
- Silent form submission failure when EmergencyType was not selected — fixed 
  with validation
- Coordinator page showed incorrect volunteer count when the same volunteer 
  appeared multiple times in the list — caused by duplicate registrations 
  during testing
- Status badge colours did not update immediately after a volunteer clicked 
  Accept — required a Refresh call to update the gallery

## 4. Engineering Lessons Learned

**Plan data types before building**
The most time-consuming errors came from not planning SharePoint column types 
before writing Power Apps formulas. In future projects, creating a data 
dictionary that specifies each column name, type and accepted values before 
writing any formula would save significant debugging time.

**Test the data layer first**
The login and navigation logic should have been tested before building the 
content screens. Because content screens were built assuming login worked 
correctly, bugs in the login formula required going back and fixing multiple 
screens.

**Keep forms simple**
The emergency report form originally had more fields including a priority 
rating and an optional image upload. These were removed because they added 
complexity without clear benefit. Simpler forms have higher completion rates 
and fewer validation errors.

**Communication matters as much as code**
When a team member became inactive partway through the project, tasks had to 
be redistributed without a clear handover. In future projects, documenting 
what each person is working on in Jira from the start would make handovers 
much smoother.

## 5. Real World Perspective

### How this reflects real software engineering
This project reflected several aspects of real software engineering practice. 
Integration between two platforms — Power Apps and SharePoint — introduced 
compatibility issues that required careful reading of documentation and 
systematic debugging. This mirrors the challenges of API integration in 
professional projects, where two systems that are designed to work together 
still require significant effort to connect correctly.

The role based access control feature (different screens for volunteers vs 
coordinators) reflects a common pattern in real enterprise software where 
different user types have different permissions and views of the same data.

The Power Apps Monitor tool provided insight into network calls and 
performance that parallels the observability tools used in professional 
systems such as application performance monitoring (APM) dashboards.

### What surprised us most
The most surprising aspect was how much a single syntax difference could break 
an entire feature. The difference between `Status = "Resolved"` and 
`Status.Value = "Resolved"` looks trivial but caused complete failure of the 
coordinator dashboard filters. In a real emergency response system, this kind 
of silent failure could have serious consequences — highlighting how important 
thorough testing and validation are even in small-scale applications.

The delegation warning from Monitor was also surprising. The app appeared to 
work correctly during testing, but Monitor revealed that it was only scanning 
500 rows locally rather than querying the full dataset. This kind of hidden 
limitation is easy to miss without observability tools and could cause 
incorrect results in production.
