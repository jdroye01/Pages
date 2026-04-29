# ChurchScore Detailed How-To Guide

## What This Guide Covers

This guide documents the current ChurchScore application behavior based on the code in the following app areas:

- Members
- Reports
- Points
- Settings
- Excel and CSV imports
- Tier tracking
- Member profile photos
- Live sync and offline queueing

It is intentionally detailed and also calls out current workflow quirks where the code behaves differently from what a user might expect.

## Application Overview

ChurchScore is a SwiftUI iOS application for tracking church members, assigning them to classes, awarding points for activities, viewing individual member progress, importing data, and generating reports. The main app opens to a splash screen and then shows four tabs:

1. `Members`
2. `Reports`
3. `Points`
4. `Settings`

On launch, the app also attempts to:

- connect to the configured sync server
- perform a full sync pull from the server

## Main Concepts

Before using the app, it helps to understand the five data types it manages.

### 1. Classes

Classes are group labels assigned to members, such as:

- Youth
- Adults
- Men
- Women
- Choir

Classes are used in:

- the member roster filter
- the point-awarding screen
- class summary reporting
- member detail view

### 2. Members

Each member record contains:

- a unique internal ID
- member name
- class name
- optional profile photo saved on device

Members appear in:

- the roster
- report filters
- point-awarding grids
- individual member detail pages

### 3. Actions

Actions are the activities that earn points, for example:

- Attendance
- Bringing a Bible
- Inviting a guest
- Memory verse completion

Each action has:

- a unique internal ID
- action name
- point value

### 4. Point Records

A point record is created when points are awarded. Each point record stores:

- member ID
- action ID
- total points for that action
- date awarded

Point records drive:

- lifetime point totals
- last-month attendance counts
- reports
- class summaries

### 5. Tiers

Tiers are milestone labels based on total points, such as:

- Bronze
- Silver
- Gold

Each tier has:

- a name
- a point threshold

The app shows the highest tier whose threshold is less than or equal to a member's lifetime points.

## Launch and Startup Behavior

When the app starts:

1. A splash screen appears for about 2.5 seconds.
2. The app tries to connect to the sync server.
3. The app triggers a full sync pull from the server if a server URL is configured.
4. The main tab view opens.

This means a synced installation may refresh its local database shortly after launch.

## Tab 1: Members

The `Members` tab is the roster view.

### What You See

At the top:

- a class picker

Below it:

- a list of members in the selected class
- each member's current lifetime point total

### How to Filter the Roster

1. Open the `Members` tab.
2. Tap the class picker.
3. Choose a class.
4. The list refreshes to show only members in that class.

### What the Roster Shows

Each row displays:

- member name
- current lifetime points, shown as `X pts`

### Important Behavior Notes

- The roster filters by one exact class at a time.
- The roster does not include an `All Classes` option in the current implementation.
- Lifetime points are calculated by summing all point records for that member.

## Member Detail Screen

Tap any member in the roster to open that member's detail page.

### What the Detail Screen Shows

The member detail screen includes:

- member name
- current tier
- profile image
- current class
- class change control
- last month attendance
- lifetime points

### How Tier Is Determined

The app:

1. loads all tiers
2. sorts them from highest threshold to lowest
3. picks the first tier the member qualifies for

If no tier threshold is met, the display remains `None`.

### How Last Month Attendance Is Calculated

Attendance is based on point records from the last month. The code counts unique days with point entries, not the number of point records. If a member receives multiple awards on the same day, that still counts as one attendance day.

### How to Change a Member's Class

1. Open the member detail page.
2. Tap the class change menu.
3. Select a new class.
4. The app updates the member's class.

This change is written locally and, in synced workflows, also emitted as a sync event.

### How to Add or Change a Member Photo

1. Open the member detail page.
2. Tap the circular profile image.
3. Choose or edit an image in the image picker.
4. Close the picker.
5. The app saves the chosen image as a JPEG in the app's document storage.

Notes:

- The app uses `UIImagePickerController`.
- Editing is enabled in the picker.
- If a member has no saved photo, the app falls back to a default image.

## Tab 2: Reports

The `Reports` tab provides two reporting modes:

1. quick template reports
2. a custom report builder

### Quick Template Reports

Available quick templates are:

- Monthly
- Q1
- Q2
- Q3
- Q4

### How Quick Reports Work

