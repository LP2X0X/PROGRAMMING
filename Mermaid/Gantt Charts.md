---
tags:
  - mermaid
  - gantt-chart
  - diagram
  - project-management
---

# Gantt Charts

Mermaid's Gantt chart syntax lets you define project schedules, task dependencies, and milestones directly in Markdown. Charts render as horizontal bar timelines grouped by section, making them ideal for sprint planning, release timelines, and project roadmaps without leaving your notes.

See also: [[Syntax Basics]] | [[Styling and Themes]]

---

## 🔹 Quick Reference

| Element | Syntax | Example |
|---|---|---|
| Chart declaration | `gantt` | `gantt` |
| Title | `title <text>` | `title Sprint 12` |
| Date input format | `dateFormat <format>` | `dateFormat YYYY-MM-DD` |
| Axis display format | `axisFormat <format>` | `axisFormat %m/%d` |
| Section | `section <name>` | `section Design` |
| Normal task | `name : id, start, duration` | `Design UI : des1, 2024-01-01, 5d` |
| Task with end date | `name : id, start, end` | `Build API : api1, 2024-01-01, 2024-01-10` |
| Active task | `active, ...` | `Coding : active, t1, 2024-01-05, 4d` |
| Done task | `done, ...` | `Planning : done, t2, 2024-01-01, 3d` |
| Critical task | `crit, ...` | `Hotfix : crit, t3, 2024-01-10, 2d` |
| Milestone | `milestone, id, date, 0d` | `Launch : milestone, m1, 2024-01-20, 0d` |
| Dependency | `after <taskId>` | `Testing : t4, after api1, 3d` |
| Exclude days | `excludes <days>` | `excludes weekends` |
| Exclude specific date | `excludes <date>` | `excludes 2024-12-25` |
| Tick interval | `tickInterval <interval>` | `tickInterval 1week` |
| Today marker | `todayMarker off` | `todayMarker off` |

---

## 🔹 Basic Structure

Every Gantt chart starts with the `gantt` keyword, followed by optional configuration directives and then your tasks.

**Required and optional directives:**

- **`gantt`** -- declares the diagram type (must be the first line)
- **`title`** -- sets the chart heading (optional but recommended)
- **`dateFormat`** -- defines how dates are parsed in task definitions (default: `YYYY-MM-DD`)
- **`axisFormat`** -- controls how dates display on the horizontal axis
- **`section`** -- groups tasks under a labeled heading

**Task syntax** follows one of two patterns:

```
Task name : taskId, startDate, duration
Task name : taskId, startDate, endDate
```

- The **task ID** is optional -- if omitted, Mermaid auto-generates one. Including IDs is necessary when you want to reference tasks in dependencies.
- **Duration** uses suffixes: `d` (days), `w` (weeks), `h` (hours), `m` (minutes).
- If you omit the start date, the task starts after the previous task ends.

```mermaid
gantt
    title My First Gantt Chart
    dateFormat YYYY-MM-DD
    axisFormat %m/%d

    section Planning
    Define requirements : req, 2024-01-01, 5d
    Create wireframes   : wire, after req, 3d

    section Development
    Build frontend      : fe, after wire, 7d
    Build backend       : be, after wire, 7d
```

---

## 🔹 Date Formats

### Input Format (`dateFormat`)

The `dateFormat` directive tells Mermaid how to **parse** the dates you write in task definitions. It follows Moment.js/Day.js formatting tokens.

| Token | Meaning | Example |
|---|---|---|
| `YYYY` | 4-digit year | `2024` |
| `YY` | 2-digit year | `24` |
| `MM` | Month (zero-padded) | `01`..`12` |
| `M` | Month (no padding) | `1`..`12` |
| `DD` | Day of month (zero-padded) | `01`..`31` |
| `D` | Day of month (no padding) | `1`..`31` |
| `HH` | Hour (24h, zero-padded) | `00`..`23` |
| `mm` | Minute | `00`..`59` |

**Common `dateFormat` patterns:**

| Pattern | Input looks like |
|---|---|
| `YYYY-MM-DD` | `2024-01-15` |
| `DD/MM/YYYY` | `15/01/2024` |
| `MM-DD-YYYY` | `01-15-2024` |
| `YYYY-MM-DD HH:mm` | `2024-01-15 09:30` |

