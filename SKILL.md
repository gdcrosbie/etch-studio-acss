---
name: etch-studio-acss
description: Build and edit pages in Etch Studio through Etch Connector with Automatic.css (ACSS). Use for live Studio page structure, styling, responsive layouts, and simple interactions with an external AI coding agent.
---

# Build in Etch Studio with ACSS

Help the user turn a page brief or supplied design into native, editable Etch blocks, styled with the project's existing ACSS framework. This is a standalone skill for an external AI agent with terminal access. It needs no other locally installed skills or private project tools.

## Working with someone new to the workflow

Use British English in explanations, reports, and newly authored page copy unless the user requests another language variant. Preserve supplied copy unless editing it is in scope. Keep technical terms, product names, API identifiers, CSS properties, commands, URLs, and exact interface labels in their original form.

Explain the division of work once: the user opens the Studio page and connects its builder tab; you inspect it, write the scripts, build the page, and check the result. They do not need to write JavaScript or paste every build step into the browser console.

Establish the target page, intended change, content, assets, and design direction from the request and existing project. Ask only for missing choices that affect the result. If no design direction is supplied, propose one for the user to choose before committing to it. Reuse existing typography, colours, buttons, and layout conventions. Identify invented copy and placeholders as such.

Proceed with the requested page edits without seeking approval for every block. Surface changes that affect the whole project, shared components, global ACSS settings, external services, or publication separately. Building and saving a page does not imply permission to publish it. Keep progress explanations brief and describe what the user can see or review.

Example request to help a newcomer get started:

> Use Etch Connector to build the open Etch Studio page with ACSS. Create a hero, three service cards, and a contact CTA using the supplied copy and images. Follow the project's existing styling, keep the blocks editable, and check desktop and mobile. Save the page for review.

## Connect to the builder

Etch Connector is a local command-line bridge, not an MCP server. It links your terminal to a live builder tab using Etch's Public API. Missing MCP tools do not establish that the Connector is unavailable.

1. Check for a running bridge with `curl -s http://127.0.0.1:7332/tabs`.
2. If it is not running, start it in a persistent terminal process:

   ```bash
   npx @digital-gravy/etch-connector serve
   ```

   This requires Node.js/npm and may download the package on first use. If those are unavailable, explain the prerequisite; do not silently install a system toolchain.
3. Guide the user to the builder's Settings → AI → AI Connector switch, then the AI sparkles toolbar button to connect the tab. UI labels can change; consult current Connector documentation if the controls differ.
4. List registered tabs and use the actual target name:

   ```bash
   npx @digital-gravy/etch-connector tabs --json
   npx @digital-gravy/etch-connector eval -t "TAB_NAME" -f inspect.js
   ```

An empty `tabs` array means the relay is running but no builder tab has registered. Connection refused means the relay is down. Avoid starting duplicate relays. Connect one builder tab per project and keep it open. Never infer connection from a URL parameter or guess a tab name from its title.

## Inspect before writing

Connector eval runs an async function body. Use `await` and return a JSON-serialisable result. Prefer script files over inline shell JavaScript to avoid quoting, backtick, and interpolation errors. In normal safe mode, scripts can use `etch` and standard JavaScript; they cannot use `window`, `document`, `fetch`, or browser storage.

Start with a read-only inspection script:

```js
return {
  version: etch.version,
  apiVersion: etch.apiVersion,
  environment: etch.environment,
  activePage: etch.navigation.getActivePostId(),
  tree: etch.blocks.getTree(),
  styles: etch.styles.list(),
  sheets: etch.stylesheets.list(),
  guides: etch.skills.list()
};
```

Confirm that the environment identifies Etch Studio and that the active page is the intended target. If the target is WordPress, flag the mismatch instead of carrying this Studio workflow across unchanged. Record affected block IDs and existing styles before editing; preserve unrelated content and user edits.

Read the host's `etch-studio` authoring guide when listed, using `await etch.skills.get("etch-studio")`. Read the relevant bundled guides and advertised references for unfamiliar features. The API is experimental: check `etch.environment.capabilities`, supported `blockTypes`, method availability, and current documentation rather than guessing from another builder or relying on a version number alone.

When navigating to another page, resolve its ID through available navigation methods. Perform navigation in one eval and inspect the active page and tree in a subsequent eval. Immediate reads after navigation can still reflect the previous page. Do not save unrelated pending work just to navigate; resolve that state with the user if needed.

## Resolve ACSS from this project

Read the generated tokens before choosing CSS values:

```js
return await etch.skills.getReference("acss", "tokens.md");
```

