# 🎬 Video-Gen - AI Video Editing Skill

A Claude Code Skill that brings AI video editing capabilities into your conversations.

## 🏗️ Architecture

**Core Concept**: Claude itself acts as the Director Agent, no additional Agent code needed.

```
~/.claude/skills/video-gen/
├── SKILL.md                # Core workflow instructions (~290 lines)
├── reference/
│   ├── storyboard-spec.md  # Complete storyboard design specification
│   ├── backend-guide.md    # Backend selection and reference image strategy
│   ├── prompt-guide.md     # Prompt writing and consistency specification
│   └── api-reference.md    # CLI parameters and environment variables
├── video_gen_tools.py           # API tools (video/music/TTS/image generation)
├── video_gen_editor.py          # FFmpeg editing tools
└── config.json             # API key configuration
```

**Responsibility Division**:
- **Claude**: Intent recognition, creative generation, storyboard design, workflow planning
- **video_gen_tools.py**: Vidu/Kling/Suno/TTS/Gemini API calls
- **video_gen_editor.py**: FFmpeg video editing operations

## ✨ Features

- ✅ **Material Analysis** - Automatic recognition of image/video content, scenes, emotions
- ✅ **Creative Generation** - Interactive question cards, customized video creative plans
- ✅ **Storyboard Design** - Generate storyboard scripts and video generation prompts
- ✅ **AI Video Generation**
  - **Seedance 2** (recommended for fiction): 4-15s, smart shot switching, multi-reference images, audio-video synchronized output
  - **Kling v3**: 3-15s, precise first-frame control, good visual quality
  - **Kling v3 Omni**: 3-15s, multi-reference images, best character consistency
  - **Veo3**: 4/6/8s, high-quality realistic shorts
- ✅ **AI Music Generation** - Suno V3.5 background music
- ✅ **TTS Voice Synthesis** - Gemini TTS (multiple voices, style prompts)
- ✅ **AI Image Generation** - Gemini 3.1 Flash Image (compass/yunwu)
- ✅ **Video Editing** - Transitions, subtitles, color grading, speed change, audio mixing

## 💡 Usage Tips

### Recommended Model

**Recommended to use multimodal models (such as Kimi K2.5) for the best experience.**

Multimodal models have stronger understanding of materials and can more accurately identify scenes, characters, emotions, and visual styles in images/videos. If your primary model is not multimodal, you can supplement visual understanding by calling the `/vision` skill.

### Guiding the Model

Current capabilities are still relatively basic, **it is recommended to guide the model more during use**. For example:
- Actively describe the desired video style, pace, and emotion
- Provide specific modification suggestions for the storyboard plan
- Provide timely feedback during the generation process to help the model adjust direction

### Project Positioning

The original intention of this project is to fully leverage Claude Code Agent's intelligence capabilities, providing all video generation-related tools and capabilities, exploring the practical effects of AI-assisted video creation. **This is an exploratory project that is still constantly being updated and iterated.** You are welcome to try various creative ideas, and your usage feedback will help us continue to improve.

**Flexible API Support**: The image and video generation APIs used in this project (such as Vidu, Gemini) can be replaced with providers you are familiar with. The API calls in `video_gen_tools.py` are clearly encapsulated, making it easy to integrate other service providers (such as OpenAI, Midjourney, Stability AI, etc.).

## 🚀 Installation

```bash
# Copy the entire directory to the skills directory
mkdir -p ~/.claude/skills/video-gen
cp -r SKILL.md reference/ video_gen_tools.py video_gen_editor.py config.json.example README.md requirements.txt ~/.claude/skills/video-gen/

# Install dependencies
cd ~/.claude/skills/video-gen && pip install -r requirements.txt

# Configure API keys
cp config.json.example config.json
# Edit config.json to fill in your API keys
```

## 📖 Usage

```
/video-gen <material_directory>
```

### Examples

```bash
# Complete creative process
/video-gen ~/Videos/travel_materials/

# Continue a previous project
/video-gen ~/video-gen-projects/trip_20260310/
```

## 🛠️ Tool Calls

### video_gen_tools.py