### Axis Format (`axisFormat`)

The `axisFormat` directive controls how dates appear on the **horizontal axis**. It uses **D3 time format** specifiers (different from `dateFormat` tokens).

| Specifier | Meaning | Example |
|---|---|---|
| `%Y` | 4-digit year | `2024` |
| `%m` | Month number (zero-padded) | `01` |
| `%d` | Day of month (zero-padded) | `05` |
| `%e` | Day of month (space-padded) | ` 5` |
| `%b` | Abbreviated month name | `Jan` |
| `%B` | Full month name | `January` |
| `%a` | Abbreviated weekday | `Mon` |
| `%A` | Full weekday name | `Monday` |
| `%H` | Hour (24h) | `14` |
| `%I` | Hour (12h) | `02` |
| `%p` | AM/PM | `PM` |
| `%W` | Week of year | `03` |

**Common `axisFormat` patterns:**

| Pattern | Output looks like |
|---|---|
| `%m/%d` | `01/15` |
| `%b %d` | `Jan 15` |
| `%Y-%m-%d` | `2024-01-15` |
| `%b %Y` | `Jan 2024` |
| `%d %b` | `15 Jan` |
| `%a %m/%d` | `Mon 01/15` |

---

## 🔹 Task Types

Mermaid supports several **task status keywords** that change how a task bar is rendered. These keywords go before the task ID in the task definition.

| Keyword | Appearance | Use case |
|---|---|---|
| *(none)* | Default filled bar | Upcoming / planned tasks |
| `active` | Highlighted / pulsing bar | Currently active work |
| `done` | Dimmed / grayed-out bar | Completed tasks |
| `crit` | Red / critical-path bar | High-priority or blocking tasks |
| `milestone` | Diamond marker (0-duration) | Key deliverables or checkpoints |

**Combining keywords** -- you can mix `done` and `crit` to show a completed critical task:

```
Completed hotfix : done, crit, fix1, 2024-01-01, 2d
```

**Milestone** tasks must have a duration of `0d` to render as a diamond marker rather than a bar.

```mermaid
gantt
    title Task Types Overview
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Status Examples
    Normal task       :         t1, 2024-01-01, 5d
    Active task       : active, t2, 2024-01-06, 4d
    Done task         : done,   t3, 2024-01-01, 3d
    Critical task     : crit,   t4, 2024-01-10, 3d
    Done + Critical   : done, crit, t5, 2024-01-04, 2d
    Key milestone     : milestone, m1, 2024-01-13, 0d
```

---

## 🔹 Dependencies

Use `after <taskId>` in place of a start date to chain tasks together. The dependent task begins when its predecessor ends.

**Syntax:**

```
Dependent task : depId, after predecessorId, duration
```

You can depend on **multiple tasks** by listing them separated by spaces -- the dependent task starts after the *latest* of them finishes:

```
Integration test : int1, after fe1 be1, 3d
```

This makes `int1` wait for both `fe1` and `be1` to complete before starting.

**Chaining** several tasks in sequence is straightforward -- each references the one before it:

```mermaid
gantt
    title Dependency Chain
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Pipeline
    Requirements   : req, 2024-01-01, 3d
    Design         : des, after req, 4d
    Implementation : imp, after des, 7d
    Testing        : test, after imp, 5d
    Deployment     : dep, after test, 2d
    Go-live        : milestone, m1, after dep, 0d
```

---

## 🔹 Sections

Sections visually **group tasks** under labeled headings on the chart. Use them to organize by project phase, team, workstream, or any logical grouping.

- Each `section <Name>` line starts a new group
- All tasks after a `section` line belong to that section until the next `section` line
- Sections render as alternating background bands for readability

```mermaid
gantt
    title Multi-Team Project
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Backend Team
    API design       : api, 2024-02-01, 5d
    Database schema  : db, 2024-02-01, 4d
    API development  : apidev, after api db, 10d

    section Frontend Team
    UI mockups       : ui, 2024-02-01, 4d
    Component build  : comp, after ui, 8d
    Integration      : integ, after comp apidev, 5d

    section QA Team
    Test planning    : plan, 2024-02-01, 3d
    Test execution   : exec, after integ, 7d
    Sign-off         : milestone, signoff, after exec, 0d
```

