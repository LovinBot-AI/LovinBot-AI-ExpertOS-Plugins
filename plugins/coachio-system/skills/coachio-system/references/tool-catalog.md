# Funnel Config MCP — Tool Catalog

Tools grouped by concern. ⚠ = destructive / real-effect → requires `confirm=true`.
Read tools and content-config tools run freely. Output mirrors the admin REST schemas.

## Products
- `list_products(type?, status?)` — list with totals (type: course|service|digital|other|agents_skill|skills_pack; status: draft|active|archived).
- `get_product(product_id)` — full product incl. linked courses.
- `create_product(name, slug, type, base_price?, status?, description?, thumbnail_url?, course_ids?)` — slug kebab-case + unique; type includes `agents_skill` + `skills_pack`; `course_ids` not allowed for `agents_skill`/`skills_pack`. A funnel needs an existing product, so create one here first (for a `skills_pack`, create SkillPack(s) in the catalog and attach with `attach_skill_pack_to_funnel` after).
- `update_product(product_id, …)` — price/name changes refresh published funnels' landing cache.
- `delete_product(product_id, confirm)` ⚠ — permanent; may affect funnels referencing it.

## Funnels
- `list_funnels(product_id?, status?)` — list with totals.
- `get_funnel(funnel_id)` — full funnel incl. checkout/success config + variables.
- `create_funnel(product_id, title, slug, currency?, zalo_link?)` — starts `draft`.
- `update_funnel(funnel_id, title?, slug?, currency?, zalo_link?, checkout_config?, success_config?, variables?, variables_meta?)` — refreshes landing cache.
- `clone_funnel_tool(funnel_id, slug, title?)` — deep copy into a new draft funnel.
- `set_funnel_status(funnel_id, status, confirm)` ⚠ — publish/unpublish.
- `delete_funnel(funnel_id, confirm)` ⚠ — permanent.

## Landing & SEO
- `get_landing(funnel_id)`, `update_landing_seo(funnel_id, …)`.
- `list_sections(funnel_id)` (slim directory — no html), `get_section(funnel_id, section_id)` (one section WITH html), `create_section(funnel_id, …)`, `update_section(section_id, …)`, `reorder_sections(funnel_id, ordered_ids)`.
- `delete_section(section_id, confirm)` ⚠.
- Section writes + `list_sections` return a slim receipt (`html_length`, not the html body) to stay under the remote-MCP connector's response-size limit; use `get_section` to read a section's current html before editing. `update_section` is a true partial patch — pass only the fields you change.
- Anchors must be unique within a landing; all writes refresh the landing cache.

## Variables
- `get_variables(funnel_id)`, `update_variables(funnel_id, variables, variables_meta?)`.

## Email templates
- `list_email_templates(funnel_id)`, `update_email_template(funnel_id, template_key, enabled?, subject?, html_body?)`, `preview_email_template(funnel_id, template_key)`.
- `test_send_email(funnel_id, template_key, to_email, confirm)` ⚠ — real send to one address.

## Broadcasts
- `list_broadcasts(funnel_id)`, `get_broadcast(cid)`, `create_broadcast(funnel_id, …)`, `update_broadcast(cid, …)`.
- `send_broadcast(cid, confirm)` ⚠ — real dispatch to the audience.

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
- `list_media(kind?, search?, page?, page_size?)`.
- `upload_media(filename, url? | base64_data?)` — upload from a URL or base64.
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

