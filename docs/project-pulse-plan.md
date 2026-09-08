# Project Pulse dashboard implementation plan

## Summary

Build a small, deterministic static Project Pulse dashboard for contributors. The first view must clearly identify Project Pulse and show multiple project cards containing each project's name, owner, status, recent activity, priority or risk, and a short contributor-friendly summary. The implementation must use the existing repository conventions for a simple static app and must not add a framework or package dependency.

Repository inspection found that `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json` are currently absent. The `docs` directory exists, so this plan is the only file to create in this planning phase. The source files and launch configuration will be created later by the assigned specialists.

## File assignments

### Designer

- `app/styles.css`: own the visual system, layout, responsive behavior, focus states, color and contrast treatment, status badges, priority treatment, spacing, typography, rounded cards, and shadows.
- Provide design decisions to the Orchestrator and Coder before implementation: information hierarchy, card anatomy, narrow-screen behavior, status/priority affordances, and accessibility tradeoffs.
- Do not edit `app/index.html`, `app/project-data.json`, `.vscode/launch.json`, or this plan.

### Coder

- `app/index.html`: own the semantic page structure, exact `Project Pulse` title, stylesheet and data references, data loading/rendering behavior, visible project cards using the `project-card` class, and user-facing loading and error states.
- `app/project-data.json`: own valid deterministic fixture data with a top-level `projects` array. Every project object must include `name`, `owner`, `status`, `recentActivity`, and `priority`; include enough varied projects to exercise the card layout.
- `.vscode/launch.json`: own the strict JSON launch configuration named `Run Project Pulse Dashboard`. It must serve the `app` directory with `python -m http.server 5500`, use `cwd` `${workspaceFolder}/app`, and open `http://localhost:%s/index.html` through `serverReadyAction`.
- Do not edit `app/styles.css` except when the Orchestrator explicitly resolves an integration defect that cannot be fixed in markup or data.

### Orchestrator

- Own coordination, dependency ordering, integration review, and final validation; no implementation file assignment is implied.
- Resolve any conflict between the Designer's visual guidance and the Coder's semantic or runtime requirements before asking a specialist to edit another owner's file.

## Dependencies

- The repository brief is the source of truth for the required fields and launch behavior.
- Designer guidance is a dependency for polished styling and accessibility review, but the Designer and Coder can begin separate non-overlapping work after the plan is accepted.
- `app/project-data.json` is a runtime dependency of `app/index.html`; the page must fetch or otherwise load it using the exact relative reference `project-data.json`.
- `app/styles.css` is a presentation dependency of `app/index.html`; the page must reference `styles.css` with a relative stylesheet link.
- A local HTTP server is required for a reliable fetch of JSON. Opening `app/index.html` directly from the filesystem may produce a browser security error and is not a supported validation path.
- `.vscode/launch.json` depends on the final server command and app location, but does not depend on the visual design. It must be reviewed after the Coder has confirmed the app entry point.

## Ordered implementation steps

1. The Orchestrator confirms this plan, the absent-file constraint, and the non-overlapping file scopes. The Orchestrator also gives the Coder the Designer's decisions before any shared integration review.
2. The Designer defines and implements the polished dashboard stylesheet in `app/styles.css`. The stylesheet must include `.dashboard` and `.project-card`, readable spacing, responsive layout, `border-radius`, `box-shadow`, visible status badges, clear priority treatment, high-contrast text, and `:focus-visible` states.
3. In parallel with step 2, the Coder creates `app/project-data.json` with deterministic sample projects and creates `app/index.html` with the semantic shell and rendering logic. The Coder also creates `.vscode/launch.json` with the required static server configuration.
4. The Coder locally checks that the page references both `styles.css` and `project-data.json`, loads the top-level `projects` array, and renders one visible `project-card` for each valid project. Rendering must expose `status`, `recentActivity`, and `priority` values rather than only storing them in JavaScript.
5. After both specialists finish, the Orchestrator performs a sequential integration review. Confirm that class names used by the markup match selectors in `app/styles.css`, data keys match the renderer, the relative URLs work from the `app` working directory, and the launch URL names `index.html` rather than the directory root.
6. The Orchestrator runs the file, syntax, content, accessibility, and browser checks below. Any failure is assigned back to the owner of the failing file; the final launch smoke test is repeated after fixes.

## Explicit parallel-work decisions

- Designer work on `app/styles.css` and Coder work on `app/index.html`, `app/project-data.json`, and `.vscode/launch.json` may run in parallel because their file scopes do not overlap and the initial data shape is fixed by the brief.
- The Orchestrator may review the Designer's written guidance while the Coder creates data and launch configuration; this review does not authorize either specialist to edit the other specialist's files.
- Independent syntax checks for `app/project-data.json` and `.vscode/launch.json` may run in parallel after both files exist.

## Explicit sequential-work decisions

- The plan must be agreed before implementation so ownership and the absent-file constraint are unambiguous.
- Integration review must wait until Designer and Coder report completion; checking selectors or rendered cards before both files exist would create false failures.
- The browser smoke test must wait until `index.html`, `styles.css`, `project-data.json`, and `launch.json` are present and reviewed together.
- If markup changes a class, field, or DOM contract, the Orchestrator must have the Coder and Designer reconcile the change sequentially before final validation. Do not have both agents edit the same file concurrently.
- Launch configuration validation must follow confirmation of the actual `app/index.html` entry point so the server cannot accidentally open a directory listing.

## Implementation requirements

### Markup and runtime behavior