```bash
# Video generation (Kling backend, default)
python video_gen_tools.py video --prompt "<description>" --duration 5 --output video.mp4
python video_gen_tools.py video --image <first_frame_image> --prompt "<description>" --output video.mp4

# Video generation (Kling Omni backend - reference image mode)
python video_gen_tools.py video --backend kling-omni --prompt "<<<image_1>>> in the scene" --image-list <reference_images> --output video.mp4

# Video generation (Vidu backend - fallback/rapid prototyping)
python video_gen_tools.py video --backend vidu --image <image> --prompt "<description>" --duration 5 --output video.mp4

# Kling multi-shot mode
python video_gen_tools.py video --prompt "<story_description>" --multi-shot --shot-type intelligence --duration 10
python video_gen_tools.py video --prompt "<overall_description>" --multi-shot --shot-type customize --multi-prompt '[{"index":1,"prompt":"shot 1","duration":"3"}]' --duration 5

# Kling first/last frame control
python video_gen_tools.py video --image <first_frame_image> --tail-image <last_frame_image> --prompt "<action_description>" --duration 5

# Music generation
python video_gen_tools.py music --prompt "<description>" --style "Lo-fi" --output music.mp3

# TTS
python video_gen_tools.py tts --text "<text>" --voice female_narrator --output audio.mp3

# Image generation
python video_gen_tools.py image --prompt "<description>" --style cinematic --output image.png
```

### 🎥 Video Generation Backend Comparison

| Backend | Model | Duration | Features |
|------|------|------|------|
| **Seedance 2** | seedance-2 | 4-15s | Smart shot switching, multi-reference images (up to 9), first/last frame control, audio-video synchronized output |
| **Kling Omni** | kling-3.0-omni | 3-15s | Multi-reference images (reference2video), character consistency, audio-video synchronized output |
| **Kling** | kling-3.0 | 3-15s | Precise first-frame control (img2video), good visual quality |
| **Veo3** | veo-3.1-generate-001 | 4/6/8s | High-quality realistic shorts, first-frame control, default audio |

**Key Differences**:
- **Seedance 2 / Kling Omni** support multi-reference images (character consistency), Seedance 2 also supports first/last frame control
- **Kling / Veo3** support precise first-frame control

**Selection Recommendations**:
| Scenario | Priority Backend | Fallback Backend | Reason |
|-----|---------|---------|------|
| **Fiction/Short Drama** | **Seedance 2** | Kling-Omni | Smart shot switching + multi-reference images |
| **Commercial (no real materials)** | **Seedance 2** | Kling-Omni | Long shots + smart shot switching |
| **Commercial (with real materials)** | Kling-3.0 | — | Precise first-frame control |
| **MV Shorts** | **Seedance 2** | Kling-Omni | Long shots + music-driven |
| **Vlog/Documentary** | Kling-3.0 | Veo3 | Precise first-frame control |
| **High-quality Documentary Shorts** | Kling-3.0 | Veo3 | Veo3 as fallback only, 4/6/8s |

### video_gen_editor.py

```bash
# Concatenation
python video_gen_editor.py concat --inputs v1.mp4 v2.mp4 --output out.mp4

# Subtitles
python video_gen_editor.py subtitle --video video.mp4 --srt subs.srt --output out.mp4

# Audio mixing
python video_gen_editor.py mix --video video.mp4 --bgm music.mp3 --output out.mp4

# Transitions
python video_gen_editor.py transition --inputs v1.mp4 v2.mp4 --type fade --output out.mp4

# Color grading
python video_gen_editor.py color --video video.mp4 --preset warm --output out.mp4

# Speed change
python video_gen_editor.py speed --video video.mp4 --rate 1.5 --output out.mp4
```

## 🔑 Environment Variables

```bash
# Seedance API - Seedance 2 video generation (recommended for fiction/short drama)
export SEEDANCE_API_KEY="your-seedance-api-key"

# Kling API - Kling v3 video generation (recommended for documentary/Vlog)
export KLING_ACCESS_KEY="your-access-key"
export KLING_SECRET_KEY="your-secret-key"

# Veo3 API - Google Veo3 video generation (high-quality documentary shorts)
export COMPASS_API_KEY="your-compass-api-key"

# Suno music generation
export SUNO_API_KEY="your-api-key"

# Gemini image generation (compass priority)
export COMPASS_API_KEY="your-compass-api-key"
export YUNWU_API_KEY="your-yunwu-api-key"  # Backup
```

**Note**:
- **Video Generation Provider**: Seedance (piapi), Kling (official/fal), Veo3 (compass)
- **Image Generation Provider Priority**: compass → yunwu

## 🔄 Workflow

