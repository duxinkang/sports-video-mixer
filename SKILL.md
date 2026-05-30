---
name: sports-video-mixer
description: Use when creating sports short-form videos from local video clips and promo images, including finding source videos, selecting low-caption/player-matched highlights, mixing promo images as small animated cards, burning captions in a target language, adding energetic voiceover, and verifying exported vertical videos.
---

# Sports Video Mixer

Use this skill for sports promo/prediction short videos, especially vertical social videos that combine game clips, promotional posters, captions, and voiceover.

## Workflow

1. Locate assets
   - Search video and poster folders with `rg --files`, `find`, and `ffprobe`.
   - Accept that user-provided folder names may be partial; search nearby directories before asking.
   - List video duration, resolution, frame rate, and available audio tracks.
   - Generate contact sheets from representative frames so clips can be matched to players, teams, and themes.
   - For large social-video folders, generate a full candidate contact sheet before editing. Use it to prefer clips with less burned-in text and cleaner action frames instead of trying to hide bad source text later.

2. Plan the edits
   - Group source clips by team/player/action, such as trophy celebration, star player close-up, crowd reaction, key shot, block, or dunk.
   - Make each output video a clear angle: team perspective, opponent perspective, matchup prediction, betting/promo angle, or highlight recap.
   - Match captions and voiceover to the on-screen subject. Do not mention a player while showing unrelated footage unless the line is explicitly about the matchup.
   - If the user asks for "low captions" or "字幕少", treat it as an edit-quality requirement:
     - Choose clips with minimal native TikTok/Instagram text when possible.
     - Use slight zoom/reframe or upward crop to reduce lower-third source captions.
     - Do not cover half the screen with dark overlays; large black masks make the video look broken and should be avoided.

3. Mix the videos
   - Prefer `ffmpeg` for deterministic edits.
   - Normalize social clips to `1080x1920`, `30fps`, H.264 video, AAC audio.
   - Use `scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,setsar=1,fps=30`.
   - When source videos have bottom captions, prefer a gentle upward reframing such as `scale=1350:2400:force_original_aspect_ratio=increase,crop=1080:1920:(iw-ow)/2:0,setsar=1,fps=30` instead of dark masking.
   - Use `trim`, `atrim`, `setpts`, `asetpts`, and `concat` for scene order.
   - Keep original crowd/audio as atmosphere unless the user asks for silence.
   - For 10 second shorts, keep the edit rhythm readable:
     - Use at most 3 source videos per finished short unless the user explicitly asks for a rapid montage.
     - Hold each source clip long enough for the action to land, usually 2.8-4.5 seconds in a 10 second edit.
     - Do not switch layout modes repeatedly inside the same short. Pick one visual structure per output, such as full-screen cuts, top/bottom split, side-by-side, or one consistent picture-in-picture style.
     - Avoid changing split-screen orientation every 1-2 seconds; fast layout changes make the video feel choppy and interrupt the highlight payoff.
     - Let the strongest source clip run the longest, then use the second and third clips as setup/reaction/closing support.
   - Mix promo posters into the action as small animated cards, not as a full-screen end card unless the user explicitly asks for one.
   - Good promo-card patterns:
     - slide in from the right for 1.0-1.6 seconds near the lower third
     - slide in from the left at mid-screen during a dribble or reset moment
     - pop up briefly near a dead corner after a dunk or replay cut
   - Keep promo cards small enough that the main sports action stays readable, usually 220-340 px wide in a 1080x1920 video.
   - Reuse promo cards 1-2 times across a 10 second edit, or 2-3 times across a 15 second edit, instead of putting one large poster only at the ending.
   - Time promo cards to moments with lower action intensity so they do not hide the dunk, shot release, block, or celebration peak.

4. Add captions
   - If `drawtext` is unavailable, generate transparent PNG caption cards and overlay them.
   - Use large high-contrast captions with a dark translucent background.
   - Keep captions short enough for mobile reading, generally 1-2 lines.
   - Put new captions near the top safe area when source videos already have lower-third captions.
   - Keep custom caption cards compact; they should label the moment, not compete with the original social-video text.
   - For Filipino/Tagalog sports captions, use energetic, natural phrasing:
     - `Prediksyon sa Finals`
     - `tempo kontra haba`
     - `pinakamalaking x-factor`
     - `kaya itong umabot sa Game Seven`
     - `defensive rebounds ang magpapasya`

5. Add voiceover
   - Use a real target-language TTS voice when possible. For Filipino, prefer `edge-tts` voices:
     - `fil-PH-AngeloNeural` for energetic male sports commentary
     - `fil-PH-BlessicaNeural` for energetic female commentary
   - If `edge-tts` is not installed, create a local virtual environment and install it there rather than modifying system Python.
   - Make voiceover scripts punchy and aligned to the clips. Avoid reading the exact captions word for word.
   - Mix voiceover over original audio by lowering original audio to about `0.25-0.35` volume and raising voiceover to about `1.4-1.8`.
   - Add a small delay, such as `adelay=350|350`, so the video breathes before the voice starts.

