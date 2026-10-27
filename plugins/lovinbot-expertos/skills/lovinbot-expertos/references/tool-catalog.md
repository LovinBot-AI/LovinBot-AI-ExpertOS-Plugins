# Funnel Config MCP — Tool Catalog

Tools grouped by concern. ⚠ = destructive / real-effect → requires `confirm=true`.
Read tools and content-config tools run freely. Output mirrors the admin REST schemas.

## Products
- `list_products(kind?)` — list with totals; `kind` = `course`|`skill_pack`|`external`. Each product: id, name, description, base_price, `deliverable` ({type, id, title, slug} or null), `kind`.
- `get_product(product_id)` — one product incl. its deliverable.
- `create_product(name, base_price?, description?, deliverable_type?, deliverable_id?)` — a product is the offer (name/price) + what buyers receive: ONE course or ONE skill pack (`deliverable_type` + `deliverable_id`), or nothing in-system (omit / `deliverable_type="external"`) — external products are delivered by the funnel's email templates + success page. A funnel needs an existing product, so create one here first.
- `update_product(product_id, name?, description?, base_price?, deliverable_type?, deliverable_id?)` — omit `deliverable_type` to keep the deliverable; `"external"` clears it. A new deliverable applies to future buyers of every funnel selling the product; past buyers keep theirs. Price/name changes refresh published funnels' landing cache.
- `delete_product(product_id, confirm)` ⚠ — permanent; its course / pack is kept. Blocked (`product_has_funnels`) while any funnel uses it.

## Funnels
- `list_funnels(product_id?, status?)` — list with totals.
- `get_funnel(funnel_id)` — full funnel incl. checkout/success config + variables.
- `create_funnel(product_id, title, slug, currency?, zalo_link?, type?)` — starts `draft`. `type` is `standard` (default) or `record_workshop`.
- `update_funnel(funnel_id, title?, slug?, currency?, zalo_link?, checkout_config?, success_config?, variables?, variables_meta?)` — refreshes landing cache. `checkout_config` is replaced whole (read `get_funnel` first): `template` (`split-hero`|`header-band`|`order-sidebar`), `section_max_width` (880|1024|1200), `headline`, `message`, `custom_html`, `use_custom_html` (bool — off keeps the HTML but shows headline/message), `pay_minutes` (15|30|60, default 15).
- `clone_funnel_tool(funnel_id, slug, title?)` — deep copy into a new draft funnel.
- `set_funnel_status(funnel_id, status, confirm)` ⚠ — publish/unpublish.
- `delete_funnel(funnel_id, confirm)` ⚠ — permanent.

## Record workshop gated content

Funnels created with `type: "record_workshop"` gate their landing content behind a
purchase check. The video ids / doc urls live server-side and are only released to
emails that match the access rule — they are never rendered into the landing HTML.

- `get_funnel_gated_content(funnel_id)` — read the current rule + resources
- `update_funnel_gated_content(funnel_id, enabled, source_funnel_ids, allowed_statuses, resources, min_amount_vnd?, include_self?, expires_at?, deny_message?, fallback_enabled?, fallback_statuses?, fallback_resources?, fallback_message?)` — full replace
- `test_funnel_gated_access(funnel_id, email)` — dry-run the rule for one email
- `list_content_unlock_attempts(funnel_id, email?, tier?, limit?, offset?)` — the audit
  trail: who tried, what tier they got, why they were refused. Newest first; `summary`
  counts every attempt on the funnel and ignores the filters. Start here for
  "I paid but cannot watch".

`allowed_statuses` accepts `subscribed`, `lead`, `purchased`. A lead counts as
`purchased` when it has a SUCCESS funnel order above `min_amount_vnd` (default
10.000 VND) on one of `source_funnel_ids`.

### Preview tier (optional)

`fallback_*` gives visitors who FAIL the paid rule a second, separate set of
resources instead of a flat refusal — a teaser, or the same recording if you
repeat the value. It reuses `source_funnel_ids` and `include_self`; only the
statuses differ, and a status listed in both is always taken by the paid rule
first (so the preview never fires for it).

`update_funnel_gated_content` is a **full replace**: read the current config with
`get_funnel_gated_content` and pass every field back, or an omitted argument (the
preview tier, the expiry, the deny message) silently resets to its default.

`test_funnel_gated_access` returns `tier`: `full` | `preview` | `none`. Only
`full` sets `granted: true` — a preview is audited as not granted, so counts of
paid access stay accurate.