```
Material Analysis → Creative Generation → Storyboard Design → Content Generation → Editing Output
```

## 📁 Output Directory Structure

```
~/video-gen-projects/{project_name}_{timestamp}/
├── state.json              # Project state
├── materials/              # Original materials
├── analysis/               # Analysis results
├── creative/               # Creative plan
├── storyboard/             # Storyboard script
├── generated/              # Generated content
│   ├── videos/
│   └── music/
└── output/                 # Final video
```

## 📦 Dependencies

- FFmpeg 6.0+ (video processing)
- Python 3.9+ (tool execution)
- httpx (HTTP client)

## 📋 Changelog

### v1.6.1 (2026-04-07)
🐛 **Bug Fixes**

- 🐛 **Seedance auto-assembly mode image path resolution** — When CLI run directory differs from project directory, relative paths could not be resolved correctly, causing `FileNotFoundError`. Added `resolve_path()` function to automatically convert relative paths to absolute paths
- 🐛 **narration command audio_idx calculation error** — FFmpeg filter_complex audio input index calculation error (`len(inputs) // 2` → `i + 1`), causing multi-narration scenes to reference non-existent input streams

### v1.6.0 (2026-04-07)
🔄 **Provider System Refactoring + Seedance 2 Upgrade**

#### Deprecation Cleanup
- 🗑️ **Remove yunwu video generation Provider** — Vidu/Kling/Kling-Omni yunwu providers all deprecated, yunwu only retains Gemini image generation
- 🗑️ **Remove FalImageClient** — Image generation only retains compass/yunwu providers
- 🗑️ **Deprecate Volcano Engine TTS** — TTS only retains Gemini TTS (via Compass API)
- 🗑️ **Remove Vidu backend** — No longer support Vidu video generation

#### Seedance 2 Upgrade
- ✨ **Duration support extended** — From 5/10/15s enum values to 4-15s any integer
- ✨ **Add 21:9 aspect ratio** — Support for cinematic widescreen ratio
- ✨ **Add `--mode` parameter** — `text_to_video` / `first_last_frames` / `omni_reference`
- ✨ **Add `--audio-urls` / `--video-urls`** — Support audio/video references
- ✨ **First/last frame control** — `mode: first_last_frames` supports precise first/last frame control

#### Architecture Optimization
- 🔄 **Provider matrix simplified** — Video 4 backends (Seedance/Kling/Kling-Omni/Veo3), image 2 providers (compass/yunwu)
- 🔄 **TTS unified to Gemini** — Remove Volcano Engine TTS call paths
- 📝 **Comprehensive documentation update** — SKILL.md, backend-guide.md, api-reference.md synchronized updates

#### Currently Supported Models
| Type | Model | Provider |
|------|------|----------|
| Video | Seedance 2 | piapi |
| Video | Kling v3 | official / fal |
| Video | Kling v3 Omni | official / fal |
| Video | Veo3 | compass |
| Image | Gemini 3.1 Flash Image | compass / yunwu |
| TTS | Gemini TTS | compass |
| Music | Suno V3.5 | official |

### v1.5.1 (2026-04-03)
🎤 **Gemini TTS Integration**

#### New Features
- ✨ **GeminiTTSClient** — New Gemini TTS client (via Compass API)
  - Priority over Volcano Engine TTS
  - Support style prompts (prompt parameter)
  - Support inline emotion tags: `[brightly]`, `[sigh]`, `[pause]`
  - Voices: Kore/Aoede/Charon/Orus etc.

#### Voice Presets
| Preset | Voice | Gender |
|------|------|------|
| `female_narrator` | Kore | Female |
| `female_gentle` | Aoede | Female (bright)|
| `female_soft` | Zephyr | Female (soft)|
| `male_narrator` | Charon | Male |
| `male_warm` | Orus | Male (steady)|

#### TTS Priority
- **Gemini TTS** (COMPASS_API_KEY) → Volcano Engine TTS (VOLCENGINE_TTS_*)

### v1.5.0 (2026-04-03)
🎬 **Seedance Smart Shot Switching + fal Image Generation**

#### New Features
- ✨ **SeedanceClient** — New Seedance video generation client (via piapi.ai proxy)
  - Smart shot switching: Time-segmented prompts auto-trigger multi-shot
  - Duration limit: Only supports 5/10/15s (three enum values)
  - Aspect ratios: 16:9 / 9:16 / 4:3 / 3:4
  - Image reference syntax: `@imageN` (not `<<<image_N>>>`)
