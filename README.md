# Prompt Weaver

Prompt Weaver is a web app for building, testing, and reusing LLM prompt
templates that are driven by dynamic user input.

Instead of retyping a long prompt every time, you write it once as a reusable
template with `{field1}`-style placeholders. At run time you fill in a short
form, and the app weaves your values into the template, sends the assembled
prompt to the LLM, and shows the generated output next to the inputs that
produced it.

The app ships with two independent prompt slots — **Prompt 1** (backed by
input fields 1–3) and **Prompt 2** (backed by input fields 4–6) — so you can
keep two templates configured side by side and switch between them without
losing either one. The default Prompt 1 template is an example that drafts an
immigration/asylum hardship brief from three pieces of client information.

Prompt Weaver is a Next.js (App Router) application. Prompt execution runs
server-side through [Genkit](https://genkit.dev) against Google's Gemini
models, so the API key never reaches the browser.

## Features

### Prompt templating
- **Reusable prompt templates** — write a prompt once and run it repeatedly with different inputs.
- **Variable substitution** — reference form values inside a template with `{field1}` … `{field6}` placeholders; every occurrence is replaced before the prompt is sent, and unfilled variables resolve to an empty string.
- **Two configurable prompt slots** — Prompt 1 and Prompt 2 are stored and edited independently, each with its own multi-line setup editor.
- **Seeded example template** — Prompt 1 comes pre-filled with a working legal-brief prompt to demonstrate placeholder syntax.

### Dynamic input form
- **Prompt type selector** — a radio group chooses which template to run.
- **Context-aware fields** — selecting Prompt 1 reveals fields 1–3, Prompt 2 reveals fields 4–6; only the relevant inputs are ever shown.
- **Tabbed interface** — an *Inputs & Output* tab for day-to-day use and a *Prompt Setup* tab for editing templates, keeping authoring separate from running.

### Validation and error handling
- **Schema-backed validation** — a [Zod](https://zod.dev) schema paired with React Hook Form enforces the rules, validating on blur.
- **Conditional required fields** — only the fields belonging to the selected prompt are required, along with that prompt's setup text; blank values are rejected before any API call is made.
- **Inline field messages** — each input renders its own error message beneath it.
- **Toast notifications** — success, validation-failure, and generation-error toasts give feedback without scrolling.
- **Surfaced API errors** — failures from the model call are displayed in a dedicated alert in the output panel rather than being swallowed.

### LLM output
- **Server-side generation** — a Genkit flow assembles the final prompt and calls the model from the server.
- **Gemini-backed** — configured for `googleai/gemini-2.0-flash` via the Genkit Google AI plugin.
- **Inline output display** — results appear on the same tab as the inputs, in a scrollable, selectable read-only text area.
- **Loading state** — a spinner and disabled submit button indicate an in-flight request and prevent double submissions.

### Interface and styling
- **shadcn/ui component library** — an extensive set of accessible Radix UI–based primitives (dialogs, tabs, toasts, forms, tables, charts, and more).
- **Themed design system** — dark blue primary, light gray surfaces, and teal accents, defined as CSS custom properties with light and dark palettes.
- **Responsive layout** — the tabbed layout and field groups adapt from mobile to desktop widths.
- **Geist typography and Lucide icons** — clean sans-serif and monospace faces with simple iconography throughout.

## Tech stack

| Area | Technology |
| --- | --- |
| Framework | Next.js 15 (App Router, Turbopack dev server) |
| Language | TypeScript |
| UI | React 18, Tailwind CSS, shadcn/ui, Radix UI, Lucide icons |
| Forms | React Hook Form + Zod (`@hookform/resolvers`) |
| AI | Genkit (`genkit`, `@genkit-ai/googleai`, `@genkit-ai/next`) with Gemini |
| Other | TanStack Query, Firebase SDK, Recharts, date-fns |

## Getting started

### Prerequisites
- Node.js 20 or later
- A Google AI (Gemini) API key

### Install

```bash
npm install
```

### Configure

Create a `.env` file in the project root:

```bash
GEMINI_API_KEY=your-google-ai-api-key
```

The key is read server-side when Genkit is initialized (`src/lib/genkit.ts`);
it is not exposed to the client.

### Run

```bash
npm run dev
```

The app starts on [http://localhost:9002](http://localhost:9002).

To run the Genkit developer UI alongside the app for inspecting flows and
traces:

```bash
npm run genkit:dev     # or: npm run genkit:watch
```

### Other scripts

| Script | Purpose |
| --- | --- |
| `npm run dev` | Start the Next.js dev server on port 9002 |
| `npm run build` | Production build |
| `npm run start` | Serve the production build |
| `npm run lint` | Run Next.js linting |
| `npm run typecheck` | Type-check with `tsc --noEmit` |
| `npm run genkit:dev` | Start the Genkit developer UI |
| `npm run genkit:watch` | Genkit developer UI with file watching |

## Usage

1. Open the **Prompt Setup** tab and write your template in the Prompt 1 or
   Prompt 2 editor, using `{field1}`–`{field3}` for Prompt 1 and
   `{field4}`–`{field6}` for Prompt 2.
2. Switch to the **Inputs & Output** tab and choose the matching prompt type.
3. Fill in the revealed input fields.
4. Press **Submit to LLM**. Validation runs first; any problems are flagged
   inline and in a toast.
5. Read the generated text in the **LLM Output** panel below the form. Adjust
   the inputs or the template and run it again.

## Project structure

```
src/
├── ai/
│   ├── dev.ts                          # Genkit dev entry point
│   └── flows/
│       └── llm-output-generation.ts    # Flow: variable substitution + model call
├── app/
│   ├── layout.tsx                      # Root layout, fonts, toaster
│   ├── page.tsx                        # Main page: form state, tabs, submit handler
│   └── globals.css                     # Theme tokens and Tailwind layers
├── components/
│   ├── prompt-weaver/
│   │   ├── inputs-tab.tsx              # Prompt selector, dynamic fields, submit
│   │   ├── prompt-setup-tab.tsx        # Template editors
│   │   └── llm-output-display.tsx      # Output, loading, and error states
│   └── ui/                             # shadcn/ui primitives
├── hooks/                              # use-toast, use-mobile
└── lib/
    ├── genkit.ts                       # Genkit + Google AI initialization
    ├── schema.ts                       # Zod form schema and conditional rules
    └── utils.ts                        # Class-name helper
```

Additional design notes live in [`docs/blueprint.md`](docs/blueprint.md).