1. Open the `Reports` tab.
2. Select a template.
3. Tap `Quick Generate Report`.
4. The app builds a text report grouped by member.

### Template Date Ranges

The app calculates ranges from the current calendar year:

- `Q1`: January 1 through March 31
- `Q2`: April 1 through June 30
- `Q3`: July 1 through September 30
- `Q4`: October 1 through December 31
- `Monthly`: one month ago through today

### What a Generated Report Contains

The report groups point records by member and shows:

- member total points for the filtered period
- dated activity lines under that member

The output format is plain text. Example structure:

```text
Jane Smith - 45 pts
  Apr 10, 2026 - Attendance: 10
  Apr 17, 2026 - Bible Verse: 15
```

### Important Report Limitation

The report only shows action names when the action ID in the point record matches a real action in the actions table. Imported Excel historical records may not always preserve a usable action link, which can cause report entries to appear without the original action label in some workflows.

## Quick Class Summary Report

There is a separate button called `Quick Generate Class Summary`.

### What It Does

The app:

1. loads all point records
2. finds each related member
3. totals points by class
4. sorts classes from highest points to lowest

The result is a text summary like:

```text
Class Summary Report

Youth – 820 pts
Adults – 675 pts
```

## Custom Report Builder

The custom report builder lets you choose:

- export format
- start date
- end date
- selected members
- selected actions

### Export Formats

Available export formats are:

- PDF
- CSV

### How to Create a Custom Report

1. Open the `Reports` tab.
2. In `Custom Report Builder`, choose `PDF` or `CSV`.
3. Set a start date.
4. Set an end date.
5. Tap member chips to include only specific members, or leave all unselected for all members.
6. Tap action chips to include only specific actions, or leave all unselected for all actions.
7. Tap `Generate Custom Report`.

### What Happens Next

The app:

1. applies the date filters
2. applies optional member filters
3. applies optional action filters
4. generates report text
5. writes the export to a temporary file
6. opens the iOS share sheet

### Important Export Note

The current CSV export writes the report text to a `.csv` file. It is not a structured row-by-row spreadsheet export. In other words, the file extension is CSV, but the content is the same text report shown on screen.

## Tab 3: Points

The `Points` tab is used to award actions to multiple members.

### What You See

The screen includes:

- a class picker
- a grid of members
- a column for each action
- toggles used like checkboxes
- a `Submit` button

### How to Award Points

1. Open the `Points` tab.
2. Choose a class.
3. Find the member row.
4. Toggle on each action that applies.
5. Repeat for other members.
6. Tap `Submit`.

### What Submit Does

For every checked box, the app:

1. finds the matching member
2. finds the matching action
3. inserts a point record using the action's point value
4. stamps the current date and time

After submission, the temporary checkbox state is cleared.

### Class Handling on the Points Screen

Unlike the roster, the points screen adds an `All Classes` option automatically if classes exist. That allows awarding points across the full member list.

### Important Points Workflow Notes

- Every checked action creates a separate point record.
- The screen does not currently show a confirmation prompt before submit.
- The screen does not prevent awarding the same action repeatedly across multiple submissions.

## Tab 4: Settings

The `Settings` tab is the administrative control center.

It includes these sections:

- Live Sync
- Classes
- Members
- Actions
- Tiers
- Import

## Settings: Live Sync

The sync status component displays:

- connection state indicator
- queued offline change count
- last sync time
- current server URL
- connect button when disconnected
- pull full sync button when connected

### Connection States

The app uses four states:

- `Connected`
- `Connecting`
- `Offline — changes queued`
- `Disconnected`

### What the Status Colors Mean

- Green: connected
- Yellow: connecting
- Orange: offline with queued work
- Red: disconnected

### How to Configure Sync

1. Open `Settings`.
2. In `Live Sync`, tap `Edit` next to the server URL.
3. Enter the WebSocket server URL, for example `ws://100.x.x.x:8080`.
4. Tap `Save`.

The app then:

- stores the server URL in `UserDefaults`
- disconnects any existing socket
- reconnects using the new address

### How Offline Queueing Works

If the app is not connected when a synced action occurs, the sync event is stored locally in `UserDefaults`. The queued count appears in the sync status area. When the app reconnects, it automatically flushes queued events to the server.

### How Full Sync Works

When you tap `Pull Full Sync from Server`, the app requests `/sync/all` from the configured server and replaces local classes, members, actions, point records, and tiers with the server payload.

