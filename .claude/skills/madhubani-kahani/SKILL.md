---
name: madhubani-kahani
description: Turn an attached Hindi story voiceover MP3 into a deterministic, code-drawn Madhubani folk-art animation with word-synchronised motion and Hindi speech bubbles. Use when the user invokes /madhubani-kahani with an audio file.
---

# Madhubani Kahani

Create a Panchatantra-style story film that looks like a living Madhubani painting. The attached Hindi voiceover is the source of truth for narration, scene timing, and dialogue.

## Non-negotiable visual style

- Draw every visual element programmatically in HTML Canvas. Render frames in headless Chrome and encode them with FFmpeg.
- Do not use stock footage, stock images, image generators, downloaded illustrations, or externally sourced visual assets.
- Keep the art deterministic: same input, configuration, and code must produce the same frames.
- Use bold black double outlines, flat folk-art colours (red, yellow, indigo, green, plus black and natural paper tones), and dense line-based patterns inside every filled shape.
- Enclose every scene in a thick decorated Madhubani border. Use repeating geometric, floral, fish, leaf, and sun motifs.
- Water uses fish-scale patterns and repeated wave lines. Avoid gradients, photorealism, 3D rendering, and modern clip-art.
- Characters must remain visually consistent across scenes. Define reusable character drawing functions and a fixed character sheet before scene production.
- Use a 1920×1080 landscape composition. Keep important characters, text, and speech bubbles inside title-safe margins.

## Workflow

### 1. Inspect the input and prepare the narration

1. Find the user-attached audio file. Accept MP3 or WAV. Do not substitute a different file.
2. Inspect duration, sample rate, channels, and whether the file can be decoded. Preserve the original audio for the final master.
3. Transcribe the complete narration in Devanagari Hindi, preserving the spoken wording. Do not silently rewrite, translate, summarise, or invent missing words.
4. Prefer an available local Hindi-capable speech recogniser. If none is installed or usable, ask the user for a Devanagari transcript rather than pretending the audio has been transcribed. Do not download model weights or send private audio to an external service without the user's explicit approval.
5. Save the transcript as UTF-8. Split it into spoken lines at natural sentence / breath boundaries. Keep punctuation and speaker attribution where clear.
6. Verify the transcript against the audio. Mark genuinely unclear spans as [अस्पष्ट] and ask for clarification if they materially affect the story. Never fabricate dialogue.

### 2. Force-align every word

1. Use torchaudio MMS_FA forced alignment for Hindi. Confirm that the installed torchaudio build exposes a Hindi-compatible MMS_FA bundle/token set before starting.
2. Supply the verified Devanagari transcript to the aligner. Normalize text only as required by the aligner's documented tokenizer, while retaining a mapping back to the original Devanagari words.
3. Produce a timestamp for every spoken word, with start and end times in seconds. Store the results in a machine-readable file such as word_timestamps.json, including original word, normalized token(s), start, end, and confidence/validity where available.
4. Validate that timestamps are monotonic, within the audio duration, and do not overlap impossibly. Listen-check suspicious gaps and boundaries.
5. Never guess, interpolate, or hand-assign word timings. If MMS_FA cannot align Hindi, the model/tokenizer is unavailable, or alignment fails validation, stop and explain the blocker. Ask for a compatible alignment setup or a corrected transcript; do not proceed with invented timings.
6. Line start/end times must be derived from the aligned words in that line, not estimated independently.

### 3. Plan one painted scene per spoken line

Create a scene_plan.json with exactly one scene for each narration line. Each scene must specify:

- line ID and exact Devanagari narration;
- aligned line start/end;
- location/background;
- characters present and their consistent character-sheet IDs;
- the single important action that communicates the line;
- props and patterned environmental details;
- speech-bubble text and the first spoken word that triggers it;
- word IDs that trigger any small animation;
- transition to the next scene.

Keep the story faithful to the audio. Use a Panchatantra-like visual storytelling tone, but do not add new plot events or dialogue that the narration does not support. For each line, prioritise one readable action rather than crowding the frame.

### 4. Build the artwork in code

