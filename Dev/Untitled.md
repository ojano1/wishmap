# Quick setup

- Open Obsidian dev console. View → Toggle Developer Tools → Console.
    
- Enable debug: filter for `[WishMap]`.
    

# 1) Indexer sanity

Create three notes via your templates or quick notes.

**goal – 10k run.md**

`🎯 goal – 10k run #status/active`

**project – base plan.md**

`🚀 project – base plan Parents: [[goal – 10k run]] #status/active`

**task – buy shoes.md**

`📌 task – buy shoes Parents: [[project – base plan]] #status/active`

Actions:

- Save each file.
    
- In console, expect “indexer ready” once, then no errors.
    
- Run command: WishMap: Rescan vault. Confirm “Rescan complete.”
    

# 2) Prompts

Open each note.

- Goal, no year tag → Notice: “Schedule this goal for a year.”
    
- Project, parent has no year yet → no quarter prompt.
    
- Add `#year/2025` to goal. Open project → Notice: “Assign a quarter.”
    

# 3) Bottom-up autofill from due

Open the task. Add YAML due.

`--- due: 2025-03-22 ---`

Expected:

- Console shows “service tags updated” for the task.
    
- Task gets a service line appended with:  
    `#year/2025 #quarter/2025-Q1 #month/2025-03 #week/2025-W12 #due/2025-03-22`
    
- Project and goal get auto-filled tags if missing. Console logs:
    
    - “Auto-filled parent from child due” for project, then goal.
        
- Open project and goal files, scroll bottom. Service line present with year/quarter.
    

# 4) Top-down inheritance

- On project, manually add `#quarter/2025-Q2`.
    
- Open the child task. If task had no quarter before, it now shows Q2 in service line. If it already had Q1 from due, it stays Q1, and you’ll see no overwrite.
    

# 5) Primary parent selection

Create another project, link the same task to both.

**project – race prep.md**

`🚀 project – race prep #status/active #quarter/2025-Q3`

Edit the task to include both parents:

`Parents: [[project – base plan]], [[project – race prep]]`

Expected:

- Primary = earliest scheduled parent. With task due 2025-03-22, primary should be **base plan** (Q1 derivable).
    
- Console log: `primary set: task -> project-base-plan.md`.
    
- Task appears only under the primary’s period in your Time Tree later. For now, verify logs and service line unchanged.
    

# 6) Primary switch test

- Set race prep to earlier quarter `#quarter/2025-Q1` or give it `#due/2025-03-15`.
    
- Modify task file to trigger reindex (add a space and save).
    
- Console log: `primary switched: task …`.
    
- Check that only the **primary chain** gets autofilled tags on parents. Secondary stays unchanged.
    

# 7) Notices behavior on task

- Remove task YAML `due`. Save. Open task.
    
- Notice should say: set due date, and if parents lack tags, mentions they’ll be filled automatically.
    
- Add due again. Verify tags and parents fill.
    

# 8) Maintenance commands

- Run “WishMap: Rebuild inference.” No changes expected if already correct.
    
- Run “WishMap: Rescan vault.” Watch no errors.
    

# 9) Service line sync

- Delete the service line in a file that has due. Save.
    
- Plugin should reinsert it with the same tags. Console shows update.
    

# 10) No double tags

- In any file, manually add conflicting month `#month/2025-04` while quarter = `2025-Q1`.
    
- `resolveConflicts` only collects messages. You won’t see UI yet. Conflicts won’t be auto corrected. Keep as is for now.
    

# 11) Rollups unaffected

- Your existing rollup flow should still fire.
    
- Create/modify a task’s `duration_hours`. Confirm project rollup runs from your logs and command.
    

# 12) Performance smoke

- Duplicate the task 50x in the same folder.
    
- Set due on one task. No lag. Console shows throttled writes, one service-line write per touched file.
    

# Troubleshooting

- No prompts? Ensure the file name starts with the correct emoji or prefix, and type is goal/project/task.
    
- No service line? Make sure you saved after adding `due` in YAML.
    
- “Cannot read property …”: reload Obsidian, then run Rescan.
    
- Desktop only git lock check. On mobile, it silently returns false.