6. Verify
   - Use `ffprobe` to confirm each output has video, AAC audio, expected dimensions, and expected duration.
   - Extract frames from early, middle, and ending sections.
   - Create a contact sheet to confirm:
     - captions are in the requested language
     - text is readable and not cut off
     - promo images appear at the intended time
     - promo images are integrated as small cards when the user asked for a mixed edit
     - no broad dark mask covers a large portion of the frame
     - original source captions are reduced by selection/reframing rather than ugly blackout overlays
     - player/team captions match the visible footage
     - each 10 second output uses no more than 3 source videos unless rapid montage was requested
     - each output uses one consistent split-screen or picture-in-picture style instead of changing layouts every 1-2 seconds
     - key highlight actions are not cut away before the payoff
   - For voiceover, confirm each final video has an audio stream and that voiceover files were generated.

## Useful Commands

Probe videos:

```bash
for f in /path/to/videos/*.{mp4,mov,mkv}; do
  echo "$f"
  ffprobe -v error -show_entries format=duration:stream=codec_type,width,height,avg_frame_rate -of default=nw=1 "$f"
done
```

Create a candidate contact sheet for a 100-video folder:

```bash
mkdir -p review_candidates
n=0
find /path/to/videos -maxdepth 1 -type f -name '*.mp4' | sort | while IFS= read -r f; do
  n=$((n+1))
  b=$(printf '%03d' "$n")
  ffmpeg -y -hide_banner -loglevel error -ss 1 -i "$f" -frames:v 1 -vf "scale=180:320" "review_candidates/${b}.jpg" || true
  printf '%03d\t%s\n' "$n" "$(basename "$f")" >> "review_candidates/index.tsv"
done
ffmpeg -y -hide_banner -loglevel error -pattern_type glob -i "review_candidates/*.jpg" -vf "tile=10x10" "review_candidates/all_100_contact.jpg"
```

Extract review frames:

```bash
mkdir -p review_frames
for f in outputs/*.mp4; do
  b=$(basename "$f" .mp4)
  ffmpeg -y -hide_banner -loglevel error -ss 00:00:02 -i "$f" -frames:v 1 "review_frames/${b}_02.jpg"
  ffmpeg -y -hide_banner -loglevel error -ss 00:00:12 -i "$f" -frames:v 1 "review_frames/${b}_12.jpg"
  ffmpeg -y -hide_banner -loglevel error -sseof -3 -i "$f" -frames:v 1 "review_frames/${b}_end.jpg"
done
```

Generate Filipino TTS:

```bash
python3 -m venv .venv-edge-tts
.venv-edge-tts/bin/python -m pip install edge-tts
.venv-edge-tts/bin/edge-tts \
  --voice fil-PH-AngeloNeural \
  --rate "+24%" \
  --volume "+25%" \
  --pitch "+3Hz" \
  --text "Ito ang malaking matchup: tempo kontra haba!" \
  --write-media voiceover.mp3
```

Mix voiceover with original audio:

```bash
duration=$(ffprobe -v error -show_entries format=duration -of csv=p=0 input.mp4)
ffmpeg -hide_banner -y -i input.mp4 -i voiceover.mp3 \
  -filter_complex "[0:a]volume=0.28[a0];[1:a]adelay=350|350,apad,atrim=0:${duration},volume=1.65[a1];[a0][a1]amix=inputs=2:duration=first:normalize=0,alimiter=limit=0.95[a]" \
  -map 0:v:0 -map "[a]" -c:v copy -c:a aac -b:a 192k -movflags +faststart output_voiceover.mp4
```

Overlay small promo cards during an edit:

```bash
ffmpeg -y -i base.mp4 \
  -framerate 30 -loop 1 -t 10 -i promo_a.png \
  -framerate 30 -loop 1 -t 10 -i promo_b.png \
  -filter_complex "[1:v]scale=250:-1,format=rgba,colorchannelmixer=aa=0.93[p1];[2:v]scale=300:-1,format=rgba,colorchannelmixer=aa=0.94[p2];[0:v][p1]overlay=x='if(lt(t,1.05),W-(t-0.75)*760,W-w-32)':y=1050:enable='between(t,0.75,2.15)'[v1];[v1][p2]overlay=x='if(lt(t,3.35),-w+(t-3.05)*760,34)':y=980:enable='between(t,3.05,4.75)'[v]" \
  -map "[v]" -map 0:a -t 10 -c:v libx264 -c:a aac output.mp4
```

## Output Naming

- Put finished videos in an `outputs/` subfolder.
- Keep non-voiceover and voiceover versions separate:
  - `01_team_prediction.mp4`
  - `01_team_prediction_voiceover.mp4`
- Keep generated caption cards and voiceover audio in subfolders so the user can reuse or revise them.
