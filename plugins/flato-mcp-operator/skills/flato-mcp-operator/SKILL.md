---
name: flato-mcp-operator
description: Use when creating, editing, inspecting, exporting, or troubleshooting Flato designs through Flato MCP tools such as flato_whoami, flato_use_project, flato_get_design_context, flato_create_pages, flato_update_pages, flato_update_canvas, flato_export_to_png, and related flato_* tools.
---

# Flato MCP Operator

Use this skill whenever the user wants Codex to operate a Flato design through MCP.

Flato is a live visual design editor. The MCP server gives Codex editor tools, design resources, prompt-skill resources, brand-kit resources, image asset tools, and feedback. It is not a hidden one-shot design agent. Codex must read current Flato context explicitly and then call concrete MCP tools.

## Non-Negotiable Resource Policy

Codex must not rely only on tool descriptions for Flato work. The MCP resources and prompts are part of the operating contract.

For every Flato MCP task that creates, edits, styles, brands, exports, or troubleshoots a design:

1. Actively enumerate or read Flato MCP resources before writing.
2. Actively select and read the scenario design skill resource.
3. Actively check user brand-kit resources when brand constraints are possible.
4. Use MCP workflow prompts when the host exposes prompt invocation. If prompt invocation is unavailable, follow the equivalent workflow in this skill and the prompt-skill resource.

Do not wait for the user to ask for these resources. Do not skip them because `flato_get_design_context` is available. Design context tells Codex what is on the canvas; resources tell Codex how to use Flato and how to design for the scenario.

## Resource Inventory

Expected Flato MCP resources:

- `flato://protocol/creative-v1`: required canvas authoring guide and Flato MCP contract.
- `flato://prompt-skills`: index of built-in Flato design scenario skills.
- `flato://prompt-skills/general-design-default`: default composition and design workflow.
- `flato://prompt-skills/presentation-cover`: presentation, pitch deck, slides, PPT, proposal, report.
- `flato://prompt-skills/social-visual`: poster, campaign, social post, ad creative.
- `flato://prompt-skills/brand-system`: brand identity, brand kit, guideline, design system.
- `flato://prompt-skills/logo-generation`: logo, icon mark, identity symbol.
- `flato://prompt-skills/motion-design`: motion, animation, timeline, interaction-heavy output.
- `flato://brand-kits`: authenticated user's brand-kit index.
- `flato://brand-kits/{skillId}`: authenticated user's normalized brand-system document.

Expected Flato MCP prompts:

- `create_single_page_design`
- `create_presentation_slide`
- `create_social_visual`
- `create_brand_system_design`
- `apply_brand_kit`
- `revise_selected_block`

When MCP resource listing is available, list resources first and prefer exact listed URIs. When direct resource reading is not available in the host, call `flato_get_creative_protocol` as the protocol fallback and continue with the best available tool/context flow. Do not invent brand-kit ids or prompt-skill ids.

## Required Context Bootstrap

Before any Flato design write:

1. List or read Flato MCP resources.
2. Read `flato://protocol/creative-v1`. If resource reading is unavailable or unreliable, call `flato_get_creative_protocol`.
3. Read `flato://prompt-skills` to see the current design-skill index when resource reading is available.
4. Select and read exactly one primary scenario skill resource before writing:
   - Presentation, pitch deck, deck, slide, proposal, or report: `flato://prompt-skills/presentation-cover`.
   - Poster, social post, campaign visual, ad creative, or marketing image: `flato://prompt-skills/social-visual`.
   - Brand identity, brand guideline, design system, or brand kit: `flato://prompt-skills/brand-system`.
   - Logo or icon mark: `flato://prompt-skills/logo-generation`.
   - Motion, animation, or interaction-heavy design: `flato://prompt-skills/motion-design`.
   - Otherwise: `flato://prompt-skills/general-design-default`.
5. If the user asks to apply a brand, uses brand-like wording, names a company/product, requests on-brand output, or the scenario is brand/logo/guideline/presentation/social marketing, read `flato://brand-kits`.
6. If `flato://brand-kits` returns one clearly relevant brand kit, read `flato://brand-kits/{skillId}`. If multiple are plausible and the user did not name one, ask the user which brand kit to apply before writing branded changes.
7. If the host exposes MCP prompts, invoke the closest workflow prompt with the task brief, selected `brandKitId` when any, and target reference when any:
   - New general page: `create_single_page_design`.
   - Presentation or slide: `create_presentation_slide`.
   - Social/poster/campaign/ad: `create_social_visual`.
   - Brand-system output: `create_brand_system_design`.
   - Applying a brand kit: `apply_brand_kit`.
   - Editing selected/targeted block: `revise_selected_block`.