- ✨ **FalImageClient** — New fal.ai Gemini 3.1 Flash image generation client
  - Text-to-image, image-to-image, multi-reference image editing
- ✨ **narration command** — video_gen_editor.py new narration synthesis command

#### Architecture Optimization
- 🔄 **Provider priority adjustment** — yunwu moved to last: `official → fal → yunwu`
- 🔄 **visual_style semantic change** — From "backend selection" to "user photo processing method"
  - `realistic`: Seedance needs to generate three-view conversion first, Kling-Omni can use directly
  - `anime`: Can be used as reference image directly

#### Documentation Corrections
- 📝 **Seedance duration limit** — Corrected to 5/10/15s (not 4-15s range)
- 📝 **Seedance image syntax** — Corrected to `@imageN`
- 📝 **Remove 21:9** — Seedance does not support 21:9 aspect ratio
- 📝 **Time-segmented prompt format** — Added complete templates and examples

#### File Changes
- 📝 `video_gen_tools.py` — Added SeedanceClient, FalImageClient
- 📝 `video_gen_editor.py` — Added narration command
- 📝 `SKILL.md` — Added Seedance execution logic, time-segmented format, photo conversion flow
- 📝 `reference/backend-guide.md` — Updated Provider priority, Seedance parameters
- 📝 `reference/storyboard-spec.md` — Added Seedance duration planning chapter

### v1.4.6 (2026-04-02)
🔧 **API Field Name Fix**

#### Bug Fixes
- 🐛 **FalKlingClient field name correction** — Use fal.ai official correct field names
  - `image_url` → `start_image_url` (first frame control)
  - `tail_image_url` → `end_image_url` (last frame control)
- 🐛 **YunwuKlingOmniClient parameter correction**
  - `audio` boolean → `sound: "on"/"off"` string
  - `_file_to_base64()` returns pure base64 not data URI format

#### Test Verification
- ✅ Text-to-video (pure prompt)
- ✅ First-frame-to-video (start_image_url, character retention correct)
- ✅ Multi-reference-to-video (image_urls + @Image1/@Image2, with audio)

### v1.4.5 (2026-04-02)
📝 **Documentation Supplement**

- Supplemented yunwu provider documentation notes missed in v1.4.4

### v1.4.4 (2026-04-02)
📚 **Yunwu Provider Documentation Enhancement**

#### Documentation Updates
- 📝 `api-reference.md` — YUNWU_API_KEY scope extended, supports Kling/Kling-Omni
- 📝 `backend-guide.md` — Added Provider selection priority explanation
- 📝 `README.md` — Added Kling can use Yunwu as official API backup
- 📝 `SKILL.md` — Added Provider selection chapter, explaining how to bypass concurrency limits

### v1.4.3 (2026-04-02)
🔌 **Yunwu Kling Provider Support**

#### New Features
- ✨ **YunwuKlingClient** — New yunwu kling-v3 client, supports text2video, img2video, multi_shot, first/last frame control, audio
- ✨ **YunwuKlingOmniClient** — New yunwu kling-v3-omni client, supports omni-video, image_list multi-reference images, multi_shot, audio
- ✨ **--provider parameter** — New provider selection (official/yunwu/fal), supports switching different providers for the same backend
- ✨ **Provider auto-selection** — When not specified, auto-select by priority: official > fal > yunwu

#### Architecture Optimization
- 🔄 **Backend/Provider separation** — backend selects functionality (vidu/kling/kling-omni), provider selects service source (official/yunwu/fal)
- 📝 **Feature support matrix** — Clarify feature support for each provider (multi_shot, first/last frame, audio, etc.)

#### API Difference Handling
- 🔧 **Yunwu kling-v3 uses `model` parameter** (official API uses `model_name`)
- 🔧 **Yunwu kling-v3-omni uses `model_name` parameter** (same as official API)
- 🔧 **Video URL parsing path** — `data.task_result.videos[0].url` (not `task_info`)

#### File Changes
- 📝 `video_gen_tools.py` — Added YunwuKlingClient, YunwuKlingOmniClient, modified cmd_video function

### v1.4.2 (2026-04-01)
🔊 **Audio Mixing Fix and Normalization**

#### Bug Fixes
- 🐛 **FFmpeg amix normalize=0** — Disable auto-normalization, preserve original volume ratio, fix narration being suppressed issue

