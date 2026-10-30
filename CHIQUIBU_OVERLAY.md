# CHIQUIBÚ OVERLAY — Spanish children's music videos from a finished Suno MP3

**Overlay version 1.0.0 · applies to base `SYSTEM_PROMPT.md` / `WORKFLOW.json` v2.12.0**

## How this document works

Read `SYSTEM_PROMPT.md` first, in full. Then read this overlay. **Where the two conflict, this overlay wins.** Everything the overlay does not mention keeps its upstream behaviour exactly.

Nothing in the upstream files is edited. All divergence lives in this file and `CHIQUIBU_CONFIG.json`, so this fork stays mergeable with `ai-playbooks/nursery_rhyme_music_video`.

**Trigger phrase:** `Generar video musical Chiquibú`

The overlay changes four things and only four:

| # | Upstream | Chiquibú |
|---|---|---|
| 1 | §5 generates the song in CloneVoice | §5-C takes a finished Suno MP3 |
| 2 | §1 asks for an *idea* + ratio | §1-C asks for the *song title* + ratio |
| 3 | §11 imports music from the CloneVoice panel | §11-C puts the MP3 in the VideoExpress library |
| 4 | Language/visual treatment inferred, English default | Spanish (es-MX) metadata, Pixar-style visual spec |
| 5 | The agent never hands browser work to the user | §U the **user performs the two audio uploads** |

Everything else — the mouth-lock method, the single GO approval, ratio invariance, dynamic `N`, exact endpoint matching, first-take acceptance, Resume — is **unchanged and still binding**.

---

## §U. Manual audio upload — the one place the user is in the loop

**This section overrides upstream's prohibition on delegating browser work to the user. It does so for the two audio uploads and nothing else.**

Upstream forbids handing browser actions back to the user, and its v2.7.0 changelog removed user-delegated steps deliberately, to stop lighter models from stalling. That prohibition **remains in force for everything except the two uploads named here.** The agent never hands back clicking, navigation, generation, timeline assembly, saving or export. If the agent finds itself about to ask the user to do anything other than these two uploads, that is a bug — it should act instead.

Why uploads are the exception: the audio file lives on the user's computer, and no route gets it into the page automatically. A web page cannot read a local path, and URL delivery is closed (see §5-C). The user's own file dialog is the only reliable door.

### The two upload points

1. **Artistly FilePond** — immediately after GO, at the start of §6.
2. **VideoExpress media library** — in §11-C, after clip generation, typically 30–40 minutes later.

### The protocol at each point

1. Bring the relevant tab to the foreground and open the upload control so the user only has to pick the file.
2. Ask in one short line, in Spanish, naming the file expected. Then **wait.** Do not poll aimlessly, do not proceed on an assumption, and do not treat the user's "listo" as evidence on its own.
3. **Verify independently, with the same evidence the agent would demand of its own upload:**
   - *Artistly:* FilePond shows the matching filename **and** a completed/success state. A selected filename without upload completion is not success.
   - *VideoExpress:* `GET /api/library/get_media/4?categoryId=<CATEGORY_ID>&orderBy=id&orderDir=desc` returns a result whose `name` matches, and its `duration` (ms) is captured as the authoritative `audio_end`. **A success toast is not evidence** — a stale video-completion toast can read "success".
4. Record `uploaded_by: "user"`, the filename, byte size where visible, and the verified state in `WORKFLOW_STATE.json`.
5. If verification fails, say exactly what is missing and ask once more. Never continue past an unverified upload: in VideoExpress, a missing track surfaces as a finished timeline with no audio, forty minutes of credits later.

While waiting, record any working upload-control selector observed. That is what eventually makes this step automatic.

---

## §1-C. First response: ask exactly two questions

Replaces §1. Ask these together in one concise message and nothing else:

1. **Canción:** the song title, with the finished Suno MP3 ready to upload — the user will be asked to upload it themselves at the two points in §U.
2. **Formato:** **Horizontal (16:9)** or **Vertical (9:16)**.

Neither may be guessed. If one is already supplied, ask only for the missing one. In `manual_upload` mode no file path and no URL is needed; the title may also be read from the uploaded filename once the first upload completes.

Optionally accept, but never ask for: the lyrics as plain text, an explicit song title, and a reusable character description. Infer the project title, protagonist, story arc and export name from the song title, filename and lyrics.

Then present the production brief (§3) and run summary in Spanish and ask for **GO** once, exactly as upstream requires.

---

## §5-C. The finished Suno song (replaces §5 entirely)

**Do not open CloneVoice. Do not generate a song. The song is an input to this workflow, never an output of it.**

Chiquibú songs are written and generated in Suno before this pipeline runs. CloneVoice plays no part — skip §5 and the CloneVoice half of §0.4 completely.

**Default delivery is `manual_upload` (§U):** the user supplies the MP3 through Artistly's own file dialog. The agent does not need the bytes, a path, or a URL — it needs the upload to be **verified**.

Still binding, whoever performs the upload:

- **Integrity, as upstream §0.4 step 3:** at least 16 KB, and an `audio/*` type or an MP3 signature (`ID3` or MPEG frame sync). Byte validation only — **never play the audio.**
- **Duration:** read `audio_seconds` once the file is in. It drives the §6 scene-count check `3N <= ceil(audio_seconds) <= 10N`. Chiquibú tracks run 2:00–2:30, giving a viable `N` of roughly 16–50. **Never impose a fixed `N`.**
- **Record** in `WORKFLOW_STATE.json`: `file_name`, `byte_size`, `mime_type`, `audio_seconds`, `transfer_method`, `uploaded_by`.
- **Never** accept a zero-byte, HTML or error upload, and never continue past one that has not shown a completed state.