8. Call `flato_whoami`.
9. Establish a target project:
   - New design: choose a concise saved `projectName`, then call `flato_create_project`.
   - Existing editor URL or project id: call `flato_use_project`.
   - Share URL: call `flato_use_share_link` to create an editable copy.
10. If the response says `waiting_for_live_editor_bridge` or `needs_open_editor`, have the MCP host open `editorUrl` automatically when supported. If the host cannot open it, ask the user to open `editorUrl`.
11. Poll `flato_get_project_status` until `canWrite=true`.
12. Do not call design-context, write, or export tools until `canWrite=true`.
13. Call `flato_get_design_context` before every create/edit sequence.

If the current Codex conversation cannot see `flato_*` tools after setup, treat it as a Codex tool-loading boundary. Ask the user to start a new Codex conversation or restart Codex App, then verify with `flato_whoami`. Do not inspect local OAuth or token stores as a workaround.

## Scenario Skill Usage

The selected `flato://prompt-skills/{skillId}` resource is not decorative reference. Use it to drive content architecture, visual strategy, layout density, typography, image treatment, and QA.

After reading a prompt-skill resource:

- Extract its recommended information architecture.
- Apply its visual strategy to the Flato page or deck.
- Follow its common-mistake warnings.
- Carry its quality checklist into the final QA pass.
- If it conflicts with the user's explicit instruction, follow the user but preserve Flato canvas/tool safety rules.

For mixed tasks, choose one primary scenario skill and optionally read one secondary skill only if it changes execution. Examples:

- Branded pitch deck: primary `presentation-cover`, plus read brand-kit resources; read `brand-system` only if creating or restructuring brand guidelines.
- Social ad using a logo: primary `social-visual`, plus read the selected brand kit; do not use `logo-generation` unless generating a new logo.
- Animated slide: primary `presentation-cover`, plus `motion-design` as secondary.

## Brand-Kit Usage

Brand-kit resources are user-scoped and must be treated as first-class context.

Read `flato://brand-kits` when any of these are true:

- The user mentions brand, brand kit, identity, guideline, logo, company, product, campaign, marketing, sales, pitch, deck, proposal, or social creative.
- The task asks for a polished design and there may be a known brand context.
- The user asks to make output consistent with existing style.
- The design is customer-facing.

After reading a specific `flato://brand-kits/{skillId}`:

- Use brand colors, typography, logo assets, imagery style, copy tone, layout principles, and notes.
- Preserve existing content unless the user asks for a rewrite.
- Use `flato_set_global_style` for reusable brand tokens and shared CSS under `.Canvas`.
- Use block/page styles for local composition.
- If brand-kit constraints are incomplete, state the missing piece and continue with the available brand facts.
 
Do not fabricate brand colors, fonts, logos, or slogans when a brand-kit resource is available. Do not claim a design is branded unless the relevant brand-kit resource was read or the user supplied the brand facts directly.

## Targeting Rules

- Prefer `projectId` for saved projects.
- Use `sessionId` only as a compatibility alias for saved projects.
- Use `clientId` only when the user selected a specific live editor tab.
- Use `draftId` for an unsaved `/editor` canvas.
- Use `socketId` only as a fallback.
- If multiple projects are plausible, ask the user which project to use before writing.
- If the same project is open in multiple tabs, call `flato_list_editors`, ask which tab should execute writes, then call `flato_bind_project_editor` with the confirmed `clientId`.
- Never guess a `pageId` or `blockId`. Use ids returned by a fresh `flato_get_design_context` or `flato_get_state`.
- Selection is only a call-time snapshot. Re-read design context before a selection-dependent edit.

## Read Tools

- Use `flato_get_design_context` as the standard rich context read. It includes effective canvas, current page, selected blocks, page summaries, global style, layout analysis, and `layoutIssueSummary`.
- Use `flato_get_state` for project/page/block detail, `content_outline`, or specific page ids.
- Use `flato_get_style_info` before changing an existing visual system.
- Use `flato_search_cjk_fonts` when Chinese, Japanese, or Korean typography quality matters.
- Use brand-kit resources when brand constraints matter: first `flato://brand-kits`, then `flato://brand-kits/{skillId}`. Brand-kit resources are not optional for branded work.
- Use prompt-skill resources before deciding layout patterns. Do not decide the composition from the user prompt alone when a matching Flato prompt skill exists.

## Write Tools

