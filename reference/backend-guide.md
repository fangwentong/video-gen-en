# Backend Selection and Reference Image Strategy

## Table of Contents

- Four Backend Capability Comparison
- Backend Selection Decision Tree
- Seedance Smart Multi-Shot Mode
- Auto Selection Logic
- Character Reference Image Two Paths
- **Path A: Kling Omni (Recommended)**
- Path B: Kling + Gemini First Frame
- Gemini Prompt Notes

---

## Four Backend Capability Comparison

| Capability | Kling | Kling Omni | **Seedance 2** | **Veo3** |
|------|-------|------------|----------------|----------|
| **Backend Name** | `kling` | `kling-omni` | `seedance` | `veo3` |
| **Provider** | official/fal | official/fal | piapi | **compass** |
| **Text-to-Video** | 3-15s | 3-15s | **4-15s (any integer)** | **4/6/8s** |
| **Image-to-Video** | First frame image (precise control) | Use image_list instead | Storyboard + reference image | **First frame image** |
| **image_list Multi-Reference** | -- | `<<<image_1>>>` reference | **`@image1` reference (max 12 images)** | -- |
| **Smart Multi-Shot** | multi-shot parameter control | multi-shot parameter control | **Time-segmented prompt auto-trigger** | -- |
| **First/Last Frame Control** | `--image` + `--tail-image` | -- | **Supported (mode: first_last_frames)** | `--image` (first frame) |
| **Audio Reference** | -- | -- | **✓ audio_urls (mp3/wav ≤15s)** | -- |
| **Audio-Video Simultaneous Output** | `--audio` | `--audio` | **✓ Auto-generate audio** | **✓ Auto-generate audio** |
| **Max Resolution** | 1080p | 1080p | **720p** ⚠️ | **720p** |
| **Aspect Ratio** | 16:9/9:16/1:1 | 16:9/9:16/1:1 | **16:9/9:16/4:3/3:4/1:1/21:9/auto** | 16:9/9:16 |
| **Best Use Case** | First frame precise control, scene consistency | Character consistency, multi-character | **Fiction/short drama, smart multi-shot, MV** | **High-quality realistic short films** |

**Key Differences**:
- Kling `--image` is a **first frame image** (video starts from this image)
- Kling Omni `--image-list` is a **reference image** (keeps character consistency)
- **Seedance 2 time segmentation = auto multi-shot**: No extra parameters needed, time-segmented prompt auto-triggers smart multi-shot
- **Seedance 2 storyboard is reference**: Not first frame precise control, but visual style reference
- **Seedance 2 supports first/last frame control**: Use `--mode first_last_frames` + `--image-list`
- **Seedance 2 duration 4-15s** (any integer), **Veo3 duration only supports 4/6/8s** (enum values)

---

## Backend Selection Decision Tree

**Scenario-Driven Selection**:

| Scenario | Priority Backend | Fallback Backend | Reason |
|-----|---------|---------|------|
| **Fiction/Short Drama** | **Seedance** | Kling-Omni | Smart multi-shot + multi-reference, character consistency |
| **Commercial (No Real Footage)** | **Seedance** | Kling-Omni | Long shots + smart multi-shot |
| **Commercial (With Real Footage)** | Kling-3.0 | — | First frame precise control, real footage |
| **MV Short Film** | **Seedance** | Kling-Omni | Long shots + music-driven |
| **Vlog/Documentary Style** | Kling-3.0 | Veo3 | First frame precise control, avoid Seedance |
| **High-Quality Realistic Short** | Kling-3.0 | Veo3 | Veo3 as fallback only, 4/6/8s |
| **Ultra-Short Shot (≤ 4s)** | Kling | Veo3 | Kling minimum 3s, Veo3 minimum 4s as fallback |

**First Frame Control Capability Comparison**:

| Backend | First Frame Control | Description |
|------|---------|------|
| **Kling-3.0** | ✅ `--image` | Video starts from this image |
| **Veo3** | ✅ `--image` | First frame precise control |
| **Seedance** | ❌ Reference image | Storyboard is visual style reference, not first frame |
| **Kling-Omni** | ❌ Reference image | Only reference2video, no img2video |

**Core Principles**:
1. **Need smart multi-shot → Seedance** (time segmentation auto-triggers multi-shot)
2. **Need first frame control → Kling/Vidu** (only these two support it)
3. **Seedance fails → Downgrade to Kling-Omni** (lose smart multi-shot, keep character consistency)

### Quick Reference by Scenario

