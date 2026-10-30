# CHIQUIBÚ OVERLAY — Spanish children's music videos from a finished Suno MP3

**Overlay version 1.0.0 · applies to base `SYSTEM_PROMPT.md` / `WORKFLOW.json` v2.12.0**

## How this document works

Read `SYSTEM_PROMPT.md` first, in full. Then read this overlay. **Where the two conflict, this overlay wins.** Everything the overlay does not mention keeps its upstream behaviour exactly.

Nothing in the upstream files is edited. All divergence lives in this file and `CHIQUIBU_CONFIG.json`, so this fork stays mergeable with `ai-playbooks/nursery_rhyme_music_video`.

**Trigger phrase:** `Generar video musical Chiquibú`

The overlay changes four things and only four:

| # | Upstream | Chiquibú |
|---|---|---|
| 1 | §5 generates the song in CloneVoice | §5-C imports a finished Suno MP3 |
| 2 | §1 asks for an *idea* + ratio | §1-C asks for the *MP3* + ratio |
| 3 | §11 imports music from the CloneVoice panel | §11-C uploads the MP3 to the VideoExpress library |
| 4 | Language/visual treatment inferred, English default | Spanish (es-MX) metadata, Pixar-style visual spec |

Everything else — the mouth-lock method, the single GO approval, ratio invariance, dynamic `N`, exact endpoint matching, first-take acceptance, Resume — is **unchanged and still binding**.

---

## §1-C. First response: ask exactly two questions

Replaces §1. Ask these together in one concise message and nothing else:

1. **Canción:** the finished Suno MP3 — an absolute file path or a direct audio URL the signed-in browser can fetch.
2. **Formato:** **Horizontal (16:9)** or **Vertical (9:16)**.

Neither may be guessed. If one is already supplied, ask only for the missing one.

Optionally accept, but never ask for: the lyrics as plain text, an explicit song title, and a reusable character description. Infer the project title, protagonist, story arc and export name from the song title, filename and lyrics.

Then present the production brief (§3) and run summary in Spanish and ask for **GO** once, exactly as upstream requires.

---

## §5-C. Import the finished Suno song (replaces §5 entirely)

**Do not open CloneVoice. Do not generate a song. The song is an input to this workflow, never an output of it.**

Chiquibú songs are written and generated in Suno before this pipeline runs. CloneVoice plays no part — skip §5 and the CloneVoice half of §0.4 completely.

1. Resolve the song to a single verified audio payload:
   - **Local file:** read the path given. Require the file to exist.
   - **URL:** from the authenticated browser, `fetch(url, {cache:'no-store'})`, require `response.ok`, read one `ArrayBuffer`/`Blob`.
2. Apply the **same binary-integrity validation as upstream §0.4 step 3**: at least 16 KB, and either an `audio/*` content type or an MP3 signature (`ID3` or MPEG frame sync). This is byte validation, not media preview — **never play the audio.**
3. Build one deterministic `File` named `<sanitized-song-title>.mp3` with type `audio/mpeg`.
4. Read the duration in seconds from the decoded audio metadata. This is `audio_seconds`, used by the §6 scene-count check `3N <= ceil(audio_seconds) <= 10N`. Chiquibú tracks run 2:00–2:30, so expect a viable `N` of roughly 12–50. **Never impose a fixed `N`.**
5. Record in `WORKFLOW_STATE.json`: `source_kind`, `file_name`, `byte_size`, `mime_type`, `audio_seconds`, and `transfer_method` (`suno_local_filepond` or `suno_url_filepond`).
6. Hand that verified `File` to Artistly FilePond using **upstream §0.4 steps 5–6 unchanged** — `DataTransfer`, dispatch `change` with bubbling, require the matching filename plus a completed/success state, and bring the Artistly tab to the foreground if FilePond stalls at "Uploading".

If the fetch or payload validation fails, retry the **audio source** up to three times. Do not spend Artistly storyboard-attempt retries on a missing or corrupt source file. Never upload a zero-byte, HTML or error file, and never delegate the upload to the user.

**Keep this exact `File` object.** §11-C uploads the same bytes to VideoExpress.

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

1. Open the **Import Media / Text to Speech** sidebar tab.
2. Upload the same verified MP3 from §5-C through the sidebar's own file-upload control.

   **The selector for that control is UNVERIFIED in this fork.** On the first run, locate it inside the Import Media sidebar using the §0.2 in-page read approach, then **record the working selector under `dom_api_contract` in `WORKFLOW_STATE.json`** so later runs reuse it. Do not guess a selector, and do not hand the upload to the user.
3. Verify it landed exactly as upstream does: poll `GET /api/library/get_media/4?categoryId=<CATEGORY_ID>&orderBy=id&orderDir=desc` for a result whose `name` matches the uploaded file. Capture its VideoExpress media `id` and `duration` (ms). **That `duration` is the authoritative `audio_end`.** Do not trust the success toast alone.

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
