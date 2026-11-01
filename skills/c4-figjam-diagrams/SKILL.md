---
name: "c4-figjam-diagrams"
description: "Create C4 model architecture diagrams (L1 System Context, L2 Containers, optional L3/dynamic) on a FigJam board with consistent colors, line styles, legends and one section per level."
---

# C4 diagrams on FigJam

Use the C4 model (https://c4model.com) as the standard for all architecture diagrams. Draw them on FigJam with the Figma connector. The result is one board section per C4 level, and each section has a legend.

## 1. Collect the inputs before you draw

- The target board URL: `figma.com/board/{fileKey}/...`. Always pass a `fileKey`. Without one, `generate_diagram` asks for a Figma plan in a widget that the user may not see.
- The actors and systems. Read the project's actor catalog, decision notes and memory first. Do not invent nodes or edges. If a fact is not known, label it (for example "product not chosen yet") or list it as an assumption in your reply.
- The levels to draw. The default is L1 and L2. Draw L3 (Components) or a Dynamic/sequence view only if the user asks for them.

## 2. Load the Figma guidance (this is mandatory)

1. `create_session` with the fileKey. Reuse the sessionId on every Figma call.
2. Read `skill://figma/figma-generate-diagram/SKILL.md` and `references/flowchart.md` before `generate_diagram`.
3. Read `skill://figma/figma-use-figjam/SKILL.md`, `references/create-section.md` and `references/position-figjam-nodes.md` before `use_figma`. Pass `skillNames: "resource:figma-use-figjam"`.

Use a plain `flowchart LR` and do not pass `useArchitectureLayoutCode`. The architecture layout does not allow external -> service edges, which C4 needs (for example a PIM that sends events into the system).

## 3. C4 content rules

**L1 System Context**
- Show one box for the software system in scope. Do not show its internals.
- Group the people (`([Name [Person]])` stadium shape) and the external systems by role: source systems, CRM, third-party services, legacy.
- Write every edge label from the source side, with 1 to 4 words ("Sends NA leads").

**L2 Containers**
- Use a system-boundary subgraph. Put nested subgraphs inside it for each domain (for example "Edge and delivery", "Middleware - product sync", "Middleware - leads and dealers").
- One node is one deployable unit. Put the technology in brackets: `Edge router [CloudFront + Lambda@Edge] - routes by locale path`.
- Use these shapes: `[( )]` for datastores and `[[ ]]` for queues.
- Keep the external systems and people from L1, with the same names.

**L3 / Dynamic** (only when asked): L3 zooms into one container and has the same legend. For a runtime flow, use a `sequenceDiagram` and do not use notes, because notes are removed.

## 4. Visual standard

Use only light FigJam fills with dark text. Mermaid applies only `fill` and `stroke`, and saturated fills make the text hard to read.

| Role | Node fill / stroke | Subgraph fill |
|---|---|---|
| Person | `#DCCCFF` / `#874FFF` | `#F7F2FF` |
| System in scope, Webflow/web app container | `#C2E5FF` / `#3DADFF` | `#EEF7FF` |
| Middleware / integration containers | `#CDF4D3` / `#66D575` | `#EDFBEF` |
| Edge, CDN, delivery | `#FFE0C2` / `#FF9E42` | `#FFF4EA` |
| Source systems (PIM, DAM, feeds) | `#FFECBD` / `#FFC943` | `#FFF8E6` |
| CRM | `#FFC2EC` / `#F849C1` | `#FFF0FA` |
| Third-party services | `#C6FAF6` / `#5AD8CC` | `#EEFDFB` |
| Legacy (to be retired) | `#D9D9D9` / `#B3B3B3` | `#F5F5F5` |
| Legend text items | `#FFFFFF` / `#B3B3B3` | `#FFFFFF`, stroke `#1E1E1E` |

Line styles:
- `==>` thick line: a call that goes through the middleware.
- `-.->` dotted line: an event, a webhook or a queue hop.
- `-->` thin line: a page request or a user action.

Add a `legend` subgraph to every diagram. It contains one node for each color in use and one node for each line style. Create the legend nodes in the order you want them shown, because step 6 sorts them by id.

Apply colors with `classDef` + `class`, and apply subgraph tints with `style <subgraphId> fill:...,stroke:...`. Do not use `end`, `graph` or `subgraph` as ids. Use camelCase ids, no emojis and no `\n`.

## 5. Generate

Call `generate_diagram` one time for each level, with the same `fileKey`. Use names like `<System> C4 - L1 System Context` and `<System> C4 - L2 Containers`. Each call puts its nodes at the top level of the board.

## 6. Put each level in its own section and fix the legend order

1. Run `get_figjam(fileKey, "0:1", includeImagesOfNodes:false)`. Each generation has its own id prefix (for example the first diagram is `3:*` and the second is `4:*`). If the board is large, the result goes to a file. Use grep on that file.
2. Run one `use_figma` call with this pattern:

```javascript
(async () => {
  const h = (r, g, b) => ({ r: r / 255, g: g / 255, b: b / 255 });
  const page = figma.currentPage;
  const num = (id) => parseInt(id.split(':')[1], 10);
  // 1) Legend order: the ids are in creation order
  for (const legId of LEGEND_SECTION_IDS) {
    const leg = await figma.getNodeByIdAsync(legId);
    const kids = [...leg.children].sort((a, b) => num(a.id) - num(b.id));
    kids.forEach((k, i) => { k.x = 48; k.y = 48 + i * 192; });
    leg.resize(leg.width, 48 + kids.length * 192);
  }
  // 2) Make one parent section for each level
  for (const g of GROUPS) { // { prefix: '3:', name: 'L1 System Context' }
    const nodes = page.children.filter((n) => n.id.startsWith(g.prefix));
    const boxes = nodes.filter((n) => n.type !== 'CONNECTOR').map((n) => n.absoluteBoundingBox);
    const minX = Math.min(...boxes.map((b) => b.x)), minY = Math.min(...boxes.map((b) => b.y));
    const maxX = Math.max(...boxes.map((b) => b.x + b.width)), maxY = Math.max(...boxes.map((b) => b.y + b.height));
    const abs = new Map(nodes.map((n) => [n.id, { x: n.x, y: n.y }]));
    const pad = 96, sec = figma.createSection();
    sec.name = g.name; sec.x = minX - pad; sec.y = minY - pad;
    sec.resize(maxX - minX + 2 * pad, maxY - minY + 2 * pad);
    sec.fills = [{ type: 'SOLID', color: h(0xf9, 0xf9, 0xf9) }];
    for (const n of nodes) { const p = abs.get(n.id); sec.appendChild(n); n.x = p.x - sec.x; n.y = p.y - sec.y; }
  }
  figma.closePlugin();
})();
```

3. Run `get_figjam` again to verify the result. A `use_figma` return value inside an async IIFE does not come back to you. Check that each level section holds its nodes and connectors, and that the sections do not overlap. If they overlap, move the second section to the right of the first.

## 7. Limits and recovery

- If the board belongs to a Figma Starter plan, the MCP calls have a low limit. Do the minimum number of calls: generate, then one inspection, then one `use_figma`, then one verification. If a call is blocked, tell the user what is on the board and what is not done.
- `get_screenshot` and thumbnail URLs can fail (rate limit, or the proxy blocks s3-alpha.figma.com). Use `get_figjam` to verify.
- Do not regenerate the same diagram more than 2 times. Each regeneration adds a new copy to the board. Ask the user if they want the old copy removed, and do not delete content that you did not create.
- To update a diagram later, regenerate it with the same Mermaid, change only what is new, and then repeat step 6.

## 8. Reply to the user

Give the board link and list the sections you made. Name each assumption or open fact in the diagram in one line each. Keep the Mermaid source in the project files (for example `c4/<level>.mmd`) so that the next change starts from it.