| Scenario | Backend | Key Parameters |
|------|------|---------|
| **Fiction/Short Drama** | **Seedance** | `--backend seedance --image-list frame.png ref.jpg` |
| **Commercial (No Real Footage)** | **Seedance** | Time-segmented prompt + storyboard |
| **Commercial (With Real Footage)** | Kling-3.0 | `--image first_frame.png` |
| **MV Short Film** | **Seedance** | Time-segmented prompt + storyboard |
| **Vlog/Documentary Style** | Kling-3.0 | `--image first_frame.png` |
| Need first/last frame animation | Kling | `--image first.png --tail-image last.png` |
| High-quality realistic short | **Veo3** | `--backend veo3 --duration 8` |
| Simple non-human scene / Quick prototype | Kling (default) or vidu | No special parameters needed |

---

## Seedance 2 Smart Multi-Shot Mode

**Key Feature**: Time-segmented prompt auto-triggers multi-shot smart cutting, no extra parameters needed.

### API Parameters

| Parameter | Value | Description |
|------|-----|------|
| `model` | `"seedance"` | Fixed value |
| `task_type` | `"seedance-2-fast"` / `"seedance-2"` | Fast / High quality |
| `prompt` | Text description | Supports `@imageN` / `@videoN` / `@audioN` references, supports time segmentation |
| `mode` | `text_to_video` / `first_last_frames` / `omni_reference` | Generation mode (required) |
| `duration` | **4-15 (any integer)** | Seconds (range 4-15) |
| `aspect_ratio` | **21:9/16:9/4:3/1:1/3:4/9:16/auto** | Seven ratios |
| `image_urls` | Array | Max 12 reference images |
| `video_urls` | Array | Max 1 video reference (omni_reference) |
| `audio_urls` | Array | mp3/wav, ≤15s (omni_reference) |

### Output Specifications

| Specification | Value |
|------|-----|
| Duration | **4-15s (any integer)** |
| Resolution | 480p / 720p (max 720p)⚠️ |
| Audio | Auto-generated (AAC stereo) |

### Time-Segmented Prompt Syntax

**Format**:
```
Referencing the {segment_id}_frame composition for scene layout and character positioning.

@image1 (character reference image), [viewpoint setting] [theme/style];

Overall: [overall camera movement summary]

Segmented actions ({duration}s):
0-Xs: [scene] + [action] + [camera movement] + [rhythm] + [sound/dialogue];
X-Xs: [cut] + [scene] + [action] + [camera movement] + [rhythm] + [sound/dialogue];
...

Maintain {ratio} composition, do not break the aspect ratio
{BGM constraint}
```

**Example**:
```
Referencing the scene_1_seg_1_frame composition for scene layout and character positioning.

@image1, first-person perspective fruit tea commercial; Element_Chuyue as female character;

Overall: First-person perspective showing the complete fruit tea making process, from picking apples to presenting the final product, natural and smooth.

Segmented actions (10s):
0-2s: Your hand picks a red Aksu apple with morning dew, fixed shot, steady rhythm, crisp apple collision sound;
2-4s: Quick cut, your hand puts apple chunks into a shaker cup, adds ice and tea base, shakes vigorously, camera slightly follows, light and quick rhythm, ice collision sounds hit the beat;
4-6s: First-person close-up of finished product, layered fruit tea pours into transparent cup, your hand gently squeezes milk foam, camera slowly pushes in, steady rhythm, liquid flowing sound;
6-8s: Camera pushes in, pink label applied to cup, showing layered texture, relaxed rhythm, soft background sound;
8-10s: First-person holding cup, @image2, fruit tea raised to camera, fixed shot, steady rhythm, cup label clearly visible, background audio: "Take a fresh sip";

Maintain landscape 16:9 composition, do not break the aspect ratio
Background audio: "Fresh cut and shaken" "Take a fresh sip", female voice tone.
```

### image_urls Order Convention

| index | Purpose | Reference Method |
|-------|------|---------|
| `image_urls[0]` | Storyboard | `Referencing the {segment_id}_frame composition...` |
| `image_urls[1]` | Character reference image 1 | `@image1` |
| `image_urls[2]` | Character reference image 2 | `@image2` |
| ... | ... | ... |
| `image_urls[9]` | Character reference image 9 (max) | `@image9` |

**Key Points**:
1. **Storyboard is reference, not first frame precise control** — Provides overall visual style reference
2. **Character reference images use `<<<image_N>>>` reference** — Unified syntax with Kling-Omni
3. **Time segmentation auto-triggers smart multi-shot** — No `--multi-shot` parameter needed
4. **Max 720p** — Use Kling or Vidu when 1080p is needed

