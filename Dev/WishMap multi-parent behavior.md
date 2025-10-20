# WishMap spec v1.2, multi-parent behavior

## Core change

- A child can have many parents.
- The child **binds to one primary parent** only.
- Primary parent = the **earliest scheduled** parent.
- Child **inherits tags** from the primary parent.
- Child **does not** roll up time or budget to non-primary parents.
- Non-primary parents only “watch” the child’s **done** state.

## “Earliest scheduled” rule

Pick the parent with the earliest concrete schedule, using this order:

1. `#due` date, earlier wins.
2. If no due, compare `#week`, then `#month`, then `#quarter`, then `#year`.
3. If ties, choose the **most recently edited** parent.
4. If still tied, choose the parent in the **same folder** as the child.

Store result as:

- `primaryParent: path`
- `secondaryParents: Set<path>`

## Tag inheritance with multi-parent

- Child takes tags from the **primary** parent if the child lacks them.
- If the child sets a `#due`, derive all period tags from the due and keep them.  
	Primary selection can change based on the new due.
- No **double tags** on the child. One value per level.

## Bottom-up autofill

When the child gets `due`:

- Derive week, month, quarter, year on the child.
- If the **primary** parent lacks any of those levels, fill them.
- Do not write tags to secondary parents.

## Top-down inheritance

When a parent gets new tags:

- If it becomes the **earliest** scheduled parent, it becomes primary.
- Update the child’s missing tags from this parent.
- Do not add tags from secondary parents.

## Rollups

- `duration_hours`, `budget`, and completion progress roll up **only** through the primary parent chain.
- Secondary parents show the child’s checkbox state for visibility, but **no totals** include that child.
- In UI, mark child as “shared” with a small dot. Show the primary parent name in tooltip.

## Prompts

- Same as v1.1.
    
- If a child has multiple parents with different schedules, sidebar shows:

	- “This item follows **Primary: Project A (Q1 2025)**. Others are observers.”


## Conflict handling

- If the child has manual tags that conflict with primary parent:
    
    - Flag ⚠️ on the child. Offer:
        
        - “Align to parent”
            
        - “Keep child tags”
            
- If two parents compete to be earliest at the same precision:
    
    - Apply tie-breakers above.

    - Log decision in console.


## Indexer requirements

- Build `parents: Set<path>`.
    
- Compute `primaryParent` each scan using the “earliest” rule.
    
- Keep `secondaryParents = parents − {primaryParent}`.
    
- Maintain `childrenOf` for both primary and secondary, but mark relation type:
    
    - `primary` or `observer`.
        

## Inference algorithm, concise

1. Index nodes and parents.
    
2. Compute `primaryParent`.
    
3. **Top-down** from primary:
    
    - Fill child’s missing tags.
        
4. **Bottom-up** from child due:
    
    - Derive tags on child.
        
    - Fill missing tags on primary parent and up its chain.
        
5. Skip writes to secondary parents.
    
6. Re-evaluate primary if any schedule changes.
    

## Examples

**Child with two parents, only primary rolls up**

`📌 task – Prepare test data #due/2025-03-22 <!-- wishmap: #year/2025 #quarter/2025-Q1 #month/2025-03 #week/2025-W12 -->  Parents: [[project – API v2]] (Q1 2025), [[project – Mobile QA]] (Q2 2025)  Primary: API v2       # earliest Secondary: Mobile QA  # observer only`

- Time and budget count under **API v2**.
    
- **Mobile QA** shows checkbox state only.
    

**Primary switch**

- If Mobile QA gets a due `2025-03-15`, it becomes earlier.
    
- Primary switches to Mobile QA.
    
- Rollups and inheritance follow Mobile QA from then on.
    

## UI chips

- Child chip: `Primary: Project A · Shared`
    
- Secondary parent list in hover.
    
- In Time Tree, child appears under the primary parent’s period only.
    

## Logging

- On primary selection:
    
    `[WishMap] primary set: task.md -> project-api-v2.md (Q1 2025)`
    
- On primary switch:
    
    `[WishMap] primary switched: task.md from project-api-v2.md to project-mobile-qa.md`
    

## Migration notes

- Existing shared children will be evaluated at next scan.
    
- First pass assigns a primary using the earliest rule.
    
- No writes to secondary parents.