Important caution:

- Full sync is a replace operation, not a merge.
- Local database tables are cleared and repopulated from the server response.

## Settings: Classes

The `Classes` section lets you:

- view all classes
- add a class
- delete a class

### How to Add a Class

1. Go to `Settings`.
2. In `Classes`, enter a new class name.
3. Tap `Add Class`.

The app:

- adds the class locally
- refreshes the UI
- selects the new class for member creation
- emits a sync event in synced mode

### How to Delete a Class

1. Go to `Settings`.
2. Use the list's delete controls.
3. Delete the chosen class.

Notes:

- Duplicate class names are prevented at the database level.
- The code does not automatically reassign or clean up members already using a deleted class name.

## Settings: Members

The `Members` section lets you:

- view all members
- add a member
- delete a member

### How to Add a Member

1. Go to `Settings`.
2. Enter the member name.
3. Select the member's class.
4. Tap `Add Member`.

Requirements:

- the member name cannot be blank
- a class must be selected

### How to Delete a Member

1. Open the `Members` list in `Settings`.
2. Use the delete control for the row.

Notes:

- The current code does not prevent duplicate member names.
- Deleting a member removes the member record, but there is no separate UI prompt shown in code before deletion.

## Settings: Actions

The `Actions` section lets you:

- view all actions
- add a new action
- delete an action

### How to Add an Action

1. Go to `Settings`.
2. Enter the action name.
3. Enter the point value as a whole number.
4. Tap `Add Action`.

### How to Delete an Action

1. Open the `Actions` list.
2. Use the delete control for the action row.

Best practice:

- Keep action names consistent because reports and imports depend on them conceptually, even if the current Excel history importer does not fully preserve action links.

## Settings: Tiers

The `Tiers` section lets you:

- view all tiers
- add a tier
- delete a tier

### How to Add a Tier

1. Go to `Settings`.
2. Enter the tier name.
3. Enter the minimum point threshold.
4. Tap `Add Tier`.

### How to Delete a Tier

1. Open the `Tiers` list.
2. Use the delete control for the tier row.

Best practice:

- Create tiers from lowest threshold to highest for easier administrative review.
- The app itself sorts thresholds internally when deciding which tier a member has earned.

## Settings: Import

The import area supports:

- `.csv`
- `.xlsx`

The import logic uses file structure to decide what to load.

## CSV Import Rules

### CSV Members Import

If the header row contains:

- `name`
- `class`

the app treats the file as a member import.

Each valid data row:

- adds the class if needed
- adds the member

Recommended columns:

```text
name,class
Jane Smith,Youth
Michael Brown,Adults
```

### CSV Actions Import

If the header row contains:

- `name`
- `points`

the app treats the file as an action import.

Recommended columns:

```text
name,points
Attendance,10
Bible Verse,15
```

### CSV Tiers Import

If the header row contains:

- `name`
- `threshold`

the app treats the file as a tier import.

Recommended columns:

```text
name,threshold
Bronze,100
Silver,250
Gold,500
```

## Excel Import Rules

Excel import is sheet-based. The app loops through workbook sheets and looks at each sheet name.

### Excel Members Sheet

If the sheet name contains `member`, and the header row contains all of these columns:

- `name`
- `class`
- `action`
- `points`
- `date`

the app imports both members and historical point records.

Expected layout:

```text
Name | Class | Action | Points | Date
```

Supported date parsing:

- ISO 8601 date strings
- `MM/dd/yyyy`

### Excel Actions Sheet

If the sheet name contains `action`, and the header row contains:

- `name`
- `points`

the app imports actions.

### Excel Tiers Sheet

If the sheet name contains `tier`, and the header row contains:

- `name`
- `threshold`

the app imports tiers.

## Critical Import Caveats

These are important enough to treat as mandatory reading.

### 1. Excel Member Import Is Really a Member + History Import

There is no separate simple Excel member-only sheet in the current code. If you use Excel for members, the app expects action history columns too.

### 2. Repeated Members Can Create Duplicates

For qualifying member-sheet rows, the importer calls `addMember` for every row. If the same person appears on multiple rows, the app can create multiple member entries with the same name.

### 3. Imported Excel Historical Actions Are Not Fully Linked to Real Actions

The Excel member-history importer stores imported point records using a newly generated action ID instead of matching an existing action record by name. That means later reports may not always resolve the original action name correctly for those imported historical entries.

