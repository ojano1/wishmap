

## 1. Overview

WishMap tracks three active types for scheduling:  
**Goal**, **Project**, and **Task.**

Scheduling is flexible.  
Users can plan top-down (year → day) or bottom-up (task → goal).  
The system keeps both paths synchronized automatically.

---

## 2. Prompt System

|Level|Trigger|Type|Placement|Purpose|
|---|---|---|---|---|
|**Goal**|No `targetYear`|Soft prompt|Sidebar banner|Suggest committing this goal to a year.|
|**Project**|Has parent goal with year, no `targetQuarter`|Soft prompt|Sidebar banner|Suggest assigning quarter.|
|**Task**|No `due`|Hard prompt|Inline, under checkbox|Request due date or week assignment. Also checks if parent has tags.|

**Behavior**

- Soft prompts can be ignored.
- Hard prompts require a decision (set due date or skip).
- Prompts update live as metadata changes.
- Sidebar always shows count of unscheduled items.
---
## 3. Prompt Logic Flow

`onNoteOpen(note):
  if type == goal and !targetYear:
      showSoftPrompt("Schedule this goal for a year?")
  if type == project and parentHasYear(note) and !targetQuarter:
      showSoftPrompt("Assign a quarter?")
  if type == task and !due:
      checkParents()
      showHardPrompt("Set due date or week.")

---

## 4. Parent Check + Auto-Fill (Bottom-Up Inference)

When user sets a due date, the system **infers and fills** missing period tags for the task and its ancestors.

**Derived Fields**

|Field|Source|Example|
|---|---|---|
|`targetWeek`|ISO week from `due`|2025-W12|
|`targetMonth`|Month of `due`|2025-03|
|`targetQuarter`|Month → quarter|2025-Q1|
|`targetYear`|Year of `due`|2025|

**Propagation Upward**

- Task → Project → Goal
- Any missing field gets filled.
- Existing manual tags are preserved.
- Conflicts are flagged, not overwritten.

---

## 5. Top-Down Inheritance

If user schedules a parent first:

- Projects inherit parent’s `targetYear` and `targetQuarter`.
- Tasks inherit `targetQuarter` and `targetMonth`.
- User can override any value manually.

---

## 6. Conflict Handling

|Case|Action|
|---|---|
|Parent=2024, Child=2025|Flag with ⚠️ “Mismatch with parent”|
|Partial tags (goal has year only)|Fill missing quarter/month automatically|
|Multiple parents|Use nearest valid ancestor|
|Recurring task|Apply only to generated instances|

---

## 7. Auto-Fill Rules

`onDueSet(task):
  derive period tags from due
  for each ancestor:
    if tag missing:
      fill derived value
  mark derived tags as (auto)
  show toast: "Tags auto-filled from due date. [Undo]"
`

Manual edit removes the “auto” mark.

---

## 8. Data Integrity

- Every scheduled node ends with a complete timeline:

`targetYear
targetQuarter
targetMonth
targetWeek
due
`
- Ensures all nodes appear correctly in the **Time Tree** view.
- Keeps rollups consistent for duration and budget.

---

## 9. User Experience Summary

|Action|System Response|
|---|---|
|Create goal|Prompt to assign year|
|Create project|Prompt to assign quarter|
|Create task|Prompt to set due date|
|Set due date|Auto-fill all missing parent tags|
|Edit parent tags|Children inherit updated tags|
|Ignore prompts|No enforcement, optional scheduling|
|Jump steps (goal → task)|Still works, inference fills gaps|

---

## 10. Key Principles

1. **Non-linear freedom:** Any order of creation allowed.
2. **Auto-coherence:** Missing tags always resolved.
3. **Minimal friction:** Prompts are context-aware, never blocking.
4. **Bidirectional sync:** Tags propagate top-down and bottom-up.
5. **Transparency:** Auto tags visible and reversible.


## **Phase 1 – Build solid base (must work before visuals)**

### **1. Metadata scanner**

Goal: detect and index all relevant notes (goal, project, task).

- Read frontmatter and inline YAML keys.
- Record: file path, type, parent links, period tags, done state.
- Save to an in-memory index (`wishmap.index`).

**Why first:** every later feature (prompts, rollups, tree, scheduler) depends on accurate data.

---

### **2. Inference engine**

Implements your new scheduling logic:

- **Top-down inheritance** and **bottom-up propagation**.
- Auto-fill missing tags.
- Mark auto-filled tags.
- Flag conflicts.

This runs after each scan or metadata change.

---

### **3. Prompt system**

After index + inference are stable, attach to `onNoteOpen`.

- Check type and tags.
- Show soft or hard prompt according to rule set (v1.0).
- Hard prompt opens date picker or drag board.

Keep prompts plain first, then refine UX later.

---

## **Phase 2 – Visualization**

### **4. Tree rework**

Upgrade your current mapping tree to pull data from the new index.  
Add chips for time tags (Year, Qtr, Month, Week).

### **5. Time tree prototype**

Reuse the same renderer.  
Group nodes by year → quarter → month → week → day.  
Show both structural and time tags for each node.

---

## **Phase 3 – Scheduling UI**

### **6. Commit / Drag board**

- Kanban-style panel for assigning period tags.
- Context-aware filters (unscheduled goals, projects, tasks).
- Drag updates tags instantly in frontmatter.
    

---

## **Phase 4 – Maintenance and performance**

### **7. Cache + diff**

- Persist small JSON cache for index data.
- Only rescan changed files.

### **8. Rollups**

- Reconnect duration, budget, and completion rollups to new index.
    

---

## **Recommended order to code**

1. Metadata scanner → test with console log of all nodes.
2. Inference engine → verify tags propagate correctly.
3. Prompt logic → show inline and sidebar prompts.
4. Update Tree → display tags next to links.
5. Build Time Tree → verify all nodes appear correctly.
6. Add drag board → confirm tag writing works.

New requirwment:

On wish page, make kanban like 4 column of dinamic quarter, yyyy-Q1 - yyyy-Q4. Theyear display depan on today's date. Make it possible that the goals listed can be dragged into one of the columns. Based on colimn selection, update the goal's tag according to its Quarter column.

On goal page, make 12  dinamic rows of month names. If children tasks has no due date,  Make the project **listed** draggable to that row and apply the month tag yyyy-01 - yyyy-12.  Title the rows same as the tags. Adjjuat the year according to created  date. So the month before the ctrated date will be labeled the next year. If the childern task already have due date, auto assign the project following the earliest due among the children. Ig nore children with no due. When children has due date, dim the rows and disable dragging . When due date change, remove old project tag and apply new month tag.
When a project assigned a month tag, assign a year tag to the parwnt: goal. If the goal has children projects, apply the earliest year. 



On project page, 