### CLI Usage Examples

```bash
# Text-to-Video (pure text generation)
python video_gen_tools.py video \
  --backend seedance \
  --prompt "Time-segmented description..." \
  --duration 10 \
  --aspect-ratio 16:9 \
  --output output.mp4

# Image-to-Video (storyboard + character reference image)
python video_gen_tools.py video \
  --backend seedance \
  --prompt "Referencing the composition... @image1..." \
  --image-list generated/frames/scene1_frame.png materials/personas/xiaomei_ref.jpg \
  --duration 10 \
  --output output.mp4
```

### Recommended Scenarios

| Scenario | Priority Backend | Fallback Backend | Reason |
|------|---------|---------|------|
| **Fiction/Short Drama** | **Seedance 2** | Kling-Omni | Smart multi-shot + multi-reference, character consistency |
| **Commercial (No Real Footage)** | **Seedance 2** | Kling-Omni | Long shots + smart multi-shot |
| **Commercial (With Real Footage)** | Kling-3.0 / Vidu | — | First frame precise control, real footage |
| **MV Short Film** | **Seedance 2** | Kling-Omni | Long shots + music-driven |
| **Vlog/Documentary Style** | Kling-3.0 | Vidu | First frame precise control, avoid Seedance |
| **Ultra-Short Shot (≤ 4s)** | **Veo3** | Kling | Veo3 minimum 4s, Kling minimum 3s |

---

## Auto Selection Logic

When `--backend` is not specified, **kling** is used by default. Special parameters will force backend switching:
- Providing `--image-list` → Auto-switch to kling-omni (only supported backend)
- Providing `--tail-image` → Keep kling (only supported backend)
- Need fast fallback → Manually specify `--backend vidu`

### Provider Selection Priority

When `--provider` is not specified, auto-select by the following priority:

| Condition | Provider | Description |
|------|----------|------|
| Has KLING_ACCESS_KEY + KLING_SECRET_KEY | `official` | Kling Official API |
| Has FAL_API_KEY | `fal` | fal.ai proxy |

**Manually specify provider**:

```bash
# Use fal.ai proxy
python video_gen_tools.py video --provider fal --backend kling-omni --image-list ref.jpg ...
```

---

## Character Reference Image Two Paths

**Only consider when character reference images are registered**

| | **Path A: Kling Omni (Recommended)** | Path B: Kling + Gemini |
|---|---|---|
| **Flow** | **Storyboard + Character reference → `image_list`** | Storyboard → `--image` → Kling img2video |
| **Advantage** | **Best of both: scene controllable + character consistent** | Precise scene control |
| **Consistency** | **Good (reference image in image_list)** | Moderate |
| **Scene Control** | **Strong (storyboard provides overall visual)** | Strong (storyboard as first frame) |
| **Use Case** | **Story video (recommended)** | First frame precision priority, can accept character variation |

**Selection Recommendation**:
- **Story video, need both → Path A: Kling Omni (Recommended)**
- First frame precision priority, can accept character variation → Path B: Kling + Gemini

---

## Path A: Kling Omni (Recommended)

**Best Practice: Storyboard + Character Reference Dual Reference**

```
Stage 1: Image Prompt → Gemini generates storyboard (control scene/style/lighting/mood/color/makeup)
         ↓
Stage 2: Storyboard + Character reference → Pass as image_list to Omni → Video
```

**Key Insight**:
- **Storyboard** passed as `image_list`, controls overall visual (scene, style, lighting, mood, color, makeup)
- **Character reference image** also passed to `image_list`, ensures character appearance/body consistency
- Omni will reference multiple images comprehensively: storyboard provides overall visual, character reference provides character features

### Quick Prototype (scenarios without scene/character makeup consistency requirements)

Pass only character reference image, no storyboard generation:

```bash
python video_gen_tools.py video \
  --backend kling-omni \
  --prompt "Character <<<image_1>>> sits by the cafe window, smiling and looking outside" \
  --image-list /path/to/person_ref.jpg \
  --audio --output output.mp4
```

### Best Practice (Storyboard + Character Reference)

**Step 1**: Generate storyboard

```bash
python video_gen_tools.py image \
  --prompt "Cinematic realistic start frame.\nReferencing the facial features...\nScene: Men's restroom entrance...\nLighting: Cold white fluorescent..." \
  --reference /path/to/person_ref.jpg \
  --output generated/frames/{shot_id}_frame.png
```

