---
name: r3
description: Build or change an R3 project (3D scenes, physics, audio, music) through the R3 MCP server. Use when the user asks to list, find, create, open, inspect, edit, script or preview an R3 project, or mentions R3, r3js.org or an R3 entityRef.
---

# Working in R3 through MCP

The `r3` MCP server this plugin connects is the authority on R3. At connect it sends its own
instructions, and every tool carries a description. Read those first: this skill only gives the
order of work and the rules that hold across tools.

You have no R3 in your pretraining. Before writing any code, call `r3_docs_list`, then
`r3_docs_read` (start with `CODE_PATTERNS.md` and `api`). A guessed R3 API saves fine and then
silently does nothing.

Before changing a component you haven't used, read its docs:
`r3_docs_read("api/component/<Type>")` (e.g. `api/component/VehicleWheel`). It lists every
option with its type, default, range, whether a change is live (and which system reacts) or
passive, and the component's methods.

## The order of work

1. **`r3_whoami`.** Confirm which R3 account you act for (through an app connection or an agent
   key) and which capabilities it has.
   Never attempt a tool the capabilities do not cover.
2. **Find or make the project.** `r3_project_list` lists the account's own and its teams'
   projects with their ids, last updated first. When the user names a project, search for it by name; if several
   match, show the candidates (name, team, last saved) and ask which one. Never guess between
   them. Use `r3_project_create` for a new project.
3. **`r3_project_load`** before you touch anything. R3 addresses every component by its
   `entityRef`, and this is the tool that tells you those names. It gives you names, not values.
4. **Change the project:**
   - `r3_component_read` before a change relative to what is there ("make it brighter"), and
     after a write to confirm what landed.
   - `r3_component_set` to change a property of something that already exists. Do not set scene
     properties from a Code component.
   - `r3_factory_list` / `r3_factory_run` to add objects; load the project again afterwards to
     learn their entityRefs.
   - `r3_code_write` to make things *do* something (`r3_code_read` shows a script's source,
     event and `codeVersion` first). Take keyboard, mouse and touch input from R3's events
     (`KEY_DOWN`, `MOUSE_CLICK`, `TOUCH_START` … as the script's event), never from a
     `window` / `document` `addEventListener`: `CODE_PATTERNS.md` → *R3 input events, not DOM
     listeners*. An input R3 does not report is an `r3_bug_report`.
   - `r3_blender_list` / `r3_blender_import` for real 3D models (a Blender model is one .blend
     the user uploaded; import it whole, or only the `meshes` you need), and `r3_image_list` /
     `r3_audio_list` for the account's own assets. Do not approximate a listed model out of
     primitives.
   - `r3_help_write` for the project's controls. Do not build your own help overlay.
5. **`r3_preview_create`**, then give the user the preview link. You cannot see what you build;
   the preview is how a person checks it (it shows your changes when reloaded).

Every tool call is a complete round trip against the project. Nothing is held between calls, and
there is nothing to save: every change is **kept** the moment you make it, in the project's live
copy, which every R3 tab with the project open shows at once (see *Every change is kept*, below).

### A changed Blender model: the user decides which projects use it

A re-uploaded .blend is always the model's latest version, but the user's projects keep the
meshes they have until the user says otherwise. **`r3_blender_update`** with
`choice: "update_projects"` and no `reportId` shows which projects (and entity templates) use an
older version and what changed; tell the user, ask them, and only then call it again with the
`reportId` and the `projectIds` / `entityTemplateIds` they chose. "Not now" is no call. The plan
only reads; the second call needs the `blender:update` right (off by default for a key, on for the
R3 terminal). Only the mesh shapes and materials change; each
project keeps its own placement, and published releases never change.

### Code is version-checked, never merged

A script's code and the project's help text each have a version, `codeVersion`. `r3_code_read`
answers it for a script (so does `r3_editor_inspect`), `r3_component_read` for the Help component.
To REPLACE one, pass that number as `baseVersion` to `r3_code_write`, `r3_editor_code` or
`r3_help_write`. A new script or the first help needs none (it starts at version 1).

If someone changed it since you read it (the user in the editor, another agent), the write is
refused, never merged, and the answer carries the current text and version and who changed it.
Apply your change to THAT text and write again with the new `baseVersion`. Never answer a
refusal by re-sending your old text on the new version without reading it: that throws their
change away. A write without `baseVersion` is refused with "read it first".

## Going live

A preview link is private and lasts 7 days. Going live makes the project **public** at its live
URL, for anyone. Do it only when the user asks.

**Changing the project does NOT change the live site; `r3_publish_live` does.** The live URL
shows a *release*, a frozen copy of the project. Every change (yours or the user's) changes the
project and its preview links, never the live page. Never tell the user "it's live now" after a
change.

- **`r3_publish_live`** makes a new release of the project as it is now and makes it live (it needs the
  `publish:live` right, which is off unless the user ticked it for this key). On a project that is
  already live this is a *publish update*: the answer names the release number, the release it
  replaced and what changed (components added, removed and changed by name, code files). Pass
  that on. An optional `note` ("new cue and rules") is stored with the release.
- **`r3_release_list`** lists the releases and versions (number, date, who, note, which one is
  live) and says whether there are changes that are not live yet.
- **`r3_release_rollback`** makes an earlier release live again, only when the user asks. Nothing
  is deleted.
- Each live project uses one slot of the project owner's plan (a publish update or a rollback
  uses none). If the plan is full, or there is
  no plan, nothing is published: the answer says how many live projects the plan allows, how
  many are live, and gives the page where the user picks a plan. Pass that on, and let the user
  decide (another project offline, or a bigger plan).
- **Never ask for or accept card details.** A plan is bought by the user on R3's own page.
- **`r3_unpublish_live`** takes a live project offline again, only when the user asks. Nothing is
  deleted and any preview link keeps working.
- **`r3_publish_settings_read`** reads a project's publish settings (the app's Deploy → Settings):
  what it works on (keyboard, touch, both or not tested), its SEO title and description, and how
  the live page starts. It also says what a scan of the code suggests for "works on": only a
  suggestion, never a fact. Confirm it with the user (or test it) before setting it.