Warning: turning on `include_self` together with `subscribed` in
`allowed_statuses` disables the paid gate — the unlock endpoint creates a
`subscribed` lead on the gated funnel before evaluating, so everyone who submits
the form would pass. The same pair in `fallback_statuses` is fine and is exactly
how you say "anyone who fills the form gets the preview".

## Landing & SEO
- `get_landing(funnel_id)`, `update_landing_seo(funnel_id, …)`.
- `list_sections(funnel_id)` (slim directory — no html), `get_section(funnel_id, section_id)` (one section WITH html), `create_section(funnel_id, …)`, `update_section(section_id, …)`, `reorder_sections(funnel_id, ordered_ids)`.
- `delete_section(section_id, confirm)` ⚠.
- Section writes + `list_sections` return a slim receipt (`html_length`, not the html body) to stay under the remote-MCP connector's response-size limit; use `get_section` to read a section's current html before editing. `update_section` is a true partial patch — pass only the fields you change.
- Anchors must be unique within a landing; all writes refresh the landing cache.

## Course & Site SEO
- `get_course_seo(course_id)` — read raw SEO fields + resolved metadata for a course.
- `update_course_seo(course_id, …)` — update course SEO fields (empty string clears; omitting leaves unchanged); returns raw + resolved metadata. Prompts: `seo_audit_course`.
- `get_site_seo()` — read raw SEO fields + resolved metadata for the root site homepage.
- `update_site_seo(…)` — update root site homepage SEO fields (empty string clears; omitting leaves unchanged); returns raw + resolved metadata. Prompts: `seo_setup_site`.


## Variables
- `get_variables(funnel_id)`, `update_variables(funnel_id, variables, variables_meta?)`.
- `variables_meta` = `{key: {name?, description?, type?}}`. `type` is `text` (default) | `number` | `date` | `time` | `datetime` | `link`; values are checked per type (number `-12.5`, date `YYYY-MM-DD`, time `HH:MM`, datetime `YYYY-MM-DDTHH:MM`, link must start with `http://` or `https://`; empty is allowed). The `{{key}}` substitution is always the raw string.
- Reserved system keys are rejected as custom keys (error, nothing saved): `product_name`, `funnel_title`, `price`, `discounted_price`, `discount_percent`, `checkout_url`, `success_url`, `zalo_link`, `skills_dashboard_url`, `customer_name`, `customer_email`, `order_code`, `order_total`, `brand_name`.

## Email templates
- `list_email_templates(funnel_id)`, `update_email_template(funnel_id, template_key, enabled?, subject?, html_body?)`, `preview_email_template(funnel_id, template_key)`.
- `test_send_email(funnel_id, template_key, to_email, confirm)` ⚠ — real send to one address. Sends from the tenant's verified sending domain when one is verified (Settings → Email); otherwise the platform sender. Returns `[code: recipient_suppressed]` (422, address on suppression list) or `[code: daily_limit_reached]` (429, platform daily cap) instead of a false `sent`; never gated by 409 (sender AUTO).
- Editable keys: `receipt`, `waiting_payment` only (course / skill-pack credential emails use built-in wording). Variables include `{{brand_name}}` (tenant name).

## Broadcasts
- `list_broadcasts(funnel_id)`, `get_broadcast(cid)`, `create_broadcast(funnel_id, …)`, `update_broadcast(cid, …)`.
- `send_broadcast(cid, confirm)` ⚠ — real dispatch to the audience. Requires a verified sending domain (Settings → Email); otherwise returns code `email_sending_not_connected` (409). Scheduled campaigns wait while no sending domain is verified and resume once it is. Suppressed recipients and send-time limits are not reported in this response.

## Leads (read-only)
- `list_leads(funnel_id?, status?, email?, converted?, created_from?, created_to?, …)`.
- `export_leads(…same filters…)` — returns CSV text; deliver to the operator only.

## Discounts
- `list_discounts(…)`, `get_discount(discount_id)` — includes scopes + default owners.
- `create_discount(code, type, value, min_amount?, usage_limit?, scopes?, defaults?)`, `update_discount(discount_id, …)`.
- `set_discount_scope(discount_id, scopes)` — replace the applicability whitelist; empty = global.
- `set_discount_default(discount_id, owner_type, owner_id)` — auto-apply; must be within a non-empty scope (else error).
- `unset_discount_default(discount_id, owner_type, owner_id)`.
- `delete_discount(discount_id, confirm)` ⚠.

## Media
- `list_media(kind?, search?, page?, page_size?)` — newest first; items carry `width`/`height` (px, null for legacy uploads).
- `upload_media(filename, url? | base64_data?)` — upload from a URL or base64; jpg/png/webp/gif ≤ 15 MB, must be a valid image (else 400). Returns the asset incl. `width`/`height`.
- `delete_media(asset_id, confirm)` ⚠.