**Step 2**: Pass storyboard + character reference image together to Omni

```bash
python video_gen_tools.py video \
  --backend kling-omni \
  --prompt "Referencing the composition, characters interact in the scene..." \
  --image-list generated/frames/{shot_id}_frame.png /path/to/person_ref.jpg \
  --audio --output output.mp4
```

**Note**: The order of images in `image_list` is important, Omni will reference all images comprehensively. Usually storyboard goes first to provide overall visual, character reference images go after to ensure character features.

### Omni Multi-Reference + multi_shot

```bash
python video_gen_tools.py video --backend kling-omni \
  --prompt "Story" \
  --image-list frame.png ref1.jpg ref2.jpg \
  --multi-shot --shot-type customize \
  --multi-prompt '[{"index":1,"prompt":"Shot 1","duration":"3"},{"index":2,"prompt":"Shot 2","duration":"4"}]' \
  --duration 7
```

### Omni Mode Storyboard Annotation

```json
{
  "shot_id": "scene1_shot2",
  "generation_mode": "omni-video",
  "generation_backend": "kling-omni",
  "video_prompt": "Xiaomei and Xiaoming talking in the cafe...",
  "reference_images": [
    "generated/frames/scene1_shot2_frame.png",
    "/path/to/xiaomei_ref.jpg",
    "/path/to/xiaoming_ref.jpg"
  ],
  "frame_strategy": "frame_as_reference"
}
```

---

## Path B: Kling + Gemini First Frame

**Reminder**: Character reference image is an **appearance reference image**, only takes facial/body features, **cannot directly be used as img2video first frame**. The scene, clothing, and pose in the character reference image are interference.

```
Character reference image → Gemini generates storyboard (specify scene/clothing/pose) → img2video (Kling standard version)
```

**Note**: This path uses standard Kling img2video (`--image` first frame), **does not use** Omni. First frame scene control is good, but character consistency is not as good as Path A.

### Single Person Shot

**Step 1**: Gemini generates storyboard based on reference image

```bash
python video_gen_tools.py image \
  --prompt "Xiaomei (25-year-old Asian woman, long straight black hair, oval face) sitting by the cafe window, looking up and smiling, afternoon sunlight, cinematic, portrait 9:16 composition" \
  --reference <reference_image_path> \
  --output generated/storyboard/scene1_shot2_frame.png
```

**Step 2**: Use storyboard for img2video

```bash
python video_gen_tools.py video \
  --image generated/storyboard/scene1_shot2_frame.png \
  --prompt "Xiaomei looks up at the waiter, smiling gently and says: 'It's really quiet here, I like it.'" \
  --backend kling --audio \
  --output generated/videos/scene1_shot2.mp4
```

### Two/Multi-Person Shot

**Step 1**: Gemini multi-reference composite into one storyboard (**reference image order is important, important characters go after**)

```bash
python video_gen_tools.py image \
  --prompt "Xiaomei and Xiaoming walking side by side on the street, warm golden light, portrait 9:16 composition" \
  --reference <secondary_character_reference> <main_character_reference> \
  --output generated/storyboard/scene2_shot1_frame.png
```

**Step 2**: Use composite image for img2video

### Kling Path Storyboard Annotation

```json
{
  "shot_id": "scene1_shot2",
  "generation_mode": "img2video",
  "generation_backend": "kling",
  "frame_strategy": "first_frame_only",
  "image_prompt": "Xiaomei sitting by the cafe window, looking up and smiling, portrait 9:16 composition",
  "video_prompt": "Xiaomei looks up at the waiter, smiling gently...",
  "reference_personas": ["Xiaomei"]
}
```

---

## Gemini Prompt Notes

Must include:
- Character identity + appearance features (matching reference image)
- Scene description (current storyboard scene, not reference image scene)
- Clothing description (may differ from reference image)
- Lighting and mood
- **Aspect ratio** (portrait 9:16 composition)

**Example**:
```
Reference for Xiaomei: MUST preserve exact appearance - 25-year-old Asian woman, long straight black hair, oval face
Xiaomei sitting by the cozy cafe window, wearing beige knitwear, afternoon sunlight streaming through the window,
cinematic tone, shallow depth of field background blur, portrait composition, 9:16 aspect ratio, character centered in frame
```

---

## Degradation Strategy When API Limited

When API encounters 429 (concurrency limit), 402 (insufficient balance), timeout, or other unrecoverable errors, degradation is needed.

### Degradation Prerequisites