1. Create reusable Canvas drawing primitives for double-outline paths, patterned fills, border tiles, speech bubbles, water, trees, foliage, buildings, sun, clouds, and character parts.
2. Implement pattern fills with Canvas paths, clipping, and repeated line motifs. Every closed coloured shape must have an interior line pattern; do not leave large plain colour regions.
3. Build each character from reusable code and fixed proportions. Keep face, clothing, markings, and colours consistent in every scene.
4. Use only the defined palette: folk red, turmeric yellow, indigo, leaf green, black, and paper/off-white. Prefer solid fills and visible hand-drawn line rhythm.
5. Add a thick decorated frame on every scene. Ensure patterns and borders do not obscure faces, action, narration, or bubbles.
6. Use Devanagari-capable fonts available locally. Check that Hindi glyphs render correctly in Chrome; do not allow missing-glyph boxes.

### 5. Animate on spoken words

- Drive all motion from the forced-alignment word timestamps.
- The monkey jumps on the relevant spoken action word; the crocodile's jaw snaps on its relevant word; water ripples on relevant water/action words. Choose triggers from the actual transcript and scene plan, not from guessed time offsets.
- Keep movements small, legible, and folk-art-like: short pose changes, simple translations, jaw pivots, blinking, tail/ear movement, and repeated ripple strokes.
- A motion event must reference a real aligned word ID and use that word's start time as its trigger. If no suitable spoken word exists, leave that motion out.
- Speech bubbles pop in at the start time of the first spoken word of that line. Show only the line's actual spoken dialogue/narration, in Devanagari. Use a short pop animation, then keep the bubble readable for the line.
- Do not display an entire line before it is spoken. Avoid covering the speaking character's face.

### 6. Preview before rendering the film

1. Render representative still frames for every scene (or a clearly labelled thumbnail for each scene) using the same Canvas renderer.
2. Assemble and show a contact sheet with scene numbers, line IDs, and short Devanagari captions.
3. Stop and ask the user to approve the contact sheet before rendering the full MP4. Incorporate requested visual corrections and regenerate the contact sheet if needed.
4. Do not skip this approval step, even on repeat runs.

### 7. Render and export

1. Render frame-by-frame at 1920×1080 using HTML Canvas in headless Chrome. Use a deterministic frame clock; do not rely on real-time playback or browser timers for frame state.
2. Pipe or save frames in order and encode with FFmpeg. Use a fixed frame rate (default 30 fps) and stable scene transitions.
3. Mux the original, unmodified voiceover audio into the 1080p master. Do not replace it with generated speech or alter its content. Keep the full audio duration; ensure the final video does not truncate narration.
4. Export a 1080p MP4 master, for example madhubani-kahani-1080p.mp4.
5. Create a 720p WhatsApp copy, for example madhubani-kahani-720p-whatsapp.mp4, with a target file size below 20 MB. Calculate the available bitrate from the actual duration, audio bitrate, and container overhead; use FFmpeg two-pass encoding where appropriate. Verify the final file size. If under 20 MB cannot be achieved without unacceptable quality or the available tools cannot do it, report the actual size and limitation rather than claiming success.
6. Verify both MP4s with FFprobe: dimensions, duration, video/audio streams, and decodability. Confirm that the original voiceover is present and synchronised.

## File safety and overwrite rules

- Use a dedicated output folder for each run, such as output/madhubani-kahani/<run-id>/.
- Before creating or replacing any output, inspect whether the target path already exists.
- Never overwrite any existing file without asking the user first and receiving explicit approval. This includes contact sheets, transcripts, scene plans, timestamp JSON, rendered frames, and MP4s.
- Prefer a new run folder when prior outputs exist. If a requested fixed filename already exists, ask before replacing it.
- Do not delete or clean up user files unless the user explicitly asks.

## Suggested project outputs

- transcript.hi.txt — verified Devanagari transcript
- word_timestamps.json — MMS_FA word-level alignment
- scene_plan.json — one scene per spoken line
- characters/ — reusable character definitions, if separated
- contact-sheet.png — preview for approval
- madhubani-kahani-1080p.mp4 — 1920×1080 master with original audio
- madhubani-kahani-720p-whatsapp.mp4 — 1280×720 copy targeting under 20 MB

## Completion report

After rendering, report the transcript/alignment status, scene count, preview approval, output paths, actual dimensions/durations, and verified file sizes. Clearly disclose any step that could not be completed. Never claim a render, alignment, preview, or size target succeeded unless it was actually verified.