#### New Features
- ✨ **Mixing rules documentation** — SKILL.md Phase 5 added "Audio Mixing Rules" chapter
  - Volume recommendations: Video ambient 0.8, narration 1.5-2.0, BGM 0.1-0.15
  - Video type adaptation: MV → 0.5-0.7, Vlog → 0.1-0.15, Cinematic → 0.2-0.3

#### File Changes
- 📝 `video_gen_editor.py` — mix_audio() function added normalize=0 (line 470)
- 📝 `SKILL.md` — Phase 5 added audio mixing rules chapter

### v1.4.1 (2026-03-31)
🎤 **Narration Segmentation Planning Feature**

#### New Features
- ✨ **Phase 2 narration requirement judgment** — Recommend whether narration is needed based on video type (documentary/Vlog usually needs, cinematic/fiction usually doesn't)
- ✨ **Phase 3 simultaneous narration design** — When generating storyboard, simultaneously plan `narration_segments`, segmented by shot timing
- ✨ **Phase 4 narration generation** — After video/music generation, add TTS narration generation step (Volcano Engine)
- ✨ **Phase 5 narration insertion** — Insert narration audio at correct position by `overall_time_range` timing

#### Documentation Updates
- 📝 `storyboard-spec.md` — Added `narration_config` and `narration_segments` field specifications
- 📝 `prompt-guide.md` — Added TTS narration generation flow and parameter instructions

### v1.4.0 (2026-03-30)
🎬 **Video Generation Best Practices Refactoring**

#### Core Architecture Changes
- ✨ **Project type-driven decision** — Auto-detect project type based on user intent (fiction/short drama, Vlog/documentary, commercial/promo, MV shorts), no manual selection needed
- ✨ **Fiction disables text2video** — All fiction content forced to generate storyboard images first, then use reference2video or img2video
- ✨ **Unified model within project** — No mixing of multiple models within a project, once selected, use throughout the project

#### Model and Generation Paths
- 📝 **Update model names** — Kling-3.0-Omni, Kling-3.0, Vidu Q3 Pro
- 📝 **Clarify model capability boundaries** — Kling-3.0-Omni supports reference2video but **does not support img2video (first frame control)**
- 📝 **Optimize decision matrix** — Fiction prioritizes Omni (character consistency), Vlog/commercial uses Kling/Vidu (first frame control)

#### Bug Fixes
- 🐛 **Omni reference format correction** — `image_1` → `<<<image_1>>>`, conforming to official documentation requirements

#### Documentation Updates
- 📝 `SKILL.md` — Backend selection overview, Phase 3 decision tree rewritten
- 📝 `storyboard-spec.md` — Reference Tag format, T2V/I2V/Ref2V selection rules rewritten
- 📝 `prompt-guide.md` — Omni mode reference format correction

### v1.3.10 (2026-03-23)
🎵 **Music Generation Parameter Normalization**

#### Fixes
- 🐛 **music command style parameter must be provided** — Removed Lo-fi Chill default value to avoid style mismatch
  - Must read from creative.json via `--creative`
  - Or manually pass `--prompt` and `--style` parameters

### v1.3.9 (2026-03-23)
🎬 **Audio-Video Sync Fix**

#### Fixes
- 🐛 **Video concatenation audio-video desync issue** — Silent segments caused subsequent video audio misalignment
  - Added `has_audio_track()` to detect if video has audio track
  - `normalize_videos()` auto-adds silent track for silent segments
  - `concat_videos()` switched to concat filter, ensuring audio-video sync

#### Improvements
- 🔄 `music` command `--prompt` changed to non-required, can read from creative.json
- 📝 SKILL.md: Phase 5 added audio protection instructions

### v1.3.8 (2026-03-23)
🔧 **Parameter Passing Normalization**

#### Fixes
- 🐛 **Hardcoded default value issue** — CLI parameters should prioritize reading from storyboard.json
  - `video_gen_editor.py`: concat/image commands added `--storyboard` parameter
  - `video_gen_tools.py`: video/image added `--storyboard`, music added `--creative` parameter
  - Unified KlingClient/KlingOmniClient default aspect_ratio to `"9:16"`

#### Improvements
- 🔄 Suno logs show both prompt and style, avoiding misleading users

### v1.3.7 (2026-03-20)
🔧 **Execution Phase Fix & Image Size Optimization**

#### Fixes
- 🐛 **Phase 4 aspect_ratio passing** — Execution phase must read aspect ratio from storyboard.json and pass to CLI
- 🐛 Fixed image size inconsistency causing generation failures

#### New Features
- ✨ **Image size auto-validation and adjustment** — Added `validate_and_resize_image()` function
  - Min edge < 720px auto-upscale to 1280px
  - Max edge > 2048px auto-downscale to 2048px
  - Auto-process before Kling/KlingOmni calls

### v1.3.6 (2026-03-20)
📝 **Documentation Fix**

#### Fixes
- 🐛 Fixed conflicts and ambiguous descriptions in `storyboard-spec.md`
- 📝 Clarified three-layer structure field definitions and usage scenarios

### v1.3.5 (2026-03-19)
🎬 **Character Consistency Flow Enhancement**

#### New Features
- ✨ **Phase 1 Character Registration Enhancement**
  - Added `personas.json` structure, supports `reference_image` being null
  - Only processes user-uploaded reference images, missing ones wait for Phase 2 supplement

- ✨ **Phase 2 Character Reference Image Collection**
  - Added question 6: Character reference image source selection
  - Supports three methods: AI generation / User upload / Pure text generation
  - Auto-calls `video_gen_tools.py image` to generate standard character reference images

- ✨ **Phase 3 Auto Backend Selection**
  - Has reference image + multi-shot character → `kling-omni` (best character consistency)
  - Has reference image + single-shot character → `kling` (precise first frame control)
  - No reference image + character → `kling` text2video (user warned)
  - Pure scene no character → `kling` text2video

#### PersonaManager Enhancement
- ✨ `list_personas_without_reference()` — List characters without reference images
- ✨ `update_reference_image()` — Update character reference image path
- ✨ `export_for_storyboard()` — Export in storyboard-compatible format
- ✨ `get_character_image_mapping()` — Generate character_image_mapping

#### SKILL.md Flow Optimization
- 📝 Added "Must read before storyboard generation" step
- 📝 Added "Step 1: Sync character info to Storyboard"
- 📝 Improved deliverable documentation and JSON structure examples

### v1.3.4 (2026-03-19)
🎬 **Kling V3-Omni Two-Stage Workflow Normalization**

#### Documentation Updates
- ✨ **Added V3-Omni three-layer structure spec** — storyboard + frame_generation + video_generation
  - `storyboard-spec.md`: Added three-layer schema definitions (storyboard/frame/video)
  - `prompt-guide.md`: Added Image Prompt and Video Prompt writing specifications
  - `backend-guide.md`: Added Path C (V3-Omni recommended path), updated decision tree and path comparison

- 📝 **Image Prompt Specification**
  - Structure: Cinematic realistic start frame → Character refs → Scene → Lighting → Camera → Style
  - Must include character references (image_1, image_2...), aspect ratio, scene, lighting, camera parameters

- 📝 **Video Prompt Specification**
  - Structure: Referencing frame composition → Motion segments → Dialogue exchange → Camera → Sound
  - Time segment format ("0-2s", "2-5s"), dialogue sync markers, sound design description

#### Architecture Adjustment
- 🗑️ Removed incorrectly created `vico-templates` Python code (should be documentation specs not code templates)

### v1.3.3 (2026-03-18)
📐 **SKILL.md Architecture Refactoring & Default Backend Switch**

#### Architecture Refactoring (Anthropic Skill Spec Optimization)
- ✨ **Progressive disclosure architecture** — SKILL.md compressed from 1401 lines to ~290 lines (-80%), complying with Anthropic's recommended 500-line limit
- ✨ **Split into 4 sub-files** — Storyboard spec, Prompt guide, Backend selection, API reference, loaded on demand
- ✨ **Optimized description** — Added Kling Omni keywords and trigger condition descriptions
- ✨ **Added workflow checklist** — Anthropic recommended checklist pattern
- ✨ **Added config.json.example** — Safe configuration template (no real keys)
- 🔄 Streamlined redundant content: Removed duplicate explanations, legacy format compatibility, overly long JSON examples

#### Default Backend Switch
- 🔄 **Default backend changed from Vidu to Kling** — CLI `--backend` default value `vidu` → `kling`
- ✨ **Enhanced auto-selection logic** — Force switch backend by feature requirements (`--image-list` → omni, `--tail-image` → kling), no longer limited to default backend trigger

### v1.3.2 (2026-03-18)
🎬 **Kling Omni Backend Integration**

#### New Features
- ✨ **Kling Omni API Support**
  - Added `--backend kling-omni` backend option
  - `--image-list` multi-reference image mode, use `<<<image_1>>>` reference in prompt
  - Supports multi-reference images + multi_shot combination
- ✨ **Auto Backend Selection**
  - Providing `--image-list` auto-uses kling-omni
  - Providing `--tail-image` auto-uses kling
- ✨ **Three-backend selection strategy** — Core trade-off between character consistency and scene precision
- ✨ **Two character reference image paths** — Omni path (recommended) vs Kling+Gemini first frame path

#### CLI Updates
- 🔧 Added `--image-list` parameter (Kling Omni multi-reference images)
- 🔧 Added `--backend kling-omni` option

### v1.3.1 (2026-03-17)
📋 **Storyboard Structure Optimization & Flow Enhancement**

#### Storyboard Structure Upgrade
- ✨ **Scene-Shot Two-Layer Structure**
  - Changed from single-layer `shots` array to `scenes` -> `shots` two-layer structure
  - Added scene fields: `scene_id`, `scene_name`, `narrative_goal`, `spatial_setting`, `time_state`, `visual_style`
  - Scene duration auto-calculated (sum of subordinate shot durations)

- ✨ **shot_id Naming Normalization**
  - New format: `scene{scene_number}_shot{shot_number}`
  - Single shot example: `scene1_shot1`, `scene1_shot2`
  - Multi-shot mode: `scene1_shot2to4_multi` (with `_multi` suffix)

#### Field Optimization
- 🔄 `vidu_prompt` → `video_prompt` (generic name)
- ✨ Added fields:
  - `multi_shot`: Whether it's multi-shot mode
  - `generation_backend`: Backend selection (kling/vidu)
  - `frame_strategy`: First/last frame strategy (none/first_frame_only/first_and_last_frame)
  - `multi_shot_config`: Kling multi-shot configuration
  - `reference_personas`: Referenced character reference images

#### Flow Enhancement
- 📝 **T2V/I2V Selection Rules**: Decision tree + rule table
- 📝 **First/Last Frame Generation Strategy**: Supports `image_tail` parameter
- 📝 **Dialogue Integration Rules**: video_prompt directly includes dialogue information
- 📝 **Review Check Mechanism**: Automated check items (structure completeness, shot rules, prompt specs, technical selection)

#### Character Reference Image Flow
- 📝 Enhanced character reference image usage flow:
  - Core principle: Reference images cannot be directly used as first frame
  - Complete flow: `Character reference image → Gemini generates storyboard image → img2video`
  - Single/dual character shot processing solutions
  - Storyboard JSON marks `reference_personas` and `notes`

#### CLI Updates
- 🔧 Added `--multi-shot` parameter to enable multi-shot mode
- 🔧 Added `--shot-type` parameter to select shot type (intelligence/customize)
- 🔧 Added `--multi-prompt` parameter to pass custom shot list (JSON format)
- 🔧 Added `--tail-image` parameter to support first/last frame control

### v1.3.0 (2026-03-16)
🎬 **Kling Video Generation API Integration**

#### New Features
- ✨ **Kling API Support**
  - Added KlingClient class, supports Kling v3 model
  - JWT Token authentication method (iss, iat, exp, nbf)
  - Text-to-video (text2video) and image-to-video (image2video)
  - Supports 3-15 second duration range
  - Supports std/pro generation modes
  - Supports audio-video synchronized output (sound: on/off)

- ✨ **Multi-Shot Mode**
  - Supports generating videos with multiple shots in one generation
  - intelligence mode (AI auto shot division)
  - customize mode (custom shots)

#### CLI Updates
- 🔧 Added `--backend` parameter to select video generation backend (vidu/kling)
- 🔧 Added `--mode` parameter to select generation mode (std/pro)

#### Documentation Updates
- 📝 SKILL.md added Kling usage instructions and multi-shot storyboard design documentation
- 📝 README updated feature list, tool call examples, environment variable configuration

### v1.2.0 (2026-03-16)
🎯 **Storyboard Flow Normalization & User Experience Optimization**

#### Flow Specifications
- ✅ **Video Ratio Full-Flow Constraint**
  - Text-to-image, image-to-video both phases' prompts forced to include ratio information
  - 9:16/16:9/1:1 different ratios have clear Chinese description specifications
  - Auto-check ratio consistency before generation

- ✅ **Strict Storyboard-Driven Generation Mode**
  - `generation_mode` must be clearly specified in storyboard, strictly forbidden to change during execution
  - img2video/text2video/existing three modes have strict execution rules
  - Stop execution immediately and report error when violation occurs

- ✅ **Dialogue Generation Method Clear Distinction**
  - Sync sound (video model generated) vs post TTS narration usage scenarios clearly divided
  - Sync sound clearly describes character, dialogue, emotion, speech rate, voice quality in vidu_prompt
  - TTS only for scene narration, background intro, not for character dialogue

- ✅ **Prompt Detail Level and Language Specification**
  - Video generation prompts must be written in Chinese
  - Forced to include camera movement, motion pace, visual stability, ratio protection, dialogue information
  - Image generation prompts must include scene, subject, lighting, style, ratio five elements

- ✅ **Material Consistency Mandatory Guarantee**
  - Cross-shot characters must include detailed identity markers and appearance descriptions in prompt
  - Key props establish material list, each shot includes complete description to ensure consistency
  - Provided character/prop description Prompt templates and examples

- ✅ **Mandatory User Storyboard Confirmation**
  - Storyboard plan must receive explicit user confirmation before entering execution phase
  - Detailed display of each shot's generation mode, prompt, duration, transition, etc.
  - state.json added `storyboard_confirmed` and `confirmation_details` fields

#### Documentation Optimization
- 📝 README added "Usage Tips" section, explaining recommended models, guiding techniques, project positioning

### v1.1.0 (2026-03-12)
🔧 **Stability Enhancement & Character Management**

#### New Features
- ✨ **Video Parameter Auto-Validation and Normalization**
  - Auto-detect all videos' resolution, encoding, frame rate before concatenation
  - Auto-normalize to unified format when parameters inconsistent (1080x1920 / H.264 / 24fps)
  - Resolved video freezing caused by inconsistent API-returned video resolutions

- ✨ **Character Persona Manager (PersonaManager)**
  - Manage character reference images in project, maintain cross-scene character consistency
  - Auto-generate Vidu/Gemini compatible prompts
  - Support different strategies for single/dual character shots

#### Documentation Optimization
- 📝 SKILL.md added conditional workflows:
  - Character identification flow (only triggers when material is portrait image)
  - Character reference image strategy (single/dual character shot processing solutions)
  - Video parameter validation flow (must execute before concatenation)
- 📝 Added key experience summary: Gemini multi-reference image notes, video generation parameter differences

#### Fixes
- 🐛 Fixed concatenation issue caused by text2video (720x1280) and image2video (716x1284) resolution inconsistency

### v1.0.0 (2026-03-10)
🎉 **Complete Initial Release**

#### Core Features
- ✨ **AI Director Workflow**: Complete video creation process - Material Analysis → Creative Confirmation → Storyboard Design → Generation → Editing Output
- ✨ **Multi-modal AI Generation**:
  - Vidu Q3 Pro image-to-video/text-to-video (720p/1080p, up to 10 seconds)
  - Suno V3.5/V4.5 music generation (custom style, duration, instrumental)
  - Volcano Engine TTS voice synthesis (multiple voices, emotions, speech rates)
  - Gemini image generation (multiple styles, ratios)
- ✨ **Professional Editing Tools**:
  - Video concatenation (auto-adjust resolution to 9:16/16:9/1:1)
  - Transitions (16 types: fade, dissolve, wipe, slide, etc.)
  - Audio mixing (auto-loop BGM to match video duration, volume adjustment)
  - Color grading presets (warm, cool, vivid, cinematic, vintage, etc.)
  - Speed change, subtitle support
- ✨ **Auto Project Management**:
  - Auto-create project directory structure
  - State tracking and checkpoint resume
  - All intermediate artifacts auto-saved
- ✨ **Interactive Creation**:
  - Auto material recognition and analysis
  - Question card style creative confirmation
  - Storyboard preview and adjustment

#### Technical Implementation
- ✨ Async API calls based on httpx, supports concurrent generation
- ✨ FFmpeg-based video processing, excellent performance
- ✨ Auto environment check and dependency management
- ✨ Comprehensive error handling and retry mechanism
- 🐛 Fixed Suno API callbackUrl missing issue, all features working

## 📄 License

MIT