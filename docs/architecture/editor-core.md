# Editor Core Architecture

## Purpose
Velti's editor core unifies **visual editing** (drag/drop, inline controls) and **code-like editing** (structured blocks and props) around a shared document model.

## Source of Truth Model
Velti stores each page as a **block graph document** serialized as JSON.

- `Document`: metadata + ordered top-level `nodeIds`
- `Node`: block instance with `type`, `props`, `children`, and layout metadata
- `Registry`: maps `type` -> renderer, schema, inspector controls, and validation rules

```ts
export type Document = {
  id: string;
  projectId: string;
  title: string;
  version: number;
  rootNodeIds: string[];
  nodes: Record<string, Node>;
};

export type Node = {
  id: string;
  type: string;
  props: Record<string, unknown>;
  children: string[];
  styleTokens?: string[];
};
```

This structure is stable enough for visual manipulation while still being explicit for code-mode editing.

## Visual ↔ Code Sync Strategy
Both editor modes write through a single command pipeline:

1. User action becomes a command (`insertNode`, `updateProps`, `moveNode`, `removeNode`).
2. Command mutates the in-memory document store.
3. Validation runs against block schema from the registry.
4. Patch event is broadcast to visual canvas and code panel.
5. Autosave persists a revision.

Because both modes mutate the same model, there is no lossy conversion between HTML and GUI state.

## Block Registry and Extensibility
Blocks are registered through a plugin-like manifest:

- `schema`: prop contract and defaults
- `render`: runtime component renderer
- `inspector`: form controls for editing props
- `capabilities`: nesting/layout constraints

This enables third-party block packs and internal presets (hero, pricing, contact form) without changing the editor core.

## PostgreSQL Persistence
Core tables:

- `projects` — workspace-level metadata
- `pages` — URL slug, title, publish status
- `page_documents` — current document JSON snapshot
- `page_revisions` — append-only history (document + timestamp + actor)
- `assets` — uploaded images/files referenced by nodes

### Revisioning
Each successful mutation batch can produce a revision record. Publishing pins a specific revision for deterministic deploy/export.

## Export Pipeline (Including Tauri)
Export flow boundaries:

1. Read published revision.
2. Compile document graph into framework output (static Svelte routes/components or neutral HTML/CSS bundles).
3. Package web output for deployment.
4. Optional: wrap exported site in the Tauri desktop export tool for offline desktop delivery.

Tauri is treated as an output packaging target, not part of authoring-time editing.

## End-to-End Edit Flow Example
**Example:** user edits a CTA button label.

1. Visual mode selects `Button` node `node_42`.
2. Inspector updates `props.label = "Book now"`.
3. `updateProps(node_42, { label: "Book now" })` command is applied.
4. Canvas rerenders `Button` immediately.
5. Code panel shows updated JSON/DSL for the same node.
6. Debounced autosave writes a new `page_revisions` row.

The same flow works if the user edits the label from code mode first.