- **`r3_publish_settings_set`** changes only the settings you name (it needs `publish:live`, like
  publishing). **SEO title and description go to the user for approval first**: propose the text,
  set it only once they agree. `worksOn: "keyboard"` shows phone visitors a "made for a keyboard"
  note and gives the project a "desktop" badge; `touch` a "mobile" badge. Invalid values are refused
  with every problem named. The answer says what changed; open live and preview pages pick it up
  without a new release.
- **`r3_preview_remove`** removes a project's preview link (every link to it stops working; a live
  project stays live), only when the user asks. A project whose preview link still works cannot
  be deleted: if the user wants it gone, the link goes first.

## The user's open editor

When the user has R3 open in a browser, `r3_whoami` reports the editor and the projects open in it.

- **`r3_editor_open`** shows the user a project: it loads it into their editor (if it is not open
  there already) and focuses it. Use it when the user wants to *see* a project ("open it", "show
  me"), and before inspecting one that is not open.
- **`r3_editor_inspect`** reads what a running tab holds right now, runtime values included, like
  a moving body's position. Use it to debug live behaviour ("why does the car drift", "where is
  the ball now"); narrow it with `type`, `entityRef`, `ids` and `fields`. `r3_component_read`
  reads the project's kept values without a tab.
- Inspect, run, stop and capture need a tab **with that project open**. With none, the refusal
  says "Open the project in R3 first": ask the user, or `r3_editor_open` it. Do not loop.
- Every answer from the editor names the **tab it reached**: when it was opened, how long ago the
  user last used it, and the projects open there (of the tabs that have the project open, the one
  the user used last). If that is not the one they are looking at, tell them to click into it.

### Run it, look at it

You can test and **see** what you build in the open editor:

- **`r3_editor_run`** presses the editor's Run buttons — physics, code, audio, music (all, or the
  ones named in `systems`); a paused simulation resumes. **`r3_editor_stop`** presses Stop (physics
  goes back to where things were placed). Neither changes the project. They need the
  `editor:run` right (on by default; a key made before it existed lacks it — the refusal says to
  create a new key).
- **`r3_editor_capture`** returns a **picture** of the project's canvas, as its camera shows it now
  (running systems included), at most 1280 px. **Capture after every visual
  change** to check it landed as you meant — before telling the user it is done — and after
  `r3_editor_run` to see it move. A blank picture means nothing is in the camera's view.

### Every change is kept