## Analytics & Orders (read-only)
- `get_funnel_analytics(funnel_id, start_date?, end_date?)` — window capped at 31 days.
- `funnel_orders_summary(…)`, `list_funnel_orders(…)`, `get_funnel_order(order_id)`.

## Capture token
- `get_capture_token(funnel_id)` — current token (generated if absent).
- `rotate_capture_token(funnel_id, confirm)` ⚠ — old token stops working immediately.

## Lucky draw
- `list_lucky_events(funnel_id?)` — events with participant/winner counts.
- `get_lucky_event(event_id)` — full event incl. form schema.
- `create_lucky_event(funnel_id, title, slug?, form_schema?, name_field_key?, success_config?, display_config?)` — starts `draft`; `form_schema` is a list of field dicts; one short_text field is the display name.
- `update_lucky_event(event_id, …)` — re-validates the form schema.
- `set_lucky_event_status(event_id, action)` — `open` / `lock` registration.
- `delete_lucky_event(event_id, confirm)` ⚠ — removes prizes/participants/winners too.
- `get_lucky_token(event_id)`, `rotate_lucky_token(event_id, confirm)` ⚠ — public registration token.
- `list_lucky_prizes(event_id)`, `create_lucky_prize(event_id, name, quantity?, sort_order?)`, `update_lucky_prize(event_id, prize_id, …)`, `delete_lucky_prize(event_id, prize_id, confirm)` ⚠.
- `list_lucky_participants(event_id)`, `add_lucky_participant(event_id, display_name, phone?, answers?)`, `remove_lucky_participant(event_id, participant_id, confirm)` ⚠.
- `spin_lucky_prize(event_id, prize_id, confirm)` ⚠ — live draw; returns winner (id, display_name, phone, email, prize, spin_order). Undo with `discard_lucky_winner`.
- `list_lucky_winners(event_id)` — winners incl. id, phone, email (derived from participant; empty when not collected).
- `discard_lucky_winner(event_id, winner_id, confirm)` ⚠ — discard a winner (e.g. absent), removes the participant and frees the prize to redraw.

## Gifts
Gift packages (perks + external links) delivered two ways: **automations** (auto-grant on a
funnel trigger) and **campaigns** (one-off send to a targeted audience). A grant ledger makes
every delivery once-per-recipient; resends re-send the email only, never re-grant perks.
Email gate: campaign, automation and gift delivery emails require a verified sending domain (Settings → Email). Without it these return code `email_sending_not_connected` (409): `send_gift_campaign`, `retry_gift_campaign`, `resend_gift_grant`, `bulk_retry_gift_grants`, `create_gift_automation` / `update_gift_automation` (only when the result is active), `test_send_gift_email`. A grant made while the domain is not verified keeps its perks; its email is `skipped` (`email_not_connected`) and is not auto-retried, so resend it once the domain is verified. `test_send_gift_email` can also return `[code: recipient_suppressed]` (422) or `[code: daily_limit_reached]` (429, platform daily cap). `test_send_gift_account_email` (login email) is not gated and falls back to the platform sender, but can return the same 422/429 codes.

