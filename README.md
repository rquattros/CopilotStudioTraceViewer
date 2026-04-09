# Copilot Studio Trace Viewer

A single-page web application for exploring and analyzing conversation traces exported from [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/). Upload a `dialog.json` (or its `.zip` archive) and navigate through every step of the conversation to understand what happened and why.

![Dark themed trace viewer](https://img.shields.io/badge/theme-dark-0d1117) ![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen) ![Single file](https://img.shields.io/badge/single%20file-index.html-blue)

## Getting Started

Open `index.html` in any modern browser — no build step, no server, no dependencies.

## How to Export a Conversation Snapshot

Copilot Studio lets you capture a snapshot of a test conversation for offline analysis.

1. Open your agent in [Copilot Studio](https://web.powerva.microsoft.com/).
2. Open the **Test your agent** panel (click **Test** at the top of any page).
3. Have a conversation with your agent.
4. At the top of the test panel, click the **three dots (…)** menu, then select **Save snapshot**.
5. Confirm the prompt — a file named **`botContent.zip`** is downloaded.

The archive contains two files:

| File | Description |
|---|---|
| `dialog.json` | Conversational diagnostics — every activity, plan, tool call, and response in the trace. |
| `botContent.yml` | The agent's topics, entities, variables, and other content definitions. |

You can upload either the `.zip` or the extracted `dialog.json` directly into the Trace Viewer.

> **Note:** The snapshot file may contain sensitive information. Handle it accordingly.
>
> For full details see the [official documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-test-bot?tabs=webApp#save-conversation-snapshots).

## Features

### Core Trace Viewer
- **Upload dialog.json or .zip** — drag-and-drop or click to upload. ZIP files are parsed natively in the browser (no libraries).
- **Activity timeline** — every activity is displayed chronologically with color-coded badges (user message, bot message, plan, thought, tool call, search, info, typing, error).
- **Expandable details** — click any step to expand and see full metadata: timestamps, parameters, observations, raw JSON auto-formatted as key-value pairs.

### Plan Tree Grouping
- **Hierarchical plan nesting** — `DynamicPlanReceived` events open collapsible plan groups; child plans nest inside parent plans based on `parentPlanIdentifier`.
- **Smart plan closing** — plans close on `DynamicPlanFinished` or when a `DynamicPlanStepFinished` reports `state: "completed"`. User messages force-close any remaining open plans to separate conversation turns.
- **Plan pills** — color-coded labels show which plan each step belongs to, with full ancestry breadcrumbs.
- **Plan status indicators** — each plan group shows its status (running, completed, cancelled, open-ended) and step count.

### Chat Simulation Pane
- **Side-by-side chat view** — a chat panel on the right displays only user and bot messages, styled like a real chat interface.
- **Resizable splitter** — drag the handle between the trace and chat panels to adjust their widths.
- **Cross-linking** — click a chat bubble to scroll the trace to the corresponding step. Double-click a message step in the trace to highlight the chat bubble.
- **Markdown rendering** — bot responses render with headings, lists, bold text, and links.
- **Citation display** — document citations from bot responses appear as clickable links below the message.

### Search & Filtering
- **Category filters** — toggle visibility by activity type (messages, plans, thoughts, tools, search, info, typing, errors).
- **Text search** — full-text search across activity titles and content with match highlighting and result count.
- **Filter + search interaction** — filters and search work together; clearing one preserves the other.

### Statistics & Navigation
- **Stats panel** — shows total duration, plan count (completed/cancelled), user turns, bot responses, tool calls, search calls, and the slowest step (excluding user think time).
- **Breadcrumb trail** — updates as you scroll to show your current position in the plan hierarchy. Click a breadcrumb to jump to that plan.
- **Expand / Collapse all** — quickly expand or collapse all plan groups.
- **Duration bars** — visual bars on each step show relative time between consecutive activities.

### Error Handling
- **Error/exception banner** — a prominent banner lists all errors and exceptions found in the trace. Click an error to jump to the corresponding step.
- **Error highlighting** — error and exception activities are visually distinct with red styling.

### Export & Comparison
- **HTML export** — download a self-contained HTML file of the current trace view, preserving all expanded/collapsed states.
- **Side-by-side comparison** — load two traces to compare them side by side with independent stats, activity counts, and a diff summary.
- **Copy to clipboard** — each step has a copy button that copies structured JSON data for that activity.

### UI / UX
- **Dark theme** — easy on the eyes with a GitHub-inspired dark color palette.
- **Responsive layout** — works on desktop and mobile. On small screens, the chat pane stacks below the trace.
- **No dependencies** — pure HTML, CSS, and JavaScript in a single file. Works offline.

## Browser Support

Requires a modern browser with support for:
- `DecompressionStream` API (for ZIP extraction) — Chrome 80+, Edge 80+, Firefox 113+, Safari 16.4+
- CSS custom properties
- ES2017+

## License

MIT