- Before choosing a write tool, explicitly map the user task plus selected prompt skill into one of: create new pages, update existing page content, update selected block, apply global style, update canvas, asset search/generation, page operation, or export/QA.
- Create new pages with `flato_create_pages`. Create at most 3 pages per call and continue in follow-up calls when needed.
- Update existing pages or blocks with `flato_update_pages`. Every page update requires a real `pageId`.
- Use `updateBlocks[].blockId` for existing blocks and `createBlocks` for new blocks.
- Do not send `canvasConfig` through `flato_update_pages` for existing pages.
- Change canvas size, grid, padding, page/global background defaults, or scope with `flato_update_canvas`.
- Set reusable project CSS with `flato_set_global_style`. CSS must be scoped under `.Canvas`.
- Delete a block with `flato_delete_block`, never by passing deletion fields to `flato_update_pages`.
- Switch, duplicate, delete, or move pages with `flato_page_operation`.
- Use `flato_make_backup` before risky canvas, layout, or broad multi-page changes.

## Layout And Style Rules

- Grid mode is the default for AI-authored layouts.
- In grid mode, `x`, `y`, `width`, and `height` are grid units, not pixels.
- Keep `x + width <= gridCols` and `y + height <= gridRows`.
- Preserve a page's existing layout system when editing an existing page.
- Page-specific background overrides live in page style. Canvas size, grid, padding, position mode, and playback duration live in sparse page canvas overrides.
- Block surface styles belong in `block.style`: background, border, radius, shadow, padding, opacity, filters, and vertical alignment.
- Text block `content` is inner HTML only. Do not create a root wrapper that simulates the resizable block surface.
- Text horizontal alignment belongs to the block text alignment field, not an invented inner wrapper style.

## Image Asset Rules

- `flato_search_image`, `flato_generate_image`, `flato_edit_image`, `flato_understand_image`, and `flato_save_image_by_url` are asset/read tools unless explicitly documented otherwise.
- Asset tools do not mutate the editor. After an asset tool returns an image URL, place or replace it explicitly with `flato_create_pages` or `flato_update_pages`.
- For image replacement, first read the current `pageId`, `blockId`, and existing image URL. Then update the verified image block.
- Do not rely on selected canvas state for image replacement.
- Paid generation/editing runs as the authenticated Flato MCP user and may consume that user's credits.

## Feedback And QA

After a write:

1. Inspect the returned `mcpFeedback` when present.
2. Re-read `flato_get_design_context`.
3. Use `layoutIssueSummary` as the compact QA decision field.
4. Check the result against the selected prompt-skill resource and any brand-kit resource, not only against geometric layout.
5. If `mcpFeedback.status` is `applied`, continue or finalize.
6. If `mcpFeedback.status` is `applied_with_issues`, fix only the reported affected page or block ids. Do not rebuild the whole page unless the user asks.
7. If `mcpFeedback.status` is `no_effect`, re-read context and retry only with verified ids.
8. If `mcpFeedback.status` is `asset_ready`, write the returned asset URL into a page or block before claiming the design was updated.
9. If `mcpFeedback.status` is `failed`, follow its recommended next action or report the specific blocker.

For important generated or revised output, call `flato_export_to_png` with exactly one of `pageId`, `blockId`, or `pageIndex`. Then call `flato_understand_image` on the returned `imageUrl` to inspect composition, readability, overlap, cropping, and visual fit. Iterate only on concrete issues.

Final visual QA must answer:

- Does the page satisfy the selected prompt-skill's structure and quality checklist?
- If a brand kit was read, are colors, type, logo/imagery direction, and tone consistent with it?
- Are all text blocks readable and non-overlapping?
- Are image assets actually placed in the design, not merely generated?
- Did the tool result or `layoutIssueSummary` identify any target-specific issue that still needs a fix?

## Timeout And Recovery

- A browser write can apply while the MCP client receives `Command timeout`. Do not recreate the page. Call `flato_get_design_context` and continue from the actual `pageId` and `blockId`s.
- If `flato_get_project_status` reports ready and `canWrite=true` but `flato_get_design_context` is cancelled, keep the editor tab open, start a new Codex conversation or restart Codex App, then retry `flato_whoami`, `flato_use_project`, `flato_get_project_status`, and `flato_get_design_context`.
- For scripted or background verification, prefer `autoSwitch=false` on `flato_create_pages` to avoid page-switch response timeouts.

## Final Report

When the task is done, report the concrete target and result:

- protocol resource or fallback read
- prompt-skill resource read
- brand-kit resource read, or why none was applicable
- workflow prompt used when available
- `projectId`
- `clientId` when tab-bound
- affected `pageId`s and important `blockId`s
- whether a backup was created
- final `mcpFeedback.status` when available
- final `layoutIssueSummary` result
- exported PNG / visual QA result when performed