First check that `acss` and that reference are listed. Read relevant advertised references such as spacing, colours, typography, grids, or buttons as needed. If the token reference is unavailable, find the ACSS Framework sheet through `etch.stylesheets.list()` and read its generated CSS using the discovered ID. Do not hardcode a stylesheet ID or infer token availability from settings alone.

Verify every token and framework class used. Names such as `--space-m`, `--section-space-l`, `--content-width`, `--grid-gap`, `--primary`, `btn--primary`, and `bg--dark` are examples to check, not a guaranteed catalogue. Enabled families, radius steps, breakpoints, and utilities vary between projects. Do not invent `--color-primary` by analogy with another framework.

Use the generated framework as the source of truth. Treat its sheet as read-only. A missing token is a reason to choose an existing appropriate token or discuss a project setting change; adding a fallback value does not fix an incorrect token name.

## Build native, semantic structure

Use `etch/element` for HTML elements and `etch/text` for their text content. Keep headings, links, buttons, images, and sections separately editable. Do not hide an ordinary page inside a raw HTML block, iframe, or injected application.

Inspect whether the current Studio project has a template or shared layout providing landmarks. If it does not, include a page header/footer where needed, one primary `<main>`, and a working skip link targeting that main. Avoid duplicate landmarks and IDs. Use a coherent heading hierarchy and an appropriate page-level heading.

For standard section/container layouts, API-created blocks need explicit `data-etch-element="section"` and `data-etch-element="container"` attributes to participate in Etch's primitive styling. Inspect the generated Etch and ACSS defaults before adding padding, gaps, or widths: their combined rules may already provide them. Avoid double gutters. Omit primitive attributes deliberately when authoring a custom full-bleed layout that should not inherit those defaults.

For `blocks.create()`, include `version: 1`, `context: {}`, and `children: []`, including on text nodes and void elements. Use the returned ID as the parent of subsequent blocks. Do not supply the read-only `styles` array to page block creation; attach classes through `attributes.class` or `etch.blocks.addClass()`.

The following illustrates a section inside an existing main. Replace `MAIN_BLOCK_ID` with a verified parent ID. Use it only when this section is actually requested:

```js
const section = etch.blocks.create({
  type: "etch/element", version: 1, context: { name: "Services" },
  children: [], tag: "section",
  attributes: { class: "services", "data-etch-element": "section" }
}, "MAIN_BLOCK_ID");

const container = etch.blocks.create({
  type: "etch/element", version: 1, context: {}, children: [], tag: "div",
  attributes: { "data-etch-element": "container" }
}, section);

const heading = etch.blocks.create({
  type: "etch/element", version: 1, context: {}, children: [], tag: "h2",
  attributes: { class: "services__title" }
}, container);

etch.blocks.create({
  type: "etch/text", version: 1, context: {}, children: [],
  text: "Our services"
}, heading);

await etch.saveAsync();
return { section, container, heading };
```

HTML in a `text` value is escaped; create actual element blocks for markup. Void elements such as `img` must have no child nodes. Use links for navigation and buttons for actions. Give informative images meaningful `alt` text and decorative images `alt=""`.

## Style with ACSS and editable class records

Follow the project's class convention. If none exists, descriptive component classes such as `.services` and `.services__title` are a suggested BEM convention, not an Etch requirement. Do not impose a personal or brand prefix.

Put custom selector-scoped CSS in Etch's class style records through `etch.styles`. Prefer verified ACSS variables for spacing, typography, colours, widths, and radii. Reuse ACSS button classes and contextual background classes where available. Background relationship classes can set appropriate descendant text and heading colours; avoid overriding those with a global dark heading token inside a dark section.

Adding a class can create an empty style record automatically. Find and update that record instead of creating duplicates:

```js
const selector = ".services__grid";
// Verify --grid-gap on the project before using this CSS.
const css = "display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: var(--grid-gap);";
const existing = etch.styles.list().find(style => style.selector === selector);
if (existing) etch.styles.update(existing.id, { css });
else etch.styles.create(selector, css);
await etch.saveAsync();
```

`styles.create(selector, css)` takes positional arguments. `styles.update(id, { css })` takes a patch object. A style record's CSS is the declarations and nested rules belonging to that selector, without another outer copy of the selector. Inspect existing CSS before replacing it; classes may be shared by other pages.

Use global stylesheets for document-level needs such as `@font-face`, `@keyframes`, or `@property`, with a clear purpose and scoped selectors where applicable. Avoid a catch-all sheet holding the entire page's component CSS. Do not change global ACSS configuration or root variables as a shortcut for one section's styling.

Choose responsive changes from the content and supplied design. Use fluid dimensions, grid/flex layouts, logical properties, and verified framework breakpoints where appropriate. Check intermediate widths as well as desktop and mobile. Never hide layout errors with page-wide overflow clipping.