Every edit is kept the moment you make it, in the project's **live copy**: the same copy every R3
tab with the project open shows (the user's laptop, their phone), so they see each change land.
No tab is needed, and **there is nothing to save**. Each write answers what was kept and where in
the project's history (`Kept · seq 57`, and any name R3 had to change because it was taken).

- **`r3_component_set`** changes properties (`r3_editor_set` is the same tool under its older
  name; it also takes a `componentId`).
- **`r3_code_write`** writes or replaces a script (`r3_editor_code` is its older name). Pass the
  `codeVersion` you read as `baseVersion`.
- **`r3_factory_run`** adds an object (`r3_editor_create`); **`r3_component_add`** adds ONE
  component of any registered type (`r3_editor_add_component`): a collider, a rigid body, a
  joint, a light, a material …, with its values and references, e.g. `{"type": "ColliderCuboid",
  "values": {"rigidBody": "rigidbody_dynamic_1", "halfExtents": {"x": 2, "y": 0.5, "z": 1}}}`. A
  reference names a component of the same project (entityRef or id). To swap a part, add the new
  one, then delete the old one. Something your code moves that other things stand on or bump into
  (a lift, a moving platform, a door) gets a `RigidBodyKinematicPosition`, not a `RigidBodyFixed`:
  a Fixed body is not meant to move, and things resting on a moved one sink in slightly.
- **`r3_editor_delete`** removes components, and only when the user asked for it (it needs the
  `editor:dispose` capability). It is **two calls**, like the editor's delete dialog: first without
  `confirm`, which changes nothing and returns the plan (what goes with the selection, an Entity's
  members, the modes and a `planToken`); then again with `confirm: {"mode": "one" | "with_dependents"
  | "bundle", "planToken": …}`. Ask the user which mode unless they said. If the project changed in
  between, the confirm is refused: plan again.
- **`r3_editor_undo`** takes back YOUR last change (everything one of your calls changed;
  `count` for more, `redo: true` to put it back). Only this key's changes, never the user's. If
  someone changed the same thing after you, it is skipped and the answer says who: tell the user,
  never force your old value over theirs.

**Others change the project too.** Every answer about a project ends with what OTHER people
changed since your last call ("Since your last call: tj set Player.position; Claude (phone)
updated Player.js (v7 → v8)") — the user in a tab, their other agents. Read it before you build
on what you read earlier. **`r3_project_changes`** lists it in full (`since` a seq re-reads a
range). The mark is per key: two conversations on one key share it, so give each Claude its own
key if that matters.

On an R3 environment that does not run the live copy yet, the tool descriptions say so: there the
editor tools' edits are UNSAVED until the user saves in their editor, and the
saved-state tools save at once. The tool descriptions are the authority.

## Reporting what you find

You will sometimes find that R3 itself is wrong, or that it lacks something the user needs. Say
so to R3, with evidence, and tell the user what you sent.

- **A bug in R3** (not in the user's project): check the docs first (`r3_docs_read`) and the
  user's own reports (**`r3_bug_reports_mine`**: when one already covers it, point the user to
  that one instead of filing it twice), then **`r3_bug_report`** with a one-line title, what you did, what you expected and what happened.
  Put what you **measured** in `measurements` (values read back with `r3_component_read` or
  `r3_editor_inspect`, the exact error text) and pass `projectId` when it happens in a project.
  It becomes one of the user's support issues; only the user and R3 see it. Both need
  `support:write`.
- **A missing feature**: read **`r3_roadmap_list`** first (search it; it also shows the user's
  own suggestions still awaiting approval, marked pending, and the ones R3 declined, with R3's
  reason — do not suggest either again). When an item already
  covers it, **`r3_roadmap_upvote`** that item (it adds the user's vote and never removes one).
  Only when nothing does, **`r3_roadmap_suggest`** a new item: say what it lets a person do and
  what you were building when you missed it. A suggestion is held until R3 approves it; it is
  not public before. R3 may decline it instead (with a short reason, shown to the user only). Upvoting needs `roadmap:vote`, suggesting `roadmap:suggest`.
- Both are rate-limited per key. Do not file the same thing twice: when a call's outcome is
  unknown, tell the user instead of sending it again.
- Always tell the user what you filed or suggested, with the link the tool returns.

## Refusals

A refusal is R3 telling you no. It is not a crash, and the same input will be refused again.

- Read the reason, and either fix the input (a wrong entityRef, a property that cannot be set)
  or tell the user what R3 said.
- **No active login on this environment**: R3 acts through the user's own session. Ask the user
  to sign in to R3 in a browser on this environment, then retry. Do not loop on it.
- **Missing capability**: the connection or key does not allow it. Do not look for a way around it.
  - Connected with Sign in with R3: the user disconnects this app under Settings → AI & apps →
    Connected apps, connects again and ticks the right on the Allow page.
  - With an agent key: the user can add the right to this key with Edit rights in R3, under
    Settings → AI & apps → Agent keys (the same key keeps working; a key from before 2026-10-09
    carrying the retired `project:delete` gets the new delete rights there too).
- **401 / not signed in**:
  - Connected with Sign in with R3: the connection was revoked or expired. In Claude Code the user
    runs `/mcp`, picks the R3 server and authenticates again; on claude.ai or in Cowork they
    reconnect R3 on the plugin's Connectors tab.
  - With an agent key: the key is wrong or revoked. The user creates a new one under
    Settings → AI & apps → Agent keys, then sets it on this plugin (`/plugin configure` in Claude
    Code, or the configure command that page shows) and restarts Claude Code or runs `/mcp`.

## Never delete

Do not delete projects or anything in them unless the user asked you to, even when a tool would
allow it. Removing work is the user's decision. If something should go, say so and let the user
decide; `r3_editor_delete` is for when they asked.

The same holds for the account's images, models, audio and Blender models. **`r3_asset_delete`**
(`asset:delete:created` for what this key made, `asset:delete:owned` for any of the user's) is
**two calls**: first without `confirm` — nothing is deleted, and the answer says which of the
user's projects use it and what would go with it (a model takes the images only it used, an
HDR / EXR image its preview image, a Blender model every version and its unused meshes). Tell
the user, and only when they say yes call again with `confirm: true`. An asset another user's
project uses is refused, and no confirm changes that.

To help the user tidy up, **`r3_assets_unused`** (`asset:read`) lists their images, models and
audio that nothing uses (no project, no entity template, no other user's project), biggest
first, with sizes and the total; a model lists the images that would go with it, an HDR image
its preview. It only reads. Blender (.blend) files are never in it: they are the user's to keep,
and you delete one only when the user names it. Show the user the
list; delete only the files they pick, one at a time with `r3_asset_delete` (plan, then
confirm). For a big clean-up, point them at the R3 app's Settings → Storage, which deletes a
whole selection after one confirmation.