### 4. Unrecognized Sheet Names Can Trigger a Warning

If a workbook contains extra sheets whose names do not contain `member`, `action`, or `tier`, the app can end with an `Unrecognized sheet name or format in Excel` warning message.

## Best Practice Import Strategy

For the cleanest results in the current build:

1. Import classes indirectly by importing members or by creating them manually in Settings.
2. Import actions separately first.
3. Import tiers separately.
4. Use Excel only when you truly need the three recognized worksheet types.
5. Use CSV for a safer simple member import when you do not need historical point rows.
6. Avoid repeated member names on Excel member-history sheets unless you accept cleanup work later.

## Suggested First-Time Setup Workflow

For a new installation, this is the safest order of operations:

1. Open `Settings`.
2. Configure sync if the app will use a server.
3. Add or import classes.
4. Add or import actions.
5. Add or import tiers.
6. Add or import members.
7. Open the `Members` tab and verify roster data by class.
8. Award a few test points.
9. Check a member detail page for tier and stats accuracy.
10. Generate a quick report to confirm records are appearing correctly.

## Recommended Weekly Operating Workflow

A practical recurring workflow for church staff or volunteers:

1. Open the app and confirm sync status.
2. Go to `Points`.
3. Select the class for the current meeting.
4. award attendance and activity points
5. submit points
6. review a few members in the roster
7. generate a monthly or custom report as needed

## Troubleshooting

### Problem: No classes appear on the Points screen

Likely cause:

- no classes exist yet

Fix:

1. Open `Settings`.
2. Add at least one class.
3. Return to the `Points` tab.

### Problem: Cannot add a member

Likely causes:

- member name is blank
- no class is selected
- no classes exist yet

Fix:

1. Add a class first if needed.
2. Choose the class.
3. Re-enter the member name.

### Problem: Imported workbook shows a warning

Likely causes:

- an extra unrecognized sheet exists
- required headers are missing
- numeric columns are not numeric
- date format is invalid

Fix:

1. Use only the recognized sheet names from the template.
2. Keep the exact header names.
3. Make sure points and thresholds are whole numbers.
4. Use `MM/DD/YYYY` or ISO 8601 dates.

### Problem: Duplicate members after Excel import

Likely cause:

- repeated member rows in the Excel member-history sheet

Fix:

1. Review imported members in `Settings`.
2. Delete duplicates manually.
3. Prefer CSV for simple member-only imports in the future.

### Problem: Sync stays offline

Likely causes:

- missing server URL
- incorrect WebSocket URL
- server unavailable

Fix:

1. Open `Settings`.
2. Check the server URL carefully.
3. Tap `Connect`.
4. Try `Pull Full Sync from Server` after connection is restored.

## Features Present in Code but Not Fully Exposed as End-User Workflows

The codebase includes some supporting behavior that is not surfaced as a complete in-app workflow.

### BackupManager

There is a `BackupManager` class that can copy the local SQLite database into timestamped backup files and keep only the newest five backups. However, there is no visible backup button or backup management screen in the current UI files reviewed here.

### Saved Reports Storage

`ReportManager` includes in-memory saved report support, but the current visible UI does not expose a saved reports management screen.

### Tutorial Flag

`ContentView` includes an `@AppStorage("hasSeenTutorial")` property, but the current UI does not show a tutorial flow in the files reviewed for this guide.

## Data Quality Recommendations

To keep the app clean and reporting reliable:

- standardize class names
- avoid near-duplicate action names
- keep tier thresholds intentional and ordered
- use one preferred spelling per member name
- verify imports in the roster immediately after loading data
- test report output after any large import

## Admin Checklist

Use this checklist after setup or large imports:

1. Confirm classes exist.
2. Confirm members appear in the expected class.
3. Confirm actions display correct point values.
4. Confirm tiers display correct thresholds.
5. Award a small test action.
6. Open one member detail screen.
7. Confirm lifetime points changed.
8. Confirm tier display still makes sense.
9. Generate a report.
10. Confirm sync status if using a server.

## Final Notes

ChurchScore already covers the core loop of class-based member management, point awarding, tier tracking, imports, reporting, and live sync status. The most important operational caution in the current build is the difference between CSV import behavior and Excel import behavior, especially for member history imports. If the provided Excel template is used carefully and the import caveats above are followed, it should give you the cleanest path into the app's current importer.
