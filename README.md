# Copilot Studio Trace Viewer

A single-page web application for exploring and analyzing conversation traces exported from [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/). Upload a `dialog.json` (or its `.zip` archive) and navigate through every step of the conversation to understand what happened and why.

![Dark themed trace viewer](https://img.shields.io/badge/theme-dark-0d1117) ![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen) ![Single file](https://img.shields.io/badge/single%20file-index.html-blue)


<img width="2524" height="1232" alt="image" src="https://github.com/user-attachments/assets/703ef96d-b678-44ae-8cb3-e27c543496c1" />

<img width="2512" height="1219" alt="image" src="https://github.com/user-attachments/assets/b016c5f7-e631-4c02-be5f-3a26ba10f900" />


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
- **Upload dialog.json or .zip** — drag-and-drop or click to upload. ZIP files are parsed natively in the browser (no libraries). When a `.zip` contains both `dialog.json` and `botContent.yml`, both are parsed automatically.
- **Activity timeline** — every activity is displayed chronologically with color-coded badges (user message, bot message, plan, thought, tool call, search, info, typing, error).
- **Expandable details** — click any step to expand and see full metadata: timestamps, parameters, observations, raw JSON auto-formatted as key-value pairs.
- **Friendly topic names** — when `botContent.yml` is available, raw schema names (e.g. `copilots_header_5948b.topic.Greeting`) are replaced with human-readable display names throughout the timeline.

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
- **Batch analysis** — upload multiple trace files at once to get aggregate statistics across conversations.

---

### Analysis Panels

Five collapsible analysis panels appear below the trace timeline when relevant data is present. Each panel can be expanded or collapsed by clicking its header.

#### Performance Waterfall

A horizontal bar chart showing the timing of every trace activity. Each bar's width represents the time gap between consecutive activities.

- Activities are sorted by timestamp and displayed as proportional bars against the total conversation duration.
- Color-coded by category: bot messages (green), user messages (blue), plan events (purple), tool steps (orange), search (teal), errors (red).
- Hover over any bar to see exact timestamps and duration.
- Helps identify bottlenecks — long bars indicate slow steps (e.g. a tool call that took several seconds).

#### Knowledge Sources

Extracts and displays all knowledge retrieval data found in the trace, gathered from `GenerativeAnswersSupportData` activities.

- **Search results** — shows all search results returned by knowledge sources, including URL, snippet text, search type (e.g. `DataverseStructuredSearch`, `BingSearch`), and rank score.
- **Verified search results** — highlights which results passed verification.
- **Query rewriting** — displays the original user question, the AI-rewritten query, extracted keywords, and hypothetical snippet used for retrieval.
- **Summarization** — shows the AI's raw summary, final summary, and citation mappings.
- **Token usage** — displays prompt and completion token counts for both query rewriting and summarization LLM calls.
- **Model information** — shows which model was used (e.g. `gpt-41-2025-04-14`) and the partner source.

#### Citation Verification

Appears alongside the Knowledge Sources panel when citations are present in bot responses.

- Maps each citation number to its source document with title, URL, and snippet text.
- Shows whether content provenance and content moderation checks were performed.
- Displays the `gptAnswerState` (e.g. `Answered`, `NotAnswered`) and `completionState`.

#### Topic Flow

A visual flow diagram showing the execution path through the conversation. Displays the sequence of orchestrator plans, topic invocations, and agent handoffs.

- **Plan nodes** — each orchestrator plan is shown as a node with its plan ID.
- **Step nodes** — individual steps (topics, tools, actions) are shown with their friendly names and types.
- **Agent handoffs** — Connected Agent and External Agent (Azure Foundry) invocations are shown as distinct nodes with handoff edges.
- **Dialog tracing** — `DialogTracingInfo` actions (e.g. `SendActivity`, `BeginDialog`) are shown with their topic context.
- **Edge labels** — connections between nodes show the relationship type (step, handoff, return).
- **Plan-step correspondence** — each step node is linked to its parent plan for visual grouping.

#### Variable Tracker

Tracks how variables are declared, bound, and populated across the conversation. Builds a consolidated view from multiple trace event types.

- **Input/output declarations** — variables declared in `DynamicPlanReceived.toolDefinitions` are listed with their names, types (Boolean, String, etc.), descriptions, and directions (input/output).
- **Argument bindings** — values assigned via `DynamicPlanStepBindUpdate` are shown with their bound values. Auto-filled arguments are tagged with an **AUTO** badge; manually set arguments show **MANUAL**.
- **Output harvesting** — output variables recovered from `DynamicPlanStepFinished.observation` are shown with a *(step completed)* label.
- **Per-step grouping** — variables are grouped by the topic/step that declared or used them, with friendly topic names when `botContent.yml` is available.

#### Agent Definition Panel (botContent.yml)

When a `.zip` file contains `botContent.yml`, this panel displays the complete agent configuration extracted from the YAML definition. Starts collapsed by default.

- **Agent Overview** — schema name, authentication mode (e.g. `Integrated`), template version, and recognizer type (e.g. `GenerativeAIRecognizer`).
- **AI Settings** — toggles for model knowledge, file analysis, semantic search, content moderation level, and opt-in for latest models.
- **Agent Instructions** — the full system prompt / instructions configured for the GPT component, displayed in a scrollable pre-formatted box.
- **GPT Capabilities** — web browsing, code interpreter, and other capability flags.
- **Topics Table** — all topics in the agent listed with display name, schema name, component type, trigger phrases, and authored action count.
- **Global Variables** — any `GlobalVariableComponent` entries with their types and scopes.
- **Connectors** — connector definitions with display name, operation count, and whether they are custom connectors.

---

### Enriched Step Details

When `botContent.yml` is loaded, individual trace steps are enriched with additional context:

#### DynamicPlanReceived (Plan Details)
- **Tool definitions** show display name (with icon if available), description, schema, and a color-coded **tool-kind badge**:
  - **Connected Agent** (blue) — `InvokeConnectedAgentTaskAction`
  - **External Agent / Foundry** (purple) — `InvokeExternalAgentTaskAction`
  - **Topic** (green) — `AdaptiveDialog`
- Tool inputs and outputs are shown with their types (Boolean, String, etc.).

#### DynamicPlanStepTriggered (Step Details)
- **Topic Name** — resolved from `botContent.yml` and highlighted in accent color.
- **Description** — topic description from the agent definition.
- **Invocation Type** — badge showing whether the step invokes a Connected Agent (Copilot-to-Copilot) or an External Agent (Azure AI Foundry).
- **Connection Reference** — for external agent calls, the connection reference ID is shown.
- **AI Thought / Reasoning** — the orchestrator's reasoning for selecting this step.
- **Trigger Phrases** — from the agent definition, showing all phrases that can trigger this topic.
- **Authored Flow** — a visual representation of the topic's authored action sequence (SendActivity, Question, SetVariable, BeginDialog, etc.).

#### DynamicPlanStepBindUpdate (Binding Details)
- Each argument shows an **AUTO** or **MANUAL** badge indicating whether it was auto-filled by the orchestrator or explicitly provided.

#### DynamicPlanStepFinished (Step Result)
- **Topic Name** resolved from `botContent.yml`.
- **Execution Time** and **State** with color coding (green for completed, red for failed).

#### DialogTracingInfo
- Action types shown with friendly topic names instead of raw schema identifiers.

### UI / UX
- **Dark theme** — easy on the eyes with a GitHub-inspired dark color palette.
- **Responsive layout** — works on desktop and mobile. On small screens, the chat pane stacks below the trace.
- **No dependencies** — pure HTML, CSS, and JavaScript in a single file. Works offline.
- **Connected agent context** — activities from connected agents are tracked and labeled with the agent name throughout the timeline and flow panel.

---

## Loading Transcripts from Dataverse (Channel Conversations)

In addition to uploading exported snapshots, you can connect directly to your Dataverse environment to browse and load conversation transcripts from **any channel** — Teams, Web Chat, Facebook, Telephony, DirectLine, etc.

This lets you troubleshoot conversations that happened outside of the Copilot Studio test chat, using data stored in the `ConversationTranscript` Dataverse table.

### What you'll see

Dataverse channel transcripts include:
- **Full conversation flow** — every user message, bot response, and adaptive card
- **DialogTraceDetail** — which topics were triggered and how the dialog navigated
- **DialogErrorDetail** — any errors that occurred during the conversation
- **Variable assignments** — variable values set throughout the conversation
- **Intent recognition** — what the system understood at each user turn
- **Topic redirects** — when and where the conversation switched topics, with **automatic friendly name resolution** from the Dataverse `botcomponents` table (no manual upload needed)
- **Session info** — engaged/unengaged status, outcome (Resolved/Escalated/Abandon), turn count
- **CSAT data** — survey requests and responses
- **Node-level trace data** (if [Enhanced Transcripts](#enable-enhanced-transcripts-optional) are enabled) — per-node execution with timing

> **Note:** Channel transcripts do **not** include the deep orchestrator trace data available in test chat snapshots (planner decisions, raw LLM prompts, connector HTTP details, AI Builder internals, token counts). The viewer gracefully handles this — panels that require that data simply won't appear.

### Prerequisites

- A **Microsoft Entra ID (Azure AD) App Registration** configured as a Single-Page Application
- The **Dynamics CRM `user_impersonation`** API permission granted to that app
- Your user account must have **Read** access to the following Dataverse tables:

| Dataverse Table | API Entity Name | Purpose |
|---|---|---|
| **Bot** | `bots` | Populate the agent filter dropdown |
| **Conversation Transcript** | `conversationtranscripts` | List, filter, and load conversation transcripts |
| **Bot Component** | `botcomponents` | Auto-resolve component GUIDs to friendly topic/dialog names |

  The built-in **Bot Transcript Viewer** security role covers the first two tables. For `botcomponents` access, your admin may need to add Read privilege on the **Bot Component** table to your security role (see Step 7 below).
- A modern browser (the same browsers that support the trace viewer)

### Step-by-step setup

Follow every step below. If you've never worked with Azure / Entra ID before, don't worry — this is a one-time setup that takes about 5 minutes.

#### Step 1 — Open the Azure portal

1. Go to [https://portal.azure.com](https://portal.azure.com) and sign in with your work or school account (the same account you use for Copilot Studio / Power Platform).

#### Step 2 — Navigate to App Registrations

1. In the search bar at the top, type **App registrations** and select it from the results.
2. Click **+ New registration**.

#### Step 3 — Register a new application

1. **Name**: Enter a descriptive name, e.g. `Copilot Studio Trace Viewer`.
2. **Supported account types**: Select **Accounts in this organizational directory only** (single tenant) — unless you need to use this across multiple tenants, in which case choose **Accounts in any organizational directory**.
3. **Redirect URI**:
   - In the **platform** dropdown, select **Single-page application (SPA)**.
   - In the URL field, enter the URL where you access the trace viewer. Examples:
     - If you open the file locally: `http://localhost` (or `http://127.0.0.1`)
     - If hosted on GitHub Pages: `https://yourusername.github.io/CopilotStudioTraceViewer/`
     - If hosted elsewhere: the exact URL of the page
4. Click **Register**.

#### Step 4 — Copy the Application (Client) ID

1. After registration, you'll land on the app's **Overview** page.
2. Find the **Application (client) ID** — it looks like `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.
3. Find the **Directory (tenant) ID** — right below the client ID on the same page.
4. **Copy both values** — you'll paste them into the Trace Viewer later.

#### Step 5 — Add the Dataverse API permission

1. In the left sidebar, click **API permissions**.
2. Click **+ Add a permission**.
3. In the panel that opens, scroll down and select **Dynamics CRM** (it may also appear as "Dataverse" or "Common Data Service").
4. Select **Delegated permissions**.
5. Check the box next to **user_impersonation**.
6. Click **Add permissions**.
7. *(Optional but recommended)* Click **Grant admin consent for [your org]** if you have admin rights. This avoids each user having to individually consent. If you don't have admin rights, each user will see a one-time consent prompt on first login.

#### Step 6 — Verify the Redirect URI (if needed)

If you skipped the redirect URI during registration, or need to change it:

1. In the left sidebar, click **Authentication**.
2. Under **Single-page application**, check that the correct redirect URI is listed.
3. If not, click **Add URI**, enter the correct URL, and click **Save**.

> **Important:** The redirect URI must **exactly match** the URL in your browser's address bar when using the Trace Viewer (including trailing slashes). If they don't match, login will fail with a redirect error.

#### Step 7 — Configure Dataverse security roles

Your user account needs permission to read conversation transcripts and bot components in Dataverse.

**Option A — Bot Transcript Viewer role (minimum for transcripts)**

1. Go to the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/).
2. Select your **environment**.
3. Click **Settings** → **Users + permissions** → **Security roles**.
4. Find and open the **Bot Transcript Viewer** role.
5. Add your user account to this role.

This grants Read access to the `bots` and `conversationtranscripts` tables.

**Option B — Add Bot Component read access (recommended)**

To enable automatic topic name resolution (so `DialogRedirect` traces show friendly names like "Greeting" instead of raw GUIDs), your security role also needs **Read** on the **Bot Component** (`botcomponent`) table:

1. In the Power Platform Admin Center, go to **Settings** → **Users + permissions** → **Security roles**.
2. Open the security role assigned to your user (e.g. **Bot Transcript Viewer** or a custom role).
3. On the **Custom Entities** tab (or search for "Bot Component"), set the **Read** privilege to **Organization** level.
4. Save the role.

> **Note:** If the Bot Component permission is not granted, the viewer still works — topic redirects will simply show the raw GUID instead of the resolved name.

Alternatively, an admin can assign the role while [sharing the agent](https://learn.microsoft.com/en-us/microsoft-copilot-studio/admin-share-bots#assign-the-bot-transcript-viewer-security-role-during-agent-sharing).

#### Step 8 — Connect in the Trace Viewer

1. Open `index.html` (or `index-dataverse.html`) in your browser.
2. Scroll down to the **"Load transcripts from Dataverse"** section.
3. Enter your **Environment URL** — this is the URL of your Dataverse environment, e.g. `https://yourorg.crm.dynamics.com`. You can find it in the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/) → Environments → your environment → Environment URL.
4. Enter the **Application (Client) ID** you copied in Step 4.
5. Enter the **Directory (Tenant) ID** you copied in Step 4.
6. Click **Connect**.
6. A Microsoft login popup will appear — sign in with your work account and accept the consent prompt (first time only).
7. Once connected, the transcript browser will load showing your recent conversations.

#### Step 9 — Browse and load transcripts

1. Use the **filters** to narrow down transcripts:
   - **Agent** — select a specific agent/bot
   - **Channel** — filter by Teams, Web Chat, DirectLine, Facebook, Telephony
   - **Date range** — set a from/to date
2. Click any transcript in the list to load it into the viewer.
3. The viewer will display the conversation with all available trace data.

### Enable Enhanced Transcripts (optional)

For richer diagnostics, you can enable **Enhanced Transcripts** in Copilot Studio. This adds node-level execution data to the transcripts stored in Dataverse.

1. Open your agent in [Copilot Studio](https://web.powerva.microsoft.com/).
2. Go to **Settings** → **Advanced**.
3. Under **Enhance Transcripts**, turn on **Include node-level details in transcripts**.

When enabled, each node that a topic invokes will include `nodeTraceData` with the node ID, type, start/end timestamps, and topic name.

### Important notes

- **Transcripts are saved after 30 minutes of inactivity** — they are not real-time. If a conversation is still in progress, it won't appear yet.
- **Default retention is 30 days** — a bulk delete job automatically removes transcripts older than 30 days. To keep them longer, follow [these instructions](https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-transcripts-powerapps#change-the-default-retention-period) to modify the retention period.
- **Not available for**: Dataverse for Teams environments, developer environments, or Microsoft 365 Copilot agents.
- **Large transcripts** (>1 MB) are automatically split across multiple Dataverse records. The viewer detects this and merges them back together.
- **Your credentials stay in your browser** — the Environment URL and Client ID are saved in `localStorage` for convenience. Tokens are stored in `sessionStorage` and cleared when you close the tab. No data is sent to any third-party server.

### Troubleshooting the Dataverse connection

| Problem | Solution |
|---|---|
| **"popup_window_error"** | Your browser is blocking popups. Allow popups for the page, or the viewer will fall back to a redirect-based login. |
| **"AADSTS50011: The redirect URI does not match"** | The redirect URI in your App Registration doesn't match your browser URL. Go to Azure Portal → App Registrations → Authentication and add the exact URL. |
| **"AADSTS65001: The user or administrator has not consented"** | Ask your admin to grant admin consent for the Dynamics CRM permission, or click the consent prompt when it appears. |
| **"HTTP 403: Forbidden"** | Your account doesn't have the required security role. Ask an admin to assign the **Bot Transcript Viewer** role and optionally add **Read** on the **Bot Component** table (see Step 7). |
| **No transcripts appear** | Check that transcripts are enabled for your environment (Settings → Advanced in Copilot Studio). Also check your date filters and that conversations are at least 30 minutes old. |
| **Connection works but "No activities found"** | The transcript record exists but its content is empty or in an unexpected format. This can happen with very old transcripts or certain channel types. |

---

## Known Limitations

### Variable Tracker — Connected Agent Topics

When a connected agent invokes a topic, some variables declared in `DynamicPlanReceived.toolDefinitions` (e.g. `isAnsweredByTopic`, `questionAnswer`) may show as **unset** in the Variable Tracker. This happens because:

- The `DynamicPlanStepBindUpdate` for connected agent topics can have empty `arguments: {}`, so input variable bindings are not available in the trace.
- Variables like `isAnsweredByTopic` and `questionAnswer` only appear in the bot's message text (e.g. *"the variable isAnsweredByTopic is: Yes"*), not in any structured trace field. Parsing natural language to extract variable values would be unreliable and is intentionally not attempted.
- **Output variables** (e.g. `outputVariable`) **are** recovered from `DynamicPlanStepFinished.observation` and shown with a *(step completed)* label in the tracker tooltip.

## Browser Support

Requires a modern browser with support for:
- `DecompressionStream` API (for ZIP extraction) — Chrome 80+, Edge 80+, Firefox 113+, Safari 16.4+
- CSS custom properties
- ES2017+

## License

MIT