Packages + email composing:
- `list_gifts(include_archived?)`, `get_gift(gift_id)`.
- `create_gift(name, description?, internal_config?, external_items?)` — `internal_config` = `{course_ids, skill_pack_ids}` (`course_ids` = list of course ids to enroll the recipient into as an active student; `skill_pack_ids` = list of skill pack ids to grant ownership of, reachable via the recipient's skills dashboard and linked in the gift email via `{{skills_dashboard_url}}`; either alone still creates their account); `external_items` = list of `{label, url, description?}`. A gift needs ≥1 internal perk or 1 external item.
- `update_gift(gift_id, …, is_archived?)`, `archive_gift(gift_id, confirm)` ⚠ — soft-archive (ledger kept); restore via `update_gift(is_archived=false)`.
- `preview_gift_email(gift_ids, email_subject?, email_html?)` / `test_send_gift_email(to_email, gift_ids, …)` — delivery email; tokens `{{recipient_name}}`, `{{access_token}}` (raw login token — append `?token=` to any route to auto-login, e.g. a gifted course `.../courses-page/<slug>?token={{access_token}}`). The URL tokens are URL only — wrap each in your own `<a>` button; each is empty unless the gift grants that perk / creates an account.
- `preview_gift_account_email(email_subject?, email_html?)` / `test_send_gift_account_email(to_email, …)` — new-account login email; tokens `{{recipient_name}}`, `{{login_email}}`, `{{password}}`, `{{login_url}}` (URL only — wrap in your own `<a>` button). Blank → default template.

Automations (mechanism 1):
- `list_gift_automations()`, `get_gift_automation(automation_id)`.
- `create_gift_automation(trigger_status, gift_ids?, funnel_id?, is_active?, max_total_grants?, email_*?, account_email_*?)` — `funnel_id` null = all funnels. ⚠ non-`purchased` triggers grant paid perks to unpaid leads.
- `update_gift_automation(automation_id, …)`, `delete_gift_automation(automation_id, confirm)` ⚠.

Campaigns (mechanism 2) — lifecycle create → preview → confirm → send:
- `list_gift_campaigns()`, `get_gift_campaign(campaign_id)`.
- `create_gift_campaign(name, gift_ids?, email_*?, account_email_*?, audience_config?, scheduled_at?)`, `update_gift_campaign(campaign_id, …)` (draft only).
- `preview_gift_audience(campaign_id? | gift_ids? + audience_config?)` — recipient breakdown `{matched, already_granted, will_receive, sample}`.
- `confirm_gift_campaign(campaign_id)` — freeze recipient snapshot into send jobs.
- `send_gift_campaign(campaign_id, scheduled_at?)` — send now or schedule (ISO with +07:00).
- `cancel_gift_campaign(campaign_id)`, `retry_gift_campaign(campaign_id)`, `gift_campaign_stats(campaign_id, failed_page?, failed_size?)`.
- `delete_gift_campaign(campaign_id, confirm)` ⚠.

Grant tracking / audit:
- `list_gift_grants(page?, size?, gift_id?, funnel_id?, source_type?, email_status?, email?, content?, new_account_created?, date_from?, date_to?)`, `gift_grant_stats(…same filters…)`, `gift_grant_detail(grant_id)`.
- `resend_gift_grant(grant_id)`, `bulk_retry_gift_grants(…filters…)` — re-send failed emails only.

## Preview
- `get_landing_preview(funnel_id)` — returns the admin `preview_url` (open in browser) and a
  draft snapshot; use before publishing. Does not change publish status.

## Courses
- `list_courses(status?, category_id?)`, `get_course(course_id)`.
- `create_course(title, level, slug?, category_id?, thumbnail_url?, description?, short_description?, …, sale_price?, discount_percent?)` — starts `draft`. To sell it, set it as a product's deliverable.
- `update_course(course_id, …subset…)` — does NOT change status; use `publish_course`/`set_course_status`.
- `delete_course(course_id, confirm, force?)` ⚠ — permanent. Blocked if learners are enrolled (`deliverable_has_learners` — archive instead, no override). Blocked while a published funnel sells it (`deliverable_on_sale`) unless `force=true`, which also switches those products to external.

## Course categories
- `list_course_categories()`.
- `create_course_category(name, slug?, description?, parent_id?)`, `update_course_category(category_id, …)`.
- `delete_course_category(category_id, confirm)` ⚠ — fails if any course still uses it.

## Course publish + status
- `get_course_publish_readiness(course_id)` — checklist (course detail, curriculum, payment) + `can_publish` + `draft_lesson_count` (lessons publishing will auto-promote).
- `publish_course(course_id, confirm)` ⚠ — gated by readiness; raises a tool error describing what's missing if not ready. Cascades to draft lessons.
- `set_course_status(course_id, status_value, confirm)` ⚠ — `draft`/`archived` only (use `publish_course` to publish). Archiving keeps learners' access but does NOT stop funnel sales; unpublish the funnels that still sell the course to stop selling.

## Course curriculum
- `get_curriculum(course_id)` — sections + lessons + materials.
- `create_course_section(course_id, title, description?, sort_order?)`, `update_course_section(section_id, …)`, `delete_course_section(section_id, confirm)` ⚠.
- `create_lesson(section_id, title, lesson_type, lesson_status?, youtube_url?, pdf_url?, banner_url?, …)` — type: `text_lesson`|`pdf_lesson`|`video_lesson`; media via URL only, no file-upload tool.
- `update_lesson(lesson_id, …subset…)`, `delete_lesson(lesson_id, confirm)` ⚠.

## Course students & enrollment
- `list_course_students(course_id, page?, page_size?, email?)` — read-only; current students (buyers) with learning progress; `email` is a substring filter.
- `enroll_course_students(course_id, students, confirm)` ⚠ — bulk-enroll by email; `students` = list of `{email, name?, phone?}`. Creates a learner account when missing (sends credentials), grants active access (idempotent), sends onboarding email. Returns a summary + per-email results (`created_account` | `enrolled` | `already_enrolled` | `failed`).
- `preview_course_lead_audience(course_id, funnel_ids, statuses)` — read-only; preview leads across multiple funnels + statuses (`subscribed`|`lead`|`purchased`) that would be enrolled — each with email, funnel, `has_account`, `already_enrolled`. Creates/sends nothing.
- `enroll_course_leads(course_id, funnel_ids, statuses, confirm)` ⚠ — enroll matching leads (skips already-enrolled) from the given funnels/statuses; same account-resolution + onboarding-email behavior as `enroll_course_students`. Capped at 500. Preview first with `preview_course_lead_audience`.
- `unenroll_course_student(course_id, user_id, confirm)` ⚠ — soft-drop a student from the course (keeps history, sends no email); idempotent, returns state `dropped` | `already_dropped` | `not_enrolled`.

## Skill packs (standalone catalog)
A product sells ONE **SkillPack** when `deliverable_type="skill_pack"`; one pack can be sold by many products. A SkillPack is a standalone, reusable catalog entity
(title/description/banner + one connected **private GitHub repo**); the same pack can
be sold by many products. Admin connects a pack's repo through a **GitHub App installation** (read-only, repo-scoped
short-lived tokens; no secret stored) and **syncs** an item manifest; a purchase records a
a `SkillPackGrant` for that pack, unlocking the WHOLE pack. Buyers browse/download
items from their authenticated learner dashboard — NOT the funnel landing. An "item" =
a repo folder containing a `SKILL.md` (content) or a `.zip` (zip); files > 100 MB are
flagged unservable.