**Preferred file: Suno's own MP3 export.** Verified 2026-10-29 — valid ID3 header, embedded cover art, ~184 kbps, and no generation loss from re-encoding. A WAV of the same 2:30 track is 28.8 MB against 3.45 MB; convert before the run rather than uploading the WAV, whose acceptance by FilePond is unverified.

### Why URL delivery is closed

Both obvious URL routes were tested on 2026-10-29 and both fail structurally. **Do not re-test them mid-run.**

- **Suno share links** (`suno.com/s/...`) expose no audio URL at all; playback runs from a blob assembled in the browser. The underlying CloudFront object *is* fetchable cross-origin from `app.artistly.ai`, but its bytes carry no audio container signature — no `ID3`, no MPEG frame sync, no MP4 `ftyp`, served as `application/octet-stream` — consistent with an obfuscated or encrypted stream. It fails integrity validation, correctly. `cdn1.suno.ai`, `cdn-o.suno.com`, `cdn1.suno.com` and `audiopipe.suno.ai` all return 403/404.
- **Google Drive links** fail CORS. Google sends no `Access-Control-Allow-Origin` on file downloads, so no third-party page can fetch them at any sharing setting. The Drive v3 API allows CORS but needs an OAuth token, and this workflow never handles credentials.

URL delivery remains valid **only** for a static host configured with permissive CORS that returns raw audio bytes. If such a URL is supplied, use upstream §0.4 unchanged: fetch, validate, build a deterministic `File` of type `audio/mpeg`, inject via `DataTransfer`, and require FilePond to confirm.

---

## §6-C. Storyboard notes (amends §6, does not replace it)

Run §6 exactly as written, with three Chiquibú notes:

- The **Nursery Rhymes** agent takes only an audio upload and derives character identity from the sung lyrics. Spanish lyrics therefore carry identity on their own — no prompt field is involved.
- **Its behaviour on Spanish-language audio is unverified in this fork.** Treat the first Spanish run as a test: if the batch fails structurally, log it to `error_history` with the exact symptom before retrying, so the Spanish-audio question gets answered with evidence rather than a guess.
- If the **Music Storyboard** fallback is reached, its 150-character Storyboard Style prompt is written in **English** (prompt fields stay English per `CHIQUIBU_CONFIG.json` → `language.scene_prompt_language`) and must carry, in this priority order when space runs out:
  1. the mouth-pose clause, as the first clause — `sealed closed-lip smile; still jaw and lower face`;
  2. the protagonist identity, from the character sheet or the lyrics;
  3. `3D Pixar-style, warm, soft light`;
  4. the ratio.

  Drop the no-text clause before the mouth-pose clause. Mouth pose affects every frame; a stray sign does not.

---

## §6.A-C. Prompt book additions (amends §6.A)

Scene prompts stay in **English** and the mouth-locked architecture from `PROMPT_GUIDE.md` remains **binding and unmodified** — `mouth_lock_check` still blocks storage of any scene that fails it.

Add the Chiquibú visual spec to each scene prompt **additively**, never in place of a mouth-lock clause:

- `3D animated Pixar-style`
- `warm color palette`
- `soft lighting`
- `no text, no letters, no numbers, no watermarks, no signage anywhere in frame`

If a prompt field's character limit cannot hold both the mouth-lock clauses and the visual spec, the mouth-lock clauses win.

---

## §11-C. Import and place the Suno music (replaces §11 steps 1–6)

The track never existed in CloneVoice, so **the "Import from CloneVoice.ai" panel will not list it.** Upload the MP3 directly instead.

1. Open the **Import Media / Text to Speech** sidebar tab and bring the control into view.
2. **Second upload point (§U).** Ask the user to upload the same MP3 through the sidebar's own file-upload control, and wait.

   The selector for that control is **UNVERIFIED** in this fork — which is precisely why the user drives it. Record any working upload selector observed under `dom_api_contract` in `WORKFLOW_STATE.json`; that recording is what later makes this step automatic.
3. Verify it landed exactly as upstream does: poll `GET /api/library/get_media/4?categoryId=<CATEGORY_ID>&orderBy=id&orderDir=desc` for a result whose `name` matches the uploaded file. Capture its VideoExpress media `id` and `duration` (ms). **That `duration` is the authoritative `audio_end`.** Do not trust the success toast alone — a stale video-completion toast can read "success".

**This is the costliest step to get wrong.** By the time §11-C runs, every clip has been generated and paid for. Continuing past an unverified upload yields a finished timeline with no audio.

**Steps 7–8 of §11 are unchanged:** drag the audio `.library-item` onto track-row index 1 with its left edge at 0, and require exactly one audio brick on track 2. §12's exact endpoint matching is unchanged.

---

## Naming and language

- **Export name:** `<Song Title> (Canciones Infantiles) - Chiquibú`
- **YouTube title:** `<Song Title> (Canciones Infantiles)`
- **VideoExpress project name:** `chiquibu-<song-slug>-<ratio>` — kept short for findability, no suffix.
- **Spanish (es-MX)** for the project title, export name, progress lines and the final report.
- **English** for scene prompts, and for all DOM selectors, API paths and `WORKFLOW_STATE.json` field names, which stay exactly as upstream defines them.

---

## What this overlay deliberately does not change

The mouth-locked prompt guide and its blocking check · ratio as a project-wide invariant · dynamic scene count `N` · exact audio/video endpoint matching · first-take acceptance and the no-preview rule · the single GO approval before credits are spent · Resume and checkpointing · every safety rule and the in-browser-only execution contract.