Packages + email composing:
- `list_gifts(include_archived?)`, `get_gift(gift_id)`.
- `create_gift(name, description?, internal_config?, external_items?)` — `internal_config` = `{unlock_agent_skills, studio_credits, enable_api_access, course_ids, skill_pack_ids}` (`course_ids` = list of course ids to enroll the recipient into as an active student; `skill_pack_ids` = list of skill pack ids to grant ownership of, reachable via the recipient's skills dashboard and linked in the gift email via `{{skills_dashboard_url}}`; either alone still creates their account); `external_items` = list of `{label, url, description?}`. A gift needs ≥1 internal perk or 1 external item.
- `update_gift(gift_id, …, is_archived?)`, `archive_gift(gift_id, confirm)` ⚠ — soft-archive (ledger kept); restore via `update_gift(is_archived=false)`.
- `preview_gift_email(gift_ids, email_subject?, email_html?)` / `test_send_gift_email(to_email, gift_ids, …)` — delivery email; tokens `{{recipient_name}}`, `{{agent_skills_url}}`, `{{studio_url}}`, `{{access_token}}` (raw login token — append `?token=` to any route to auto-login, e.g. a gifted course `.../courses-page/<slug>?token={{access_token}}`). The URL tokens are URL only — wrap each in your own `<a>` button; each is empty unless the gift grants that perk / creates an account.
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
- `create_course(title, level, slug?, category_id?, thumbnail_url?, description?, short_description?, …, sale_price?, discount_percent?, product_id?, variables?, variables_meta?, success_config?)` — starts `draft`.
- `update_course(course_id, …subset…)` — does NOT change status; use `publish_course`/`set_course_status`.
- `delete_course(course_id, confirm)` ⚠ — permanent.

## Course categories
- `list_course_categories()`.
- `create_course_category(name, slug?, description?, parent_id?)`, `update_course_category(category_id, …)`.
- `delete_course_category(category_id, confirm)` ⚠ — fails if any course still uses it.

## Course publish + status
- `get_course_publish_readiness(course_id)` — checklist (course detail, curriculum, payment) + `can_publish` + `draft_lesson_count` (lessons publishing will auto-promote).
- `publish_course(course_id, confirm)` ⚠ — gated by readiness; raises a tool error describing what's missing if not ready. Cascades to draft lessons.
- `set_course_status(course_id, status_value, confirm)` ⚠ — `draft`/`archived` only (use `publish_course` to publish).

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

## Zalo reminders
Event = a workshop that fan-outs reminder messages to N Zalo groups. Each event has reminder **slots** (jobs): 8 preset `default` slots auto-created at offsets (`T_MINUS_1D`, `T_MINUS_12H`, `T_MINUS_3H`, `T_MINUS_1H`, `T_MINUS_15M`, `T_MINUS_5M`, `AT_TIME`, `T_PLUS_5M`) plus ad-hoc `CUSTOM` slots. A slot stays `draft` (no send) until **registered** with n8n → `pending` → `sent`. All times are **VN local** (naive, e.g. `2026-07-15T20:00:00`).
- `zalo_connection_status()` — read-only; session status (connected|disconnected|unknown), zalo_name, heartbeat.
- `list_zalo_groups()` — read-only; cached groups (group_id, name, member count).
- `refresh_zalo_groups()` — force-sync group cache from the live Zalo session (n8n); rate-limited to once/30s.
- `list_zalo_events(status?, q?)` — read-only; events newest-first with `sent_count`/`pending_count`; filter by status (`draft`|`scheduled`|`cancelled`) or name `q`.
- `get_zalo_event(event_id)` — read-only; event + all its slots.
- `create_zalo_event(name, event_at_vn, zalo_group_ids, group_names?, slots?)` — creates the event + auto-spawns 8 default draft slots (template messages; nothing sends yet). Optional `slots` items `{slot_key, message, media_urls?, slot_name?, send_at_vn?}`: a default `slot_key` OVERRIDES that auto-slot's content; `slot_key='CUSTOM'` ADDS an ad-hoc slot (needs `send_at_vn` + `slot_name`). Returns `{event, jobs}`.
- `update_zalo_event(event_id, name?, event_at_vn?, zalo_group_ids?, group_names?)` — partial event edit.
- `add_zalo_reminder_slots(event_id, slots)` — add draft slots (same `slots` shape) to an existing event.
- `update_zalo_reminder_slot(job_id, message?, media_urls?, slot_name?, send_at_vn?)` — edit one slot; service enforces slot-kind × status rules (default slots reject reschedule; editing a pending slot re-registers on n8n).
- `register_zalo_reminder_slots(job_ids, confirm)` ⚠ — register draft slots with n8n for REAL scheduled sending; promotes event draft→scheduled. Partial: returns `{registered, skipped, failed}`.
- `cancel_zalo_reminder_slots(job_ids, confirm)` ⚠ — cancel pending slots (removes their n8n schedule); returns `{cancelled, skipped}`.
- `cancel_zalo_event(event_id, confirm)` ⚠ — cancel the event + cascade-cancel its pending slots.
- `delete_zalo_event(event_id, confirm)` ⚠ — hard-delete event + slots; rejected if it still has pending slots (cancel first).

## Skill packs (standalone catalog + funnel attach)
A `skills_pack` product sells access to one or more **SkillPacks** — a standalone,
reusable catalog entity (title/description/banner + one connected **private GitHub
repo**), independent of any funnel. The same pack can attach to many funnels, and one
funnel can sell many packs (M:N). Admin connects a pack's repo (read-only PAT, stored
encrypted, never echoed) and **syncs** an item manifest; a purchase records a
`SkillPackGrant` per attached pack, unlocking the WHOLE pack. Buyers browse/download
items from their authenticated learner dashboard — NOT the funnel landing. An "item" =
a repo folder containing a `SKILL.md` (content) or a `.zip` (zip); files > 100 MB are
flagged unservable.