**Must meet the following conditions before degrading**:
1. Already retried once and still failed (Seedance timeout needs to wait 10 minutes)
2. User explicitly agrees to degradation
3. Still has available backend after degradation

### Degradation Path

| Original Mode | Degraded Mode | Backend Change | Capability Change |
|--------|-----------|----------|----------|
| `seedance-video` | `omni-video` (kling-omni) | Seedance → Kling-Omni | Lose smart multi-shot, need manual multi-shot |
| `omni-video` (kling-omni) | `img2video` (kling) | Omni → Kling | Lose multi-reference capability, character consistency reduced |
| `img2video` (kling) | `text2video` (kling) | No change | Lose first frame control |

**Prohibited Degradations**:
- ❌ `seedance-video` → Vidu text2video (Vidu doesn't support image_list)
- ❌ `omni-video` → Vidu text2video (Vidu doesn't support image_list)

### Seedance → Kling-Omni Degradation Flow

When Seedance times out or fails, degrade to Kling-Omni:

**Step 1: Ask User**
```
Seedance generation failed (retried 1 time).

Options:
A. Degrade to Kling-Omni (lose smart multi-shot, need manual multi-shot)
B. Modify prompt and retry Seedance
C. Cancel this generation

Please choose:
```

**Step 2: After user selects A, modify storyboard.json**

```json
// Original (Seedance)
{
  "generation_mode": "seedance-video",
  "generation_backend": "seedance",
  "reference_images": ["storyboard", "character_reference_1", "character_reference_2"]
}

// Degraded (Kling-Omni)
{
  "generation_mode": "omni-video",
  "generation_backend": "kling-omni",
  "reference_images": ["character_reference_1", "character_reference_2"]
}
```

**Step 3: Execute Kling-Omni**

- Provider auto-select by priority: official → fal → yunwu
- Each shot calls API separately (no merge)
- Keep character reference images, maintain character consistency

### Kling-Omni → Kling img2video Degradation Flow

When Kling-Omni cannot execute, degrade to img2video:

**Step 1: Ask User**
```
Kling-Omni API currently unavailable (reason: 429 concurrency limit / 402 insufficient balance).

Options:
A. Wait and retry (may take longer)
B. Degrade to Kling img2video (character consistency will decrease, need to generate storyboard first)
C. Cancel this generation

Please choose:
```

**Step 2: After user selects B, modify storyboard.json**

Need to modify fields for each shot:

```json
// Original (Omni)
{
  "generation_mode": "omni-video",
  "generation_backend": "kling-omni",
  "reference_images": ["character_reference", "storyboard"],
  "frame_strategy": "none"
}

// Degraded (img2video)
{
  "generation_mode": "img2video",
  "generation_backend": "kling",
  "reference_images": [],
  "frame_strategy": "first_frame_only",
  "frame_path": "generated/frames/{shot_id}_frame.png",
  "image_prompt": "..." // Must add, for generating storyboard
}
```

**Step 3: Execute Path B**

1. First generate all storyboards (using Gemini + character reference images)
2. Use storyboards as first frame to call Kling img2video

### storyboard.json Degradation Modification Details

| Field | Path A (omni-video) | Path B (img2video) |
|------|---------------------|---------------------|
| `generation_mode` | `omni-video` | `img2video` |
| `generation_backend` | `kling-omni` | `kling` |
| `reference_images` | `[character_reference, storyboard]` | `[]` (storyboard passed separately) |
| `frame_strategy` | `none` | `first_frame_only` |
| `frame_path` | None | `generated/frames/{shot_id}_frame.png` |
| `image_prompt` | Optional | **Required** |

**Unchanged Fields**:
- `shot_id`, `duration`, `shot_type`, `description`
- `video_prompt` (may need minor adjustment: remove `<<<image_N>>>` references)
- `dialogue`, `transition`, `audio`
- `characters` (characters involved in the shot)

### Pre-set fallback_plan (Recommended)

During Phase 3 storyboard generation, pre-write degradation plan to avoid writing `image_prompt` on the fly:

```json
{
  "shot_id": "scene1_shot1",
  "generation_mode": "omni-video",
  "generation_backend": "kling-omni",
  "video_prompt": "...",
  "reference_images": ["..."],

  "fallback_plan": {
    "mode": "img2video",
    "backend": "kling",
    "image_prompt": "Cinematic realistic start frame.\nScene: ...\nLighting: ...\nStyle: ...",
    "frame_strategy": "first_frame_only",
    "reason": "Fallback when Omni API unavailable"
  }
}
```

When degrading, just switch fields and use `fallback_plan.image_prompt` directly.