- Use a semantic document with `<!doctype html>`, `lang`, a useful `<title>Project Pulse</title>`, a `<header>` with an exact visible `Project Pulse` heading, a `<main>` content region, and a dashboard container with the `.dashboard` class.
- Render each project as an `<article>` or equivalent landmark with the `.project-card` class. Use headings for project names and structured labels or a definition list for owner, status, recent activity, and priority.
- Reference `styles.css` and `project-data.json` with relative paths. Use a deterministic browser-side loader and render the data into the page; do not hard-code cards as the only data source.
- Show a useful loading state, an actionable error state when the fetch fails or JSON is invalid, and an empty-state message when `projects` is empty. Do not silently render a blank dashboard.
- Insert fetched values as text, not unsanitized HTML. Missing or blank fields should display an explicit fallback such as `Not provided` while preserving the card layout.
- Keep JavaScript dependency-free and deterministic. Do not add build tooling, external APIs, fonts, analytics, or npm packages.

### Accessibility and responsive behavior

- Maintain heading order and landmark structure, ensure keyboard focus is visible, and do not use color as the sole indicator of status or priority.
- Use sufficient text/background contrast for normal text and badges, meaningful accessible text for any icons, and no required hover-only interaction.
- Make the grid readable on narrow screens by collapsing to one column as needed; prevent long names, activity text, or priority labels from causing horizontal scrolling.
- Preserve readable line height and spacing at desktop and mobile widths. The first viewport should communicate the dashboard purpose without requiring a user to discover hidden controls.
- Respect reduced-motion preferences if any transitions are added. Avoid unnecessary animation.

### Data contract

`app/project-data.json` must parse as JSON and have this shape:

```json
{
  "projects": [
    {
      "name": "Example project",
      "owner": "Example owner",
      "status": "Active",
      "recentActivity": "Recent contributor activity",
      "priority": "High"
    }
  ]
}
```

Use several representative records so active, complete or blocked status and at least two priority levels can be visually reviewed. Keep values contributor-friendly and deterministic; do not include secrets or personally sensitive information.

### Launch configuration

`.vscode/launch.json` must be strict JSON with no comments and must include a configuration named `Run Project Pulse Dashboard`. The configuration must run exactly `python -m http.server 5500`, set `cwd` to `${workspaceFolder}/app`, and use `serverReadyAction` with a capture pattern for the server port and `uriFormat` set to `http://localhost:%s/index.html`. The launch target must open the frontend entry point, not `http://localhost:5500/` or a directory listing. Use deterministic VS Code launch fields and do not add unrelated configurations.

## Edge cases and risks

- Direct `file://` opening can block the JSON fetch; validation must use the launch configuration or an equivalent local HTTP server.
- A missing, malformed, or non-object JSON response must show an informative error rather than throw an unhandled promise rejection.
- A missing `projects` key, non-array `projects`, an empty array, or records with missing fields must produce a stable empty/fallback presentation.
- Long project names, owners, activity strings, or priority labels must wrap without breaking the grid or hiding text.
- Status and priority combinations must remain understandable in grayscale, keyboard use, and color-vision differences.
- A slow server response should leave the loading state visible until data arrives; a failed response should not leave stale cards on screen.
- If the server port is occupied, the launch smoke test should report the launch failure and use a free port only for an independent manual diagnostic; the committed launch configuration must remain on port 5500.
- The committed launch command uses `python -m http.server 5500` so it works with the local Windows setup and remains available in the Codespaces environment.
- Relative asset paths must work with `cwd` set to `app`; paths that assume the repository root are an integration defect.

## Validation expectations

### Repository and syntax checks

- Confirm all four assigned files exist: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.
- Run `python -m json.tool app/project-data.json` and `python -m json.tool .vscode/launch.json`; both must succeed.
- Run the existing repository check `bash scripts/validate-exercise.sh` when the shell environment has the required tools. This is an exercise-level check and should remain green; it does not replace the app smoke test.
- Review the diff and confirm that only the assigned implementation files were changed during the implementation phase, with no secrets, generated directories, or package files.

### Content and contract checks

- Verify `app/index.html` contains the exact `Project Pulse` title, references `styles.css` and `project-data.json`, contains `.project-card` markup or the rendering template, and exposes `status`, `recentActivity`, and `priority`.
- Verify `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`, plus responsive and visible focus styling.
- Verify the JSON has a top-level `projects` array and that every fixture includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- Verify `.vscode/launch.json` contains `Run Project Pulse Dashboard`, `python -m http.server 5500`, `${workspaceFolder}/app`, `serverReadyAction`, and `http://localhost:%s/index.html`, with no comments.

### Browser and accessibility checks

- Start **Run Project Pulse Dashboard** from VS Code Run and Debug. Confirm the browser opens `http://localhost:5500/index.html` and shows the Project Pulse UI, not a server directory listing.
- Confirm multiple cards render from `project-data.json`, each card has a name and owner, and status, recent activity, and priority are visibly understandable.
- Resize to a narrow viewport and confirm cards reflow, text wraps, and no horizontal scrolling or clipped content occurs. Tab through the page and confirm focus is visible and the reading order is logical.
- Inspect the page for console errors, failed network requests, broken stylesheet/data references, missing accessible names, and unreadable badge contrast. Stop the preview server after the smoke test.

## Handoff

The Orchestrator should report the four files created, the Designer and Coder ownership, the validation commands and browser checks that passed, and any remaining environment limitation. No agent should stage, commit, or push changes; git operations remain under the learner's control.

## Open questions

- No product-specific branding, real project records, or framework requirement exists beyond the brief, so use neutral contributor-friendly sample data and a dependency-free implementation.
- If the environment cannot launch `python`, install or configure Python so the committed launch command remains consistent across local validation and Codespaces.