**Catalog** (independent of any funnel):
- `list_skill_packs(status?)` — read-only; catalog packs newest-first.
- `get_skill_pack(skill_pack_id)` — read-only; metadata + non-secret repo connection (provider, repo, ref, root_path, connected, synced sha/time, items_count). Never returns the token.
- `create_skill_pack(title, description?, banner_url?, status?)` — creates an empty pack, not attached to any funnel yet.
- `update_skill_pack(skill_pack_id, title?, description?, banner_url?, status?)`.
- `delete_skill_pack(skill_pack_id, confirm)` ⚠ — blocked (409) if any buyer already owns it (has a grant) or it's still attached to a funnel; detach first.
- `connect_skill_pack_repo(skill_pack_id, repo, token, ref?, root_path?, provider?, confirm)` ⚠ — validate reachability + store the encrypted PAT. Real effect (stores a secret) → `confirm=true`.
- `sync_skill_pack(skill_pack_id)` — walk the repo tree, upsert items (preserve admin overrides, drop stale), record the commit sha. Returns `{total, zip, content, unservable, removed, truncated}`.
- `list_skill_pack_items(skill_pack_id)` — read-only; synced items (id, skill_key, title, kind `zip`|`content`, is_visible, is_servable, sort), incl. hidden/unservable.
- `edit_skill_pack_item(skill_pack_id, item_id, title?, description?, is_visible?, sort?)` — curate a synced item (no repo change).

**Funnel attach** (`skills_pack` funnels only — 400 otherwise):
- `attach_skill_pack_to_funnel(funnel_id, skill_pack_id)` — adds at the end of the current order; no-op if already attached.
- `detach_skill_pack_from_funnel(funnel_id, skill_pack_id)` — no-op if not attached.

**Delivery** (buyer-facing; replaces v1's landing + localStorage flow). Buyers see and
open owned packs from their authenticated learner dashboard, gated by session (401) +
a per-pack grant (403) on every call:
`GET /me/skill-packs` (owned packs), `.../skill-packs/{packId}/items`,
`.../items/{itemId}/tree`, `.../items/{itemId}/file?path=`, `.../items/{itemId}/download`
(→ `{url}` short-lived signed URL). The sales landing may additionally show a public,
unauthenticated teaser of the attached packs (no items, no repo identity):
`GET /public/funnels/{slug}/skill-packs` — attached + `active` packs only
(id/title/description/banner_url).

> Exact tool names may vary slightly; list the live tools with the client's tool browser
> if a name does not resolve.