**Catalog** (independent of any funnel):
- `list_skill_packs(status?)` — read-only; catalog packs newest-first.
- `get_skill_pack(skill_pack_id)` — read-only; metadata + non-secret repo connection (provider, repo, ref, root_path, connected, synced sha/time, items_count). Never returns credentials.
- `create_skill_pack(title, description?, banner_url?, status?)` — creates an empty pack, not sold by any product yet.
- `update_skill_pack(skill_pack_id, title?, description?, banner_url?, status?)` — `status='archived'` keeps buyers' access but does NOT stop funnel sales; unpublish the funnels that still sell the pack to stop selling.
- `delete_skill_pack(skill_pack_id, confirm, force?)` ⚠ — blocked if any buyer owns it (`deliverable_has_learners` — archive instead, no override); blocked while a published funnel sells it (`deliverable_on_sale`) unless `force=true`, which also switches those products to external.
- `list_github_installations(include_repos?)` — read-only; GitHub accounts connected to the workspace: `{installation_id, account_login, account_type, status, repos_count, repos_synced_at, packs_using, repos?}`; `include_repos=true` adds the allowed `full_name` list. MCP cannot install or refresh access: the admin connects GitHub once in **Settings › GitHub accounts** (browser). A repo missing from the list → admin uses "Refresh repository access" there.
- `connect_skill_pack_repo(skill_pack_id, installation_id, repo, ref?, root_path?, provider?, confirm)` ⚠ — `repo` ('owner/name' or URL) must be in that installation's allowed repos; validates reachability, stores no token. Real effect → `confirm=true`. Flow: `list_github_installations(include_repos=true)` → `connect_skill_pack_repo(confirm=true)` → `sync_skill_pack`.
- `sync_skill_pack(skill_pack_id)` — walk the repo tree, upsert items (preserve admin overrides, drop stale), record the commit sha. Returns `{total, zip, content, unservable, removed, truncated}`.
- `list_skill_pack_items(skill_pack_id)` — read-only; synced items (id, skill_key, title, kind `zip`|`content`, is_visible, is_servable, sort), incl. hidden/unservable.
- `edit_skill_pack_item(skill_pack_id, item_id, title?, description?, is_visible?, sort?)` — curate a synced item (no repo change).

**Delivery** (buyer-facing; replaces v1's landing + localStorage flow). Buyers see and
open owned packs from their authenticated learner dashboard, gated by session (401) +
a per-pack grant (403) on every call:
`GET /me/skill-packs` (owned packs), `.../skill-packs/{packId}/items`,
`.../items/{itemId}/tree`, `.../items/{itemId}/file?path=`, `.../items/{itemId}/download`
(→ `{url}` short-lived signed URL). The sales landing may additionally show a public,
unauthenticated teaser of the product's pack (list of 0..1; no items, no repo identity):
`GET /public/funnels/{slug}/skill-packs` — the product's pack, only if `active`
(id/title/description/banner_url).

> Exact tool names may vary slightly; list the live tools with the client's tool browser
> if a name does not resolve.
