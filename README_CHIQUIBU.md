# Chiquibú fork — how to run it

This is a fork of [`ai-playbooks/nursery_rhyme_music_video`](https://github.com/ai-playbooks/nursery_rhyme_music_video), adapted for **Chiquibú**: Spanish-language children's music videos built from songs that already exist as finished Suno MP3s.

Upstream's `README.md` still describes the original three-app pipeline. **This file describes how the fork actually runs.** Read this one.

## What's different

Upstream turns *one idea* into a video, generating the song itself in CloneVoice. Chiquibú songs are written and generated in Suno first, so the fork starts one step later:

```
upstream:  idea → CloneVoice (song) → Artistly (storyboard) → VideoExpress (clips) → export
Chiquibú:  Suno MP3 ──────────────▶ Artistly (storyboard) → VideoExpress (clips) → export
```

CloneVoice is not used at all. You need accounts and credits on **Artistly.ai** and **VideoExpress.ai** only.

## The overlay approach

No upstream file is edited. Everything specific to Chiquibú lives in two new files:

| File | What it is |
|---|---|
| `CHIQUIBU_OVERLAY.md` | The prose contract. Read after `SYSTEM_PROMPT.md`; overrides it where they conflict. |
| `CHIQUIBU_CONFIG.json` | The machine-readable mirror of the same overrides. |

So `git pull upstream main` stays clean — upstream's improvements to the mouth-lock method, the timeline logic or the recovery gates arrive without conflicts, and the overlay keeps applying on top.

To add the upstream remote once:

```bash
git remote add upstream https://github.com/ai-playbooks/nursery_rhyme_music_video.git
git fetch upstream
git merge upstream/main
```

## Running it

1. Sign in to **Artistly.ai** and **VideoExpress.ai** in the browser the agent controls.
2. Export the finished song from Suno as an MP3. Have the file path (or a direct audio URL) ready.
3. Start the agent with the contents of **`SYSTEM_PROMPT.md` followed by `CHIQUIBU_OVERLAY.md`**, or with the trigger phrase **"Generar video musical Chiquibú"**.
4. Answer the two questions: the **MP3**, and **Horizontal (16:9)** or **Vertical (9:16)**.
5. Optionally paste the lyrics as plain text — not needed for the default storyboard agent, but it makes the fallback agent's character prompt accurate.
6. Review the brief and reply **GO**. That remains the only approval before credits are spent.
7. Say **`Resume`** if a run stops early.

Upstream's model guidance still applies: this is a 100+ tool-call run, so use a model with stamina rather than raw reasoning.

## Chiquibú defaults baked in

- **Language:** Spanish (es-MX) for the project title, export name, progress lines and final report.
  Scene prompts stay in **English** — the mouth-lock method in `PROMPT_GUIDE.md` is specified and validated in English, and the blocking `mouth_lock_check` matches English clauses. Translating them would silently break the check.
- **Visual spec:** 3D animated Pixar-style, warm colors, soft lighting, explicit no-text-in-frame.
- **Naming:** export `<Song Title> (Canciones Infantiles) - Chiquibú`, YouTube title `<Song Title> (Canciones Infantiles)`, project `chiquibu-<song-slug>-<ratio>`.

## Two things that are unverified

Both are flagged in the overlay and should be resolved on the first real run rather than guessed at now:

1. **The VideoExpress upload selector.** Upstream imports music through an "Import from CloneVoice.ai" panel, which can't list a track that was never in CloneVoice. The overlay replaces it with a direct library upload, but the DOM selector for that control hasn't been confirmed. The agent is told to find it at runtime and record it in `WORKFLOW_STATE.json` so later runs reuse it.
2. **Whether the Artistly "Nursery Rhymes" agent handles Spanish audio well.** It takes only an audio upload and derives the character from the sung lyrics — no prompt field. If Spanish lyrics trip it up, the Music Storyboard fallback accepts a typed character-lock prompt instead; flip `artistly_agent_policy.default` to `music_storyboard_first` in `CHIQUIBU_CONFIG.json` once there's evidence. That fallback is also the better path for keeping a recurring character consistent across releases.

## What is deliberately untouched

The mouth-locked prompt guide and its blocking check, ratio invariance, dynamic scene count, exact audio/video endpoint matching, first-take acceptance, the single GO approval, Resume and checkpointing, and every safety rule.