---

## 🔹 Excluding Days

Use `excludes` to skip weekends or specific dates (holidays) from the schedule. Tasks that span excluded days are extended automatically.

**Exclude weekends** (Saturday and Sunday):

```
excludes weekends
```

**Exclude specific dates** (holidays, company closures):

```
excludes 2024-12-25, 2024-12-31, 2025-01-01
```

**Combine both:**

```
excludes weekends, 2024-12-25, 2024-12-31
```

When `excludes weekends` is active, a task with duration `5d` spans Monday through Friday (one working week), skipping Saturday and Sunday. Duration is counted in **working days** only.

> **Note:** The `excludes` directive goes at the top of the chart, alongside `title` and `dateFormat`.

---

## 🔹 Real-World Examples

### Software Development Sprint

A two-week sprint with design, development, testing, and deployment phases, using dependencies and a mix of task statuses.

```mermaid
gantt
    title Sprint 14 — User Authentication
    dateFormat YYYY-MM-DD
    axisFormat %b %d
    excludes weekends

    section Design
    User stories          : done, us, 2024-03-04, 2d
    Technical design      : done, td, after us, 2d
    Design review         : done, crit, dr, after td, 1d

    section Development
    Auth API endpoints    : active, auth, after dr, 4d
    JWT token service     : active, jwt, after dr, 3d
    Login UI              : login, after dr, 3d
    Registration UI       : reg, after login, 2d
    Password reset flow   : reset, after jwt, 3d

    section Testing
    Unit tests            : unit, after auth jwt, 2d
    Integration tests     : integ, after unit reg reset, 3d
    Security audit        : crit, audit, after integ, 2d

    section Release
    Staging deploy        : stage, after audit, 1d
    UAT sign-off          : milestone, uat, after stage, 0d
    Production deploy     : prod, after stage, 1d
    Sprint complete       : milestone, done_m, after prod, 0d
```

### Product Launch Timeline

A multi-month product launch with research, build, and go-to-market workstreams converging at a launch milestone.

```mermaid
gantt
    title Product Launch — Q2 2024
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Research
    Market analysis       : done, ma, 2024-03-01, 14d
    Competitor review     : done, cr, 2024-03-01, 10d
    Feature prioritization: done, crit, fp, after ma cr, 5d

    section Build
    MVP development       : active, mvp, after fp, 30d
    Beta testing          : beta, after mvp, 14d
    Bug fixes             : crit, fix, after beta, 7d

    section Go-to-Market
    Marketing site        : site, after fp, 21d
    Press kit             : press, after site, 7d
    Launch campaign       : camp, after press, 10d

    section Milestones
    Feature freeze        : milestone, m1, after mvp, 0d
    Beta release          : milestone, m2, after beta, 0d
    Public launch         : crit, milestone, m3, after fix camp, 0d
```

### Project Plan with Parallel Workstreams

A renovation project showing how independent workstreams run in parallel, then converge at a shared dependency point.

```mermaid
gantt
    title Office Renovation Project
    dateFormat YYYY-MM-DD
    axisFormat %b %d
    excludes weekends

    section Planning
    Budget approval    : done, budget, 2024-04-01, 5d
    Contractor bids    : done, bids, after budget, 7d
    Permit filing      : crit, permit, after bids, 10d

    section Demolition
    Clear furniture    : clear, after permit, 2d
    Wall removal       : demo, after clear, 3d
    Debris cleanup     : cleanup, after demo, 1d

    section Construction
    Electrical work    : elec, after cleanup, 8d
    Plumbing updates   : plumb, after cleanup, 6d
    Wall framing       : frame, after cleanup, 5d
    Drywall install    : drywall, after frame elec plumb, 4d
    Painting           : paint, after drywall, 3d

    section Finishing
    Flooring           : floor, after paint, 4d
    Furniture delivery : furn, after floor, 2d
    IT setup           : it, after furn, 3d
    Final walkthrough  : milestone, walk, after it, 0d
    Move-in day        : milestone, move, after walk, 0d
```

---

**See also**: [[Syntax Basics]] | [[Flowcharts]] | [[Styling and Themes]]