## Assets, reusable content, and interactions

Use supplied or existing project assets; get approval before introducing paid assets or external services. Studio's Asset Manager is not a WordPress media library. Do not invent an `etch.assets` API or attachment IDs. Use documented capabilities or the existing Asset Manager UI for uploads.

Check asset paths in both Studio preview and the intended published output. Authenticated absolute Studio asset URLs may fail on the published site. Published assets commonly use `/assets/filename`; the preview may need a project-prefixed path. Verify the actual paths rather than assuming either works everywhere. Store image alt text on the image block or its source data.

Reuse an existing component when it fits. Create a reusable component when repetition or future editing justifies it; a single static section does not require abstraction. Read the current component schema before authoring definitions: component block JSON and page block creation can have different requirements.

For repeated data in Studio, consult its current guide for `etch.dataSources` and `etch/loop`, commonly with `target: "data('key')"` and an `itemId`. Do not reach for WordPress custom fields, PHP, query loops, or REST endpoints. Verify the data source shape and template expression syntax before writing them.

Prefer native HTML interactions, such as `details`/`summary` for simple disclosures. Add JavaScript only when the requested behaviour needs it. Runtime JS belongs in a block's documented `script.code` surface, not a raw HTML script tag. Runtime scripts can use the DOM even though Connector safe-mode eval cannot. Scope initialisation to each instance, avoid duplicate listeners, preserve keyboard behaviour, and respect reduced motion. Keep runtime state selectors such as `&.is-open` nested in an emitted class record so usage-based CSS output does not drop them.

A form needs an actual submission destination. Do not present decorative inputs or a success message as a working contact form. Establish the user's chosen submission method before adding a service.

## Save and recover carefully

Page block and class-style changes are buffered: finish a successful mutation batch with `await etch.saveAsync()`. Stylesheets and component definitions persist immediately through their async mutation methods. Studio data source mutations may also persist independently; follow the current host guide. Page undo or a later save is not a transaction covering all namespaces.

Read-only inspection scripts should not call `saveAsync()`. It can commit pending changes unrelated to the inspection.

Build in small coherent batches and return the changed IDs. After an error or timeout, inspect the tree and styles before retrying: earlier mutations may already exist even if the final save did not run. Reconcile or remove only artefacts from your failed attempt. Never clear the whole page to recover from a partial failure unless replacement of that page is explicitly in scope.

If the API rejects a patch or block type, read the supported schema rather than retrying guessed keys. If authentication or connection expires, explain how to reconnect and preserve pending work before suggesting a reload. Do not claim that a failed request left the project unchanged.

## Verify what the page renders

A successful eval and save establish that Etch accepted the changes; inspect the actual rendered page to verify the result. Use an available browser inspection tool, Studio preview, or Connector CDP inspection. CDP requires a compatible browser debugging session; it is optional, not a reason to reconfigure the user's normal browser without discussion.

Typical Connector inspection commands, when CDP is configured:

```bash
npx @digital-gravy/etch-connector shot --cdp -t "UNIQUE_TARGET" -s ".services" -o services.png
npx @digital-gravy/etch-connector html --cdp -t "UNIQUE_TARGET" ".services"
npx @digital-gravy/etch-connector computed --cdp -t "UNIQUE_TARGET" ".services" --props display,padding,gap
```

Read the resulting screenshot. Target the content document rather than the builder toolbar; browser inspection may need to enter the canvas iframe. Refresh a stale preview before concluding a saved change failed.

Check layout, text wrapping, asset loading, contrast, visible focus, keyboard interaction, links, and horizontal overflow at desktop, mobile, and an intermediate width. Compare with supplied design evidence when present. Test real interaction outcomes, including form submission when configured and authorised. If publishing is requested, check the published page as well; preview success does not prove published asset paths or runtime behaviour.

Report what was built, what was saved, what was visually or functionally checked, and any remaining placeholders or limitations. If render inspection is unavailable, state that the page is saved but visually unverified and give the user a focused review task.

## Authoritative references

Use the live host guides for Studio-specific behaviour and generated ACSS tokens. For unfamiliar or changed APIs, consult the relevant current official documentation:

- [Connector usage](https://docs.etchwp.com/etch-connector/usage)
- [Connector AI agent guide](https://docs.etchwp.com/etch-connector/ai-agent-guide)
- [Etch Public API](https://docs.etchwp.com/public-api)
- [Automatic.css documentation](https://docs.automaticcss.com/)

These references supplement this single file; they do not require another local skill. Documentation and examples can lag the live runtime. Resolve disagreements through generated CSS, supported API schemas, and observed rendering.
