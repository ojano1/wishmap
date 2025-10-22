### 🧭 Context Brief for Tomorrow

You’re working on an **Obsidian plugin** that extends the **WishMap** system.  
The plugin currently has a working module:  
`src/schedulers/areaScheduler.ts` — it creates a **“Yearly Schedule” grid** under area pages where users can **drag wish links** (e.g., `✨Wish - drag1`) into year columns like `2025` and `2026`.

- It updates the wish’s frontmatter with a tag (`Y2025`, `Y2026`).
    
- It displays a purple card in that column.
    
- Links under “✍️ My Wishes” get a light purple `#Y2025` bubble when already scheduled.
    
- The drag interaction disables link preview popups and prevents duplicate drags.
    
- The layout uses Obsidian theme variables and `color-mix` for adaptive coloring.
    

Tomorrow’s task:  
Create a **similar column layout for the “My Wishes” page itself**, where users can drag wish links into different **custom categories or priorities** (e.g., “Short-term”, “Long-term”, “Someday”).

The new scheduler should:

- Work exactly like `areaScheduler` (drag + drop cards).
    
- Use `kindFromBasename` and frontmatter updates as before.
    
- Allow adding or changing a wish’s category tag (e.g., `#WishShort`, `#WishLong`).
    
- Visually follow the same design style: light grey containers, purple cards, rounded corners.
    
- Be implemented as a new file, likely `src/schedulers/wishScheduler.ts`.
    
- Use the same helpers and CSS approach as `areaScheduler.ts`.
    

You can reference `areaScheduler.ts` for structure:  
`class WishColsChild extends MarkdownRenderChild` → mounted via `postProcess()` inside a new `WishScheduler` class.