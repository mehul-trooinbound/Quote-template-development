---
name: quote-custom-modules-dev
description: Use when building, extending, or converting a HubSpot CPQ quote, proposal, or agreement design (screenshot, HTML, PDF, or spec) into a custom React quote module. Analyzes the file first, asks at most one short round of questions if needed, sets up the quote-dev-starter project if missing, then plans and builds the module with local preview and deploy steps.
---

# Custom Quote Modules for HubSpot CPQ

## Overview

This skill covers building custom React modules for HubSpot CPQ Quotes. The workflow enables rapid visual development with local preview before deploying to HubSpot.

The starter project lives at [HubSpot/quote-dev-starter](https://github.com/HubSpot/quote-dev-starter). Type definitions for `quoteTemplateContext` are published in [`@hubspot/quote-dev-sdk`](https://www.npmjs.com/package/@hubspot/quote-dev-sdk).

## Intake: Analyze, Clarify, Plan (RUN THIS FIRST)

This skill is self-contained. Do NOT invoke brainstorming, planning, or any other skill. Do the whole intake here, briefly, to save time and tokens.

When the user selects this skill with a file (screenshot, HTML, PDF, spec, or existing module), follow these phases in order before writing any code.

### Phase 0: Project setup (only if no starter project exists)

Check the working directory for `hsproject.json` and `src/cms-assets/my-react-assets/`. If they are missing, the user has no starter project, so set one up yourself without asking:

```bash
git clone https://github.com/HubSpot/quote-dev-starter .
npm install
```

- If the directory is not empty, clone into a subfolder named `quote-dev-starter` instead and work inside it.
- `npm install` runs a postinstall step that installs dependencies in `src/cms-assets/my-react-assets/`.
- Creating quote-module projects is not supported by the HubSpot CLI scaffolding commands, so the starter repo is the only supported way to start.
- Prerequisites to mention only if a step fails: HubSpot CLI installed and authenticated (`hs auth`), and a Revenue Hub Professional or Enterprise account.
- Reference docs (fetch only if something in this skill is insufficient or seems outdated): https://developers.hubspot.com/docs/cms/start-building/building-blocks/modules/quotes/create-quote-modules

**Clean up the clone (only for a project you just cloned):** the starter ships example code that must not end up in the user's template project. After the real modules exist (use `QuoteExampleModule` as the scaffold first, then remove it):
- Delete sample-only modules and islands, for example `QuoteExampleModule/` and `islands/InteractiveButton.tsx`, unless the design actually needs that island.
- Delete sample mock data, demo text, and unused example files.
- Keep only what the template needs: `hsproject.json`, root and asset `package.json`, `cms-assets.json`, `tsconfig.json`, `Globals.d.ts`, `preview/`, and the modules and shared code for this design.
- Only delete files that came from the fresh clone and are not referenced by the new modules. Never delete files the user created. Confirm nothing imports a file before removing it, then tell the user in one line what was removed.

**Rename the project (only for a project you just cloned, do it right after the clone and before the first upload):** the starter is called `quote-dev-starter`. Give it a name that matches the user's project or requirement, so it is easy to recognise in HubSpot and on disk.

1. **Choose the name.** Ask for it in Phase 2 (question: "Project name?"). If the user does not answer, build it as `<client or brand>-<document type>` from the request, for example `goodwood-proforma-invoice`, `acme-service-proposal`, `northwind-agreement`. If there is no client or brand in the request, use the document type and year, for example `proforma-invoice-2026`.
2. **Format.** Lowercase letters, numbers, and hyphens only. No spaces, no underscores, no dots, no emoji. Keep it short and readable. It must NOT be `cpq-theme` (reserved by HubSpot).
3. **Replace the old name everywhere it appears.** Search the clone for `quote-dev-starter` (exclude `node_modules` and `.git`) and change each hit that is the project's own name:
   - `hsproject.json`: `name` (this is the HubSpot project name, the most important one)
   - Root `package.json`: `name`
   - `src/cms-assets/my-react-assets/package.json`: `name`
   - Both `package-lock.json` files: the top-level `name` fields (or run `npm install` after changing the `package.json` files so the lock files update)
   - `README.md`: the title and any mention, plus a one-line description of the real project
   - `.claude/` skill or launch files that mention it
   - The `label` in `src/cms-assets/my-react-assets/cms-assets.json` (`react-modules-only`) can stay, or be renamed to the project name
   Do NOT touch the `@hubspot/quote-dev-sdk` dependency, URLs of the original GitHub repo that you are citing as a source, or anything inside `node_modules`.
4. **Rename the folder only if you cloned into a subfolder named `quote-dev-starter`.** Rename it to the project name, and tell the user the new path. If the clone went into the user's existing working directory, do not rename that directory.
5. **Reconnect git (if the user wants to push their own copy):** the clone's `origin` still points to HubSpot's starter repo. Tell the user, and offer to remove it or point it to the user's own repository. Never push to HubSpot's repository.
6. **Use the name in the module labels.** The "App name" in the Module Naming Rule comes from the project name (title case, hyphens turned into spaces). Set the project name first, then name the modules.
7. **Verify.** Search again for `quote-dev-starter` and confirm no project-name hit is left. Show the user a one-line summary: old name, new name, files changed.
8. **Never rename a project that is already deployed.** HubSpot treats a different `name` in `hsproject.json` as a different project: the next upload creates a second project and leaves the old one (and its modules) behind in the account. Rename only before the first upload. If the user truly wants a new name for a deployed project, explain this and let them decide.

### Phase 1: Analyze the file, then report to the user

Read the provided file once and note:
- Sections and layout, colors, fonts, spacing, and content
- Which data is dynamic (quote, deal, line items, contacts, company, signers) and which is static
- Interactive parts that need an Island (tabs, toggles, sliders, accordions, e-sign)
- Whether an existing module should be extended, or a new one created
- Anything unclear, contradictory, or not supported (for example a classic HubL module, a missing asset, unreadable text, a field that does not exist in `quoteTemplateContext`)

Check the existing `components/modules/` folder first so you reuse naming, fields, and shared helpers instead of asking about them.

Then show the user an **Analysis Report** (one compact table, plus a short list of findings). For every module and every data-bearing element (tables, lists, totals, names, dates, images, repeated blocks), state where the data would come from:

| Module | Element | Looks like | Proposed data source | Sure? |
|---|---|---|---|---|
| Line Items | Product table | Table of products, qty, price | `quoteTemplateContext.lineItems` (automatic, from quote line items) | Yes |
| Content | Scope list | Bullet list of deliverables | Editor field (repeater), typed in by the rep | Ask |
| Details | Buyer name and address | Contact block | `buyerContacts` / `buyerCompany` | Yes |
| Pricing Summary | Total | Amount | `quote.hs_total` or calculated | Ask |

Source types to choose from: **Line items** (automatic), **Quote / deal / contact / company property** (automatic), **`crm_object()` lookup** (custom or sparse data), **Editor field** (rep types it manually, including repeaters), **Static** (fixed text in the module). For each table, say clearly whether its rows come from the quote line items or are entered manually, and which columns map to which properties. Mark anything you are not certain about with "Ask".

### Phase 2: Ask every question needed for a proper template (ONE message, then stop)

Quality of the template matters more than brevity here. Ask about every point that could change how a module is built or where its data comes from, so nothing is guessed:
- Every row marked "Ask" in the Analysis Report (table data from line items or manual, totals calculated or from a property, missing or unreadable content)
- Module split (confirm the proposed module list) and module order
- Which parts must be editable by the sales rep in the editor, and which are fixed
- Interactivity (optional line items, toggles, tabs, sign area) and what it should do
- Responsive scope, fonts and assets that are not in the file, and branding that is unclear
- Anything contradictory or missing in the file
- **Project name** (ask when the starter was just cloned, see "Rename the project" in Phase 0): what should the project be called? Default: `<client or brand>-<document type>`, lowercase with hyphens, for example `goodwood-proforma-invoice`
- **Module name** (always ask, see "Module Naming Rule"): what should the module be called in the quote editor? Default: `<App name> - <Module name>`
- **Page orientation** (always ask, see "Page Orientation Rule"): portrait or landscape for print and PDF? Default: portrait, unless the design has a wide table (more than 8 columns), then landscape
- **Line item property names** (when a table uses custom columns): the HubSpot internal names of any custom line item properties (for example CBM, material, image URL). Default: a configurable text field per property
- **Currency and number format** (when the file uses a non-US format): keep the file's format or follow the portal locale? Default: keep the file's format

Rules for asking:
- Send ONE message only, with a numbered list grouped by module, followed by the HubSpot prompts from Phase 2b (Part A: questions, Part B: prompts). Do not ask in several rounds, and do not drip-feed questions.
- Every question is closed-ended with a recommended default, for example: `3. Pricing table rows: from quote line items (default) or typed manually?`.
- Do not offer multiple approaches or alternatives to choose between. Pick the best one and state it as the default.
- Do not ask about things you can decide yourself (file names, internal structure, placeholder text, code style).
- If the file is fully clear and every row in the report is "Yes", skip the questions and say so.
- End with: "Reply 'ok' to accept all defaults, or answer by number." Then wait for the answer.
- Ask a second round only if an answer creates a new blocker. Validating the pasted HubSpot answers (Phase 2b, step 4) is not a new round.

### Phase 2b: HubSpot Data Request (ask the portal, do not guess)

Claude cannot see the user's HubSpot portal unless the HubSpot MCP is connected with the right permissions. A template built on guessed property names looks fine in preview and then shows blank cells on a real quote. So after the analysis, work out which facts only the portal knows, and give the user ready-to-paste prompts for HubSpot's own assistant (Breeze Assistant, the AI chat inside HubSpot). The user pastes the prompts in HubSpot, then pastes the answers back to you.

**Step 1: Decide what is needed.** For every row in the Analysis Report with a data source (not "Static"), list the HubSpot facts the module depends on. Typical ones:
- Internal names and types of line item properties used by the table (standard and custom: SKU, CBM, material, image, and so on)
- Real SKU examples, to confirm any grouping or parsing rule
- Quote properties (number, title, currency, dates, sender details) and any custom quote properties for totals, deposit, or notes
- Buyer contact, company, billing, and address properties
- Currency, number and date format, and tax settings
- Whether custom React quote modules are available in the account and for which template types (quote, invoice)
- Product image property and the form of its URL

**Step 2: Show a "HubSpot Details Needed" table**, one compact table: what you need, why (which module or column uses it), the prompt number to use, and what you will do if the answer is "not available".

**Step 3: Provide the prompts.** Use the prompt library below. Adapt the placeholders (`<...>`) to the design, drop prompts that do not apply, and give them in ONE message together with the Phase 2 questions, as "Part A: questions" and "Part B: prompts for the HubSpot assistant". Put each prompt in its own code block so it can be copied with one click. Every prompt must:
- Be self-contained (the assistant has no memory of this chat)
- Ask for a fixed output format (a table or JSON) so the answer can be pasted back and read without editing
- Ask the assistant to say "not found" instead of guessing
- Ask for real values only, never invented examples

**Step 4: Wait, then validate.** When the user pastes the answers:
1. Compare every answer with the mapping in the plan (property names, types, example values, SKU patterns).
2. Report in one short table: confirmed, mismatch (what to change), and still unknown.
3. Update the module fields and mapping from the real names, then continue to Phase 3. This validation does not count as a second round of questions.
4. If an answer is missing or says "not found", say what the module will do instead (a configurable field, a placeholder, or the column left out) and let the user decide.

**If the HubSpot MCP is connected and has the needed permissions** (for example `crm.schemas.line_items.read`), read the facts directly with the MCP tools and show the findings instead of prompts. If a permission is missing, tell the user which one, and fall back to the prompts.

#### HubSpot assistant prompt library

Use these as templates. Replace `<...>` placeholders. Each one ends with the required return format.

**P1. Line item properties (structure check)**
```
List every line item property in my HubSpot account. For each one return: internal name, label, type, field type, group, whether it is a custom property (true/false), and the option values if it is a dropdown. Include especially any property related to: <e.g. CBM, volume, material, upholstery, finishing, dimensions, image, product family>. Return a markdown table. Do not guess; if a property does not exist, list it under "Not found".
```

**P2. Real line item data from a real quote**
```
Open the most recent quote that has line items (or quote <quote name or number>) and return ALL line items with the values of these properties: hs_sku, name, description, price, quantity, amount, discount, hs_discount_percentage, hs_images, <custom property internal names from P1>. Return JSON: an array with one object per line item, using the internal property names as keys. Use the real values only. If a property is empty, return null. Do not invent data.
```

**P3. SKU pattern check (for grouping or parsing rules)**
```
List the SKU (hs_sku) of every product in my product library, with product name. Then check this rule: "SKU is split by '-' and the middle part is the group code (for example FR-LC528A-306 gives LC528A)". Return: (1) a table of SKU, middle part, and product name; (2) which middle parts are shared by more than one SKU; (3) the SKUs that do not have exactly 3 parts. Do not change any data.
```

**P4. Product to line item property mapping**
```
For these custom properties: <list>, tell me: do they exist on BOTH products and line items with the same internal name? When I add a product to a quote, does each value get copied from the product to the line item automatically? Show one real example (product value and the line item value). If you are not sure, say so. Do not guess.
```

**P5. Quote properties and custom quote properties**
```
List the quote properties I can use in a quote template: internal name, label, type for these: hs_quote_number, hs_title, hs_currency, hs_expiration_date, hs_last_published_date, hs_tax_total, hs_quote_amount, hs_net_payment_terms, and all sender properties (hs_sender_company_name, hs_sender_company_address, hs_sender_company_city, hs_sender_company_zip, hs_sender_company_country, hs_sender_image_url). Then list any CUSTOM quote properties (for example for deposit %, additional cost, other charges, bank details, notes). Return a markdown table, and "Not found" for missing ones.
```

**P6. Buyer, company and billing data**
```
For the most recent quote, return the buyer contact (firstname, lastname, email, phone), buyer company (name, address, city, state, zip, country, domain) and billing company if different. Return JSON using internal property names. Say "empty" for blank values.
```

**P7. Currency, number format and tax**
```
What are my account's default currency, additional currencies enabled, number format (decimal and thousand separators) and date format? Are taxes used on quotes (tax type, rate, shown per line or per quote)? Are discounts used on line items, and are they percent or amount? Return a short list.
```

**P8. Template capability check**
```
Does my account (name the subscription tier) support custom-coded React quote modules built with the HubSpot quote development SDK? Which template types can use them: quotes, quote templates, invoices, other? What value should the module's content_types contain for each? Cite the HubSpot documentation page. If not documented, say "not documented".
```

**P9. Images and files**
```
Which product or line item property holds the product image, and what does its value look like (a full URL, a file ID, or a list)? Show one real example value. Also give the file manager URL of our company logo, <logo file name>. Return as a short list.
```

**P10. Overall structure check (run this last)**
```
Describe how quotes are structured in my account: how quotes, line items, products, deals and companies are associated; whether quotes use multiple currencies; whether recurring, optional or one-time line items are used; and how many line items a typical quote has (min, max, average over the last 20 quotes). Return a short bullet list. Do not guess.
```

**Choosing prompts:** line item table in the design means P1, P2, P3 (if grouping or parsing), P4 (if custom columns), P9 (if pictures). Totals or payment block means P5 and P7. Contact or address block means P6. Any new project means P8 and P10.

### Phase 3: Short plan, then build

Write a compact plan and show it to the user (keep it tight, but include the confirmed data source of every table and dynamic element from the Analysis Report):
1. Module list (see "Module Splitting Rule" below): each module name (final editor label, see "Module Naming Rule"), what it contains, new or extension, and the order they appear in the quote
2. Page orientation (portrait or landscape) and which module owns the page rule, plus the default padding and margin of each module (see "Spacing Settings Rule")
3. Sections to build, in order
4. Data binding: each dynamic value mapped to its `quoteTemplateContext` field (or `crm_object()` lookup)
5. Islands needed (or "none")
6. Assumptions and answers from Phase 2
7. Files to create or change, and any leftover starter example files to remove

If the user already answered questions or said to proceed, start building right after showing the plan. Otherwise wait for approval once. Then continue with the Development Workflow below.

## Module Splitting Rule (DO NOT put everything in one module)

A quote template is a page built from several modules that the sales rep can add, remove, and reorder in the quote editor. Never build the whole design as one big module. Split it in the plan, before writing code.

Create a separate module for each section that is independent, reusable, or that a rep might want to move or remove. Typical split:

| Module | Contains |
|---|---|
| Header / Cover | Logo, quote title, dates, sender, prepared-for |
| Content | Intro, scope, about, free text or rich text blocks |
| Line Items | Product table, quantities, prices, discounts, totals |
| Pricing Summary | Subtotal, tax, total, payment terms, billing schedule |
| Details | Buyer contact and company, deal info, sender info |
| Terms / Notes | Terms and conditions, disclaimers |
| Signature / Form | Signers, counter-signers, accept or sign area, any form |
| Footer | Contact info, legal line, page footer |

Rules:
- Split by section, not by component. A table row or a button is a component inside a module, not a module.
- Keep one module only when the whole design is a single small, inseparable block. State the reason in the plan.
- Each module has its own directory under `components/modules/<ModuleName>/` with its own `Component`, `fields`, `meta`, and `hublDataTemplate`, and its own small `HublData` type that selects only the data that module uses.
- Put code shared by several modules in `components/shared/` (for example `theme.ts` for colors, fonts, and spacing, `format.ts` for currency and date helpers, and common small components). Do not copy helpers between modules.
- Extending an existing module? Add to it only if the new content belongs to the same section. Otherwise create a new module.
- Every module works on its own: it needs its own `isQuoteBlueprint` placeholder data and `is_in_editor` fallback.

## Module Naming Rule (make every module easy to find)

Reps pick modules from a list in the quote editor, so the name decides whether they can find it.

1. Always ask the user for the module name in Phase 2 (one question per module, or one name pattern for all).
2. If the user gives a name, use it exactly as the `meta.label`.
3. If the user does not answer or says "default", build the label as `<App name> - <Module name>`:
   - **App name** = the HubSpot project name from `hsproject.json` (`name`), written in Title Case with dashes turned into spaces (`quote-dev-starter` becomes `Quote Dev Starter`). If the user named the product or client in the request, use that name instead.
   - **Module name** = what the section is (`Header`, `Line Items`, `Pricing Summary`, `Terms`, `Signature`, `Footer`).
   - Example: `Goodwood Proforma - Line Items`, `Goodwood Proforma - Banking And Totals`.
4. Use the same app prefix for every module of one template, so all of them group together in the list.
5. Keep labels under 50 characters, plain text only, no emoji, and never reuse a label that another module in the project already has.
6. The directory name stays PascalCase and matches the label (`GoodwoodProformaLineItemsModule`). Do not rename a deployed module's directory later, because that creates a second module instead of updating the first.
7. Show the final labels in the plan so the user can correct them before code is written.

## Page Orientation Rule (portrait or landscape)

HubSpot has no setting or field for page orientation. Quotes are web pages, and HubSpot renders the PDF and print view from the page with `?print=true`. So orientation can only be controlled with print CSS, and it is not an official, documented feature. Community posts show the `@page` rule below working for quote templates, but it is not guaranteed, so always tell the user to verify it on a downloaded PDF.

1. Always ask the user in Phase 2: "Portrait or landscape for the printed/PDF quote?" Default: portrait; landscape when the line item table has more than 8 columns.
2. Implement it with one print rule:

```css
@media print {
  @page { size: A4 portrait; margin: 12mm; }   /* or: size: A4 landscape */
}
```

3. `@page` applies to the whole document, not to one module. Put it in exactly ONE module (the Header or Cover module, or a shared style constant used by one module only). If two modules declare different sizes, the result is unpredictable.
4. Also size the content for the chosen orientation: portrait is about 186mm usable width on A4 with 12mm margins, landscape about 273mm. Tables must fit that width in print (`min-width: 0`, smaller font, wrapped text); the on-screen horizontal scroll does not work on paper.
5. Add `break-inside: avoid` to rows or row groups, and `break-after: page` after a cover page, so nothing is cut across pages.
6. Add `print-color-adjust: exact` (and `-webkit-print-color-adjust: exact`) to every shaded header, band, or badge, or the PDF will print them white.
7. Tell the user plainly in the final message: "Orientation is set with print CSS, not a HubSpot setting. Download the PDF of a real quote to confirm it."

## Design and Full-Size Rules (fit the module to the rest of the quote)

Each module is one slice of a page made of other modules. It must fill the space it is given and sit flush with its neighbours.

1. **Full width.** The module root is `width: 100%` with `boxSizing: 'border-box'` and `margin: 0`. Do not set a fixed pixel width or a centred `max-width` on the root unless the design is deliberately narrow. Let the quote page decide the outer width.
2. **No page chrome inside a module.** Do not add a page background, outer shadow, outer border, or `min-height: 100vh` to a module root. Those belong to the quote page. A coloured full-width band is fine when the design has one.
3. **Spacing comes from the source page.** The outer padding and margin of each module follow the source HTML (see "Spacing Settings Rule"). Internal gaps use a shared scale in `components/shared/theme.ts`, built from the source's own values (for example the source uses 6, 10, 36, 60 px, so the scale uses those). Do not invent a new scale.
4. **One type system.** Fonts, sizes, and colours come from `theme.ts`. Do not hard-code a new font in a single module. Inherit the font from the quote page unless the design requires a specific one, and then load it once.
5. **Tables fill the width.** `width: 100%`, `table-layout: auto`, and wrap in a container with `overflowX: 'auto'` for screens. Give the picture and number columns fixed widths and let the description column take the rest. Right-align money, centre quantities.
6. **Grids and flex rows fill the row.** Use `flex: 1` or `grid-template-columns: repeat(...)` so columns share the full width. Avoid fixed-width blocks that leave empty space on the right.
7. **Match the design, not an approximation.** Before finishing, compare the preview with the source file section by section: every block, label, column, heading, note, footer, logo, and legal line in the source must exist in the module. Missing details are a defect. Add a checklist of the source's sections to the plan and tick them off.
8. **Images.** Logos and pictures are image fields or URL properties, never base64 embedded in code. Constrain with `max-width: 100%` and `height: auto`, and show a neutral placeholder when the image is empty.
9. **Responsive and print are part of the design.** The same module must work at mobile width, desktop width, and on paper (see "Page Orientation Rule").
10. **Check the neighbours.** When adding a module to an existing template, open the other modules' styles first and reuse their spacing, colours, and heading style so the new module looks like part of the same document.

## Spacing Settings Rule (padding and margin on all four sides, in EVERY module)

Every module must let the rep set its padding and margin on all four sides (top, right, bottom, left) from the editor, so the module can be fitted to the modules around it without a code change. This is mandatory for every module in a template, including small ones such as a footer or a signature block.

1. **Use HubSpot's `SpacingField`** (type `spacing`, imported from `@hubspot/cms-components/fields`). It gives separate padding and margin controls for top, right, bottom, and left, each with a value and a unit.
2. **Put it in the STYLE tab**, inside the module's `styles` group, named `spacing`, with the label "Spacing (padding and margin)":

```tsx
import { ModuleFields, FieldGroup, ColorField, SpacingField } from '@hubspot/cms-components/fields';

<FieldGroup name="styles" label="Styles" tab="STYLE">
  <ColorField name="accentColor" label="Accent color" default={{ css: '#1f4d2b' }} />
  <SpacingField
    name="spacing"
    label="Spacing (padding and margin)"
    // Example only: these numbers come from the source CSS (.page { padding: 56px 64px }).
    // Always replace them with the values measured from YOUR source page.
    default={{
      padding: {
        top: { value: 56, units: 'px' }, right: { value: 64, units: 'px' },
        bottom: { value: 56, units: 'px' }, left: { value: 64, units: 'px' },
      },
      margin: {
        top: { value: 0, units: 'px' }, right: { value: 0, units: 'px' },
        bottom: { value: 0, units: 'px' }, left: { value: 0, units: 'px' },
      },
    }}
  />
</FieldGroup>
```

3. **Take the default padding and margin from the source HTML page, not from generic values.** Before writing the field, read the source file's CSS and measure the spacing of the section this module covers, then use those exact numbers and units as the `default` of the `SpacingField`:
   - Padding: the `padding` of the section's container (for example `.page { padding: 56px 64px }` gives top 56, right 64, bottom 56, left 64).
   - Margin: the space the source puts around the section (its own `margin`, the `margin-top` or `margin-bottom` it gets from neighbours, or the `gap` between it and the next block, for example `.summary { margin-top: 36px }` gives margin top 36).
   - Expand shorthand correctly: `padding: 10px 20px` is top 10, right 20, bottom 10, left 20; `padding: 10px 20px 30px` is top 10, right 20, bottom 30, left 20; a single value applies to all four sides.
   - Keep the source's unit (`px`, `rem`, `em`, `%`). Convert `mm` or `pt` only if the field limits do not allow them, and say so in the plan.
   - If the source is a screenshot or PDF with no CSS, measure from the image (pixels at 100% zoom) and state that the values are estimated.
   - If the section has no spacing of its own in the source, use 0 for that side. Never invent a default such as 24 or 32 to make it look "nicer".
   - The module must look the same as the source with the defaults untouched. Record the source value next to each default in the plan (for example `padding 56 64 56 64, from .page`).
4. **Optional limits.** Add `limits` (min, max, allowed units) when a value could break the layout, for example `max: 120` for padding. Allow `px`, `rem`, and `%` only when the design can handle them.
5. **Apply it to the module root element only**, as inline `style`: `padding` and `margin` built from the four values. Inner elements keep their own fixed internal spacing from `theme.ts`. Do not apply the setting to several inner elements.
6. **Use one shared helper** in `components/shared/spacing.ts` so every module converts the field the same way and falls back to the defaults when the value is empty:

```ts
type Side = { value?: number; units?: string };
type Box = { top?: Side; right?: Side; bottom?: Side; left?: Side };
export type SpacingValue = { padding?: Box; margin?: Box };

const side = (s: Side | undefined, fallback: string) =>
  s && typeof s.value === 'number' ? `${s.value}${s.units || 'px'}` : fallback;

export function boxToCss(box: Box | undefined, fallback = '0') {
  return [box?.top, box?.right, box?.bottom, box?.left].map((s) => side(s, fallback)).join(' ');
}

export function spacingStyle(spacing: SpacingValue | undefined, defaults?: { padding?: string; margin?: string }) {
  return {
    padding: spacing?.padding ? boxToCss(spacing.padding) : defaults?.padding,
    margin: spacing?.margin ? boxToCss(spacing.margin) : defaults?.margin,
  };
}
```

   Use it as `style={{ width: '100%', boxSizing: 'border-box', ...spacingStyle(fieldValues.styles?.spacing, { padding: '56px 64px', margin: '0' }) }}`, where the fallback strings repeat the same source-page values as the field defaults.
7. **Keep `boxSizing: 'border-box'` and `width: 100%`** on the root, so padding never makes the module wider than its container (see "Design and Full-Size Rules").
8. **Print.** The spacing setting applies to print as well. For a page with a fixed paper margin (`@page`), keep module padding small in print (`@media print`) so content is not pushed off the page.
9. **Show it in the demo template.** In `preview/index.html`, expose the same four-side padding and margin values for each module (for example in the module's label bar), so the spacing can be checked before deploy.
10. **List it in the plan.** For every module, state the default padding and margin values and the source CSS rule each came from. If a module has no spacing setting, or its defaults were not taken from the source page, it is incomplete.

## Data Source Rules (line items versus typed fields)

1. Default for any product or price table: read from `quoteTemplateContext.lineItems`. Typed (editor field) rows are only for content a rep writes by hand, such as a scope list. Confirm with the user in Phase 2 and state the choice in the plan.
2. Standard line item properties can be read directly: `hs_sku`, `name`, `description`, `price`, `quantity`, `amount`, `discount`, `hs_discount_percentage`.
3. Custom line item properties (for example CBM, material, image URL) have unknown internal names. Do not hard-code guesses. Add a text field per property in the module (with the best-guess default) so the rep can correct the name without a code change. If the HubSpot MCP can read line item properties (`crm.schemas.line_items.read`), look the real names up instead.
4. Totals shown on the page must be calculated from the same line items (use `amount` for net after discount), so they match the quote total. Do not mix typed numbers with line item totals.
5. Grouping or merging rows is done in the component on the mapped line item array (group key, sort inside group, `rowSpan` for merged cells). Rows without a matching key stay as single rows. Keep the group key rule configurable in a module field.
6. In the template editor (`isQuoteBlueprint` is true) show dummy data so the module is never blank. On a real quote use the real line items.
7. HubSpot documents `QUOTE` and `QUOTE_BLUEPRINT` as the only module content types. If the user wants the same module on invoices, do not guess a value. Tell the user it is not documented and ask them to confirm with HubSpot.

## Field and Build Rules (avoid failed uploads)

1. A module field must never be named `name`. HubSpot rejects it with "field name cannot be 'name'". Use `itemName`, `title`, or `label`. Check reserved words before choosing field names.
2. Values of fields inside a `FieldGroup` arrive nested under the group name (`fieldValues.styles.accentColor`). Either read them that way, or keep content fields flat and use groups only for the STYLE tab.
3. Field internal names: lowercase or camelCase, no spaces, unique inside the module.
4. Only ASCII letters, digits, `-`, `_`, and `.` in file and folder names. A single bad character fails the whole upload ("contains invalid character").
5. Before every `hs project upload`, list the project for stray files (names containing `)`, `(`, `>`, `|`, or spaces). These are usually created by a badly quoted shell command, for example an inline `node -e "..."` containing `=>`. Run test snippets from a script file or a heredoc with a quoted delimiter, never with unescaped `>` inside a double-quoted shell string.
6. Type-check first (`npx tsc --noEmit -p .` inside `src/cms-assets/my-react-assets`) and fix errors before uploading. Then read the upload output to the end: a build can upload and still fail in "Building".
7. Deploying changes the user's HubSpot account. Deploy only when the user has asked for it, name the target account in your message, and report the build number.
8. A successful build is not proof that the module looks right. Say clearly what was verified (type-check, build) and what was not (rendering in the real quote, PDF output), and ask the user to preview a real quote.

## Demo Template (full-page preview)

Always build a demo template in code so the whole quote can be previewed as one page, the way the modules will look stacked in the real quote template:

- Create `preview/index.html` that renders ALL modules from the plan, in the order of the design, stacked in one page.
- Use ONE shared `MOCK_DATA` object for every module, shaped like `quoteTemplateContext` (quote, deal, lineItems, buyerContacts, buyerCompany, signers), so the modules show consistent data.
- Use the same component structure, styles, and field default values as the real module files, so what you see in preview matches the deployed quote. When a module changes, update its part of the preview in the same step.
- Add a small toolbar at the top of the preview (not part of the design) with: a desktop / tablet / mobile width switch, and a toggle for `isInEditor` and `isQuoteBlueprint` to see the editor and placeholder states.
- Each module in the page sits in its own wrapper with a thin label (module name) that is easy to hide, so modules can be checked one by one.
- Keep the page background, max width, and spacing the same as the HubSpot quote page so the look is close to the real thing.
- The demo template is preview only. It is never deployed.

## Project Structure

```
├── hsproject.json                          # HubSpot project config
├── package.json                            # Root package (deploy scripts)
├── preview/
│   └── index.html                          # Local preview with mock data (not deployed)
├── .claude/
│   ├── launch.json                         # Dev server config for Claude Preview
│   └── skills/
│       └── quote-custom-modules-dev.md      # This file
└── src/cms-assets/my-react-assets/
    ├── package.json                        # React + @hubspot/cms-components deps
    ├── cms-assets.json                     # Asset bundle config
    ├── tsconfig.json                       # TypeScript config
    └── components/modules/
        └── <ModuleName>/
            ├── index.tsx                   # Module entry: Component, fields, meta, hublDataTemplate
            └── islands/
                └── <IslandName>.tsx        # Interactive client-side components
```

## Key Concepts

### Module Anatomy

Every module exports four things from `index.tsx`:

1. **`Component`** — React component receiving `{ fieldValues, hublData }`
2. **`fields`** — JSX defining editor sidebar fields (`TextField`, `ColorField`, `FieldGroup`, etc.)
3. **`meta`** — Module metadata. `content_types` MUST include `['QUOTE', 'QUOTE_BLUEPRINT']`
4. **`hublDataTemplate`** — HubL template string that passes server-side data to the component

Standard imports:

```tsx
import { ModuleFields, TextField, FieldGroup, ColorField } from '@hubspot/cms-components/fields';
import { Island } from '@hubspot/cms-components';
import type { QuoteTemplateContext } from '@hubspot/quote-dev-sdk';
```

### Typing with `@hubspot/quote-dev-sdk`

The SDK exports `QuoteTemplateContext` and its member types (`QuoteProperties`, `DealProperties`, `LineItemProperties`, `ContactProperties`, `CompanyProperties`, `SignerProperties`, `QuoteDocumentProperties`). Use them to type the `hublData` prop so autocomplete works and property-name typos become compile errors.

**Only pass the data the module actually uses.** Each module's `hublDataTemplate` should select specific fields from `quoteTemplateContext`, and its `HublData` interface should type exactly that shape. This keeps the HubL-to-React bridge lean, makes the component's data dependencies explicit, and gives TypeScript something meaningful to check.

```tsx
interface HublData {
  quote: Pick<QuoteTemplateContext['quote'], 'hs_title' | 'hs_currency'>;
  lineItems: QuoteTemplateContext['lineItems'];
}

export const hublDataTemplate = `
  {% set hublData = {
    "quote": {
      "hs_title": quoteTemplateContext.quote.hs_title,
      "hs_currency": quoteTemplateContext.quote.hs_currency
    },
    "lineItems": quoteTemplateContext.lineItems
  } %}
`;
```

Useful narrowing patterns:

```ts
import type { QuoteTemplateContext, LineItemProperties } from '@hubspot/quote-dev-sdk';

type LineItemsOnly = Pick<QuoteTemplateContext, 'lineItems'>;
type WithoutDocuments = Omit<QuoteTemplateContext, 'quoteDocuments'>;
type BillingFields = Pick<LineItemProperties, 'name' | 'amount' | 'hs_recurring_billing_period' | 'hs_term_in_months'>;
```

### quoteTemplateContext

The `quoteTemplateContext` HubL global provides all quote data. Its top-level shape:

```
quoteTemplateContext: {
  quote: QuoteProperties          — title, status, currency, amounts, sender info, dates, settings
  deal: DealProperties            — name, stage, amount, pipeline, close date
  lineItems: LineItemProperties[] — name, quantity, price, amount, SKU, billing terms, discounts
  buyerContacts: ContactProperties[]  — name, email, company, address
  buyerCompany: CompanyProperties     — name, domain, address
  billingContact: ContactProperties
  billingCompany: CompanyProperties
  quoteDocuments: QuoteDocumentProperties[]
  signers: SignerProperties[]     — firstName, lastName, email, avatarUrl
  counterSigners: SignerProperties[]
}
```

The full property list for each type is defined in `@hubspot/quote-dev-sdk`. Install it as a dev dependency and refer to the type definitions for the complete set of available fields. All CRM-object interfaces include an index signature (`[key: string]: CrmPropertyValue | undefined`) — properties beyond the typed surface may exist at runtime.

> **Note:** `buyerCompany` from `quoteTemplateContext` may be a sparse object. To get the full company record (including `hs_logo_url`), look it up in `hublDataTemplate` via `crm_object("company", quoteTemplateContext.buyerCompany.hs_object_id, "name,domain,hs_logo_url")`.

To enrich a record beyond what `quoteTemplateContext` exposes — including custom properties defined in the portal — use `crm_object()` or `crm_associations()` inside the HubL block:

```tsx
export const hublDataTemplate = `
  {% if quoteTemplateContext.buyerCompany.hs_object_id %}
    {% set companyDetails = crm_object("company", quoteTemplateContext.buyerCompany.hs_object_id, "name,domain,hs_logo_url") %}
  {% endif %}

  {% set hublData = {
    "quoteTitle": quoteTemplateContext.quote.hs_title,
    "companyName": companyDetails.name if companyDetails else null,
    "companyDomain": companyDetails.domain if companyDetails else null
  } %}
`;
```

### Islands Architecture

Modules are server-side rendered (SSR). For client-side interactivity, use Islands:

```tsx
// @ts-expect-error -- ?island not typed
import InteractiveComponent from './islands/MyComponent?island';
import MyComponent from './islands/MyComponent';

// In editor: render static version
// On published quote: render island with hydration
{isInEditor ? (
  <MyComponent {...props} />
) : (
  <Island module={InteractiveComponent} hydrateOn="load" {...props} />
)}
```

**Props passed to an `<Island>` are serialized onto the DOM.** Only pass scalar values and small objects that are directly relevant to the island's rendered output. Avoid passing entire `quoteTemplateContext` slices, large arrays, or objects with fields the island doesn't use — the data ends up in the HTML payload and inflates page weight for no benefit.

### Editor Detection and Template Fallbacks

Use HubL variables to detect editor/previewer context:

```
"isInEditor": is_in_editor,
"isQuoteBlueprint": isQuoteBlueprint
```

`is_in_editor` is important because Islands cause full-page reloads in the editor. Render static fallbacks in editor mode.

`isQuoteBlueprint` is `true` when the module renders inside a quote *template* rather than an individual quote. In that context, `quoteTemplateContext` contains no real data — buyer contacts, company, and line items will be empty or null. Modules should detect this and substitute sensible placeholder values so the template editor still shows a representative preview:

```tsx
const companyName = hublData.isQuoteBlueprint ? 'Acme Corp' : hublData.companyName;
const lineItems = hublData.isQuoteBlueprint ? PLACEHOLDER_LINE_ITEMS : hublData.lineItems;
```

## Development Workflow

### Building a New Quote from a Screenshot (primary workflow)

Complete the Intake section above first (analyze, at most one clarifying message, short plan). The user provides a screenshot of the target design. Build **both** `preview/index.html` and `<ModuleName>/index.tsx` in a single pass — do NOT build the preview first and port later.

#### Step-by-step:

1. **Confirm the plan** — reuse the analysis and plan from Intake; do not re-analyze
2. **Create mock data** matching the screenshot content (company names, line items, amounts, contacts)
3. **Write `preview/index.html`** — the demo template: standalone HTML with React CDN + Babel standalone + one shared mock data object, rendering every module from the plan stacked in order (see "Demo Template")
4. **Write one `components/modules/<ModuleName>/index.tsx` per module in the plan** — the real modules using the same component structure, typed with `@hubspot/quote-dev-sdk`, with shared code in `components/shared/`
5. **Start preview server** — `python3 -m http.server 3456 -d preview` (or use `.claude/launch.json`)
6. **Iterate** — user reviews preview, requests tweaks, update both files

#### Preview HTML template:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet" />
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    const MOCK_DATA = { /* ... */ };
    // Components here — same structure as the TSX module
    function App() { /* ... */ }
    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
  </script>
</body>
</html>
```

#### Styling

All styles should be inline React `style` objects — do not use CSS files. The one exception is a small `<style>` tag inside the module (`dangerouslySetInnerHTML`) for things inline styles cannot express: `@media print`, `@page`, `@media (max-width)`, and pseudo-selectors. Keep it in a `styles.ts` constant. Load fonts via `@import` or `<link>` in the preview HTML. Use the `useContainerWidth()` hook for responsive layouts.

### Preview with Claude Preview

The `.claude/launch.json` should be configured to serve the preview directory:

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "preview",
      "runtimeExecutable": "python3",
      "runtimeArgs": ["-m", "http.server", "3456", "-d", "preview"],
      "port": 3456
    }
  ]
}
```

If Claude Preview tools are available, use them to iterate visually:
- `mcp__Claude_Preview__preview_start` — Start the preview server
- `mcp__Claude_Preview__preview_screenshot` — Capture the current state
- `mcp__Claude_Preview__preview_resize` — Test responsive layouts (desktop, tablet, mobile)
- `mcp__Claude_Preview__preview_eval` — Debug DOM state, scroll, reload
- `mcp__Claude_Preview__preview_snapshot` — Get accessibility tree for text verification

If Claude Preview tools are NOT available, start the server manually:
```bash
python3 -m http.server 3456 -d preview &
```
Then tell the user to open http://localhost:3456 in their browser.

The `preview/index.html` is the demo template for the whole quote (all modules stacked). It is never deployed. Update it whenever a module changes.

### Local Dev Server

The starter project includes `hs-cms-dev-server` for local development against real HubSpot rendering:

```bash
npm start
```

This boots the CMS dev server from the React assets directory and serves a local preview of the module with HubL evaluation. Use this to verify `hublDataTemplate` behavior before deploying.

### Make It Responsive

Use a `ResizeObserver`-based hook to measure container width (not viewport width — the module may be embedded in a narrower container):

```tsx
function useContainerWidth() {
  const ref = useRef<HTMLDivElement>(null);
  const [width, setWidth] = useState(1200);
  useEffect(() => {
    if (!ref.current) return;
    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) setWidth(entry.contentRect.width);
    });
    observer.observe(ref.current);
    setWidth(ref.current.offsetWidth);
    return () => observer.disconnect();
  }, []);
  return { ref, width };
}
```

Pass `compact` (e.g., `width < 700`) to all sub-components and adjust:
- 2-col grids → 1-col
- 4-col grids → 2-col
- Tables → horizontal scroll with `overflowX: 'auto'`
- Smaller font sizes and padding

Test with `preview_resize` using `preset: "mobile"` and `preset: "desktop"`.

### Deploy

```bash
npm run deploy
```

This runs `hs project upload`, which builds and auto-deploys to the HubSpot portal. The module becomes available in the quote template editor.

## Requirements & Constraints

- Requires a Revenue Hub Professional or Enterprise account
- Project name cannot be `cpq-theme`
- Module `content_types` must include `QUOTE` and `QUOTE_BLUEPRINT`
- Modules must be React (classic HubL modules are not supported)
- Island interactivity should not render in the editor — use `is_in_editor` to conditionally render static fallbacks
- New module versions apply to future quotes and unpublished drafts, but not to already-published quotes
- Deleting or removing a module from the project removes it from unpublished quotes and templates; published quotes are unaffected

## Starter Module

The starter project ships with `QuoteExampleModule` as a minimal reference for module structure (`Component`, `fields`, `meta`, `hublDataTemplate`) and SDK typing. Use it as a scaffold when creating a new module — copy the directory, rename, and replace the contents.

Each section of the design gets its own module directory under `components/modules/` (see "Module Splitting Rule"), and shared code goes in `components/shared/`.

## Common Mistakes

- **Don't build the whole quote as one module.** Split by section (header, content, line items, details, terms, signature, footer) and keep shared code in `components/shared/`.
- **Don't skip the demo template.** Always keep `preview/index.html` rendering all modules stacked with one shared mock data object.

- **Don't use CSS files.** All styles should be inline React `style` objects.
- **Don't render Island interactivity in the editor.** Check `is_in_editor` and render a static fallback instead — Islands cause full-page reloads in editor context.
- **Don't pass whole objects or arrays to Islands.** Island props are serialized onto the DOM. Pass only the scalar values and small objects the island actually renders.
- **Don't forward the entire `quoteTemplateContext`.** Each module's `hublDataTemplate` should select only the fields the component uses. Type the `HublData` interface to match.
- **Don't forget `isQuoteBlueprint` fallbacks.** When rendering in a quote template, `quoteTemplateContext` has no real data. Substitute placeholder values so the template editor shows a representative preview.
- **Don't leave the clone named `quote-dev-starter`.** Rename the project to match the user's requirement right after cloning (before the first upload), and never rename one that is already deployed.
- **Don't guess HubSpot property names.** Give the user the Phase 2b prompts for the HubSpot assistant, wait for the pasted answers, and check them against the plan before building. If the MCP can read the data, read it instead.
- **Don't skip the module name and orientation questions.** Always ask both in Phase 2; if there is no answer, use `<App name> - <Module name>` and portrait (landscape for wide tables).
- **Don't leave out details from the source design.** Check every section, column, heading, note, and footer of the source against the module before calling it done.
- **Don't size a module for itself.** Full width, no page chrome, shared spacing and fonts, so it fits the modules around it.
- **Don't name a field `name`**, and don't leave stray files with odd characters in the project before uploading.
- **Don't declare `@page` in more than one module.**
- **Don't ship a module without a spacing setting.** Every module needs the `SpacingField` (padding and margin on all four sides) in its STYLE tab, applied to the root with the shared `spacingStyle` helper.
- **Don't claim it works in HubSpot after only a build.** Say what was and was not verified.
- **Don't skip the `@ts-expect-error` on `?island` imports.** TypeScript doesn't understand the `?island` query suffix without an explicit module declaration.
