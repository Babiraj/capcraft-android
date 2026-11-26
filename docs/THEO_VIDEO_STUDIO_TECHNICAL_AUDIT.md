# THEO VIDEO STUDIO — Technical Audit (Phase 1)

**Audit date:** 2026-11-26  
**Repository:** `Babiraj/capcraft-android`  
**Audited branch baseline:** `main` at `637346f5b9ef8ac76e64e5a8f9081942f845adfe`  
**Audit branch:** `feature/theo-video-studio-audit`  
**Reference project:** `Augani/openreel-video`

## 1. Executive summary

THEO VIDEO STUDIO should continue from the existing Android-native project. Replacing it with the OpenReel web application would be a regression for the Android APK-first requirement.

The Android project already has a Kotlin + Jetpack Compose application, a Rust media engine exposed through UniFFI, project/media storage, a media picker, editing panels, and an Android APK workflow. The previous GitHub Actions run (#1, 2026-11-25) completed successfully and uploaded the APK artifact. That proves the CI build pipeline worked for that commit; it does not prove every editor feature works correctly on a physical device.

## 2. Repository facts verified

- Default branch: `main`
- Current baseline commit: `637346f5b9ef8ac76e64e5a8f9081942f845adfe`
- Fork: `Babiraj/capcraft-android`, forked from `artcraft-android/capcraft-android`
- Repository license: AGPL-3.0
- App module: `src/crates/capcraft-android/android`
- Rust workspace: `src`
- Android UI: Kotlin + Jetpack Compose
- Native media engine: Rust; Kotlin bindings generated with UniFFI
- Minimum Android API: 29 (Android 10)
- Current configured ABI: `arm64-v8a`
- Current app version: `0.0.4-beta`
- Android application ID / namespace: `io.github.artcraftandroid.capcraft`

Do not change the application ID during this phase. Changing it would create a different Android app identity and can prevent in-place updates over the installed build.

## 3. Existing implementation found

The repository tree contains native Android screens and panels for:

- Home, editor, timeline, preview, export and project management
- Media picker and project media import
- Clip tools, crop, speed, volume, opacity and transforms
- Audio effects, sounds, voiceover, voice isolation and beat tools
- Text, captions, stickers, drawing and overlays
- Filters, adjustments, effects, masks, transitions and stabilization
- Auto captions, auto reframe, background removal, enhancement and other assisted tools

These are source-code findings, not a claim that every tool has passed runtime QA on the user's phone.

### Media import path

The Android source includes:

- `ui/common/MediaPicker.kt` — Android media picker/grid and permission handling
- `engine/Media.kt` — copies a selected content URI into the project's media folder
- `ui/editor/EditorViewModel.kt` — editor session and timeline state

Therefore, Gallery import is implemented in source. The next gate is a physical-device test of import, preview, timeline thumbnails, playback, and reopening the project. Do not rewrite the picker before this test identifies a reproducible defect.

### Native engine and export

- `engine/Engine.kt` defines dedicated coroutine dispatchers for serialized edit commands, preview rendering and thumbnail work.
- `engine/ExportService.kt` keeps long-running exports alive with a foreground service and notification.
- The Android manifest declares media, microphone, network and foreground-service permissions relevant to these features.
- The build scripts prepare Android FFmpeg and sherpa-onnx dependencies.

## 4. Build and CI audit

The workflow is `.github/workflows/release.yml`.

Verified workflow run:
- Run ID: `45564707598`
- Name: `THEO VIDEO STUDIO - Android APK`
- Trigger: manual workflow dispatch
- Commit: `637346f5b9ef8ac76e64e5a8f9081942f845adfe`
- Result: success
- APK build and signature verification steps: success
- Signing: no release keystore was configured; the workflow warning says the manual build uses the debug signing key.

**Release warning:** This artifact is suitable for testing, not a production release. A production release requires a securely configured release keystore and verified signing process. Never commit signing credentials or API keys.

The Android build requires JDK 21, pinned Rust toolchain 1.93.1, Android NDK 29.0.14206865, cargo-ndk 4.1.2, CMake/libclang, Android FFmpeg and sherpa-onnx runtime files. Keep these versions aligned with the existing workflow.

## 5. OpenReel reference-project compatibility

OpenReel is a separate browser-first editor:
- Repository: `Augani/openreel-video`
- License: MIT
- Stack: React 19, TypeScript, WebCodecs, WebGPU, Vite and browser-side media packages
- Its core package contains timeline, video effects, color grading, transitions, audio and media processing modules.

**Compatibility decision:** Do not embed the OpenReel React application in the native editor and do not attempt to compile browser-only WebGPU/WebCodecs code as Android native code. Evaluate individual algorithms, data structures, UX ideas and test cases, then implement native equivalents in Kotlin/Rust only where there is a clear benefit.

When copying or adapting MIT-licensed portions, preserve the required copyright and license notices. The Android project itself is AGPL-3.0; review all third-party notices and dependency licenses before distribution.

## 6. Initial risk register

| Risk | Priority | Required action |
|---|---|---|
| Gallery import may fail for some content providers or large files | High | Test several local MP4 files, including a longer/high-resolution file; inspect logcat if it fails |
| Preview, thumbnail and export may behave differently across Android devices/codecs | High | Test on the user's arm64 phone and record codec, resolution and duration |
| Debug-signed APK is not a production release | High | Keep test builds separate; configure release signing only for a release |
| Android permissions vary by OS version and selected-media access | Medium | Test Android 14+ selected photos/videos and denied-permission fallback |
| Browser-specific OpenReel modules are not directly portable | High | Port only isolated logic with native equivalents |
| AGPL and third-party license obligations | High | Keep notices and source/license obligations in every distributed build |
| Large native builds may be expensive on a phone-only workflow | Medium | Prefer GitHub Actions for repeatable APK builds |

No security vulnerability is asserted by this checklist alone. It identifies areas requiring tests and code review.

## 7. Phase 1 device acceptance checklist

Use a short, non-sensitive test video (about 10–15 seconds) stored locally on the Android device.

- [ ] Open THEO VIDEO STUDIO.
- [ ] Create a new project.
- [ ] Open Gallery / media picker.
- [ ] Select one local video and import it.
- [ ] Confirm the editor preview displays the correct video.
- [ ] Confirm the main timeline clip and thumbnails appear.
- [ ] Play, pause and scrub the clip.
- [ ] Close and reopen the project; confirm the imported media is still available.
- [ ] Try a second video only after the first succeeds.
- [ ] If anything fails, capture the exact screen and note which checklist step failed.

Do not share private videos. A screenshot of the app UI and a description of the failing step are sufficient for the next diagnostic pass.

## 8. Development sequence after device acceptance

1. Fix only reproducible import/preview/timeline defects found in Phase 1.
2. Add regression tests around the affected Kotlin/Rust path.
3. Validate trim, split, undo/redo, project persistence and audio playback.
4. Validate export presets, progress notification, output file and playback in another app.
5. Select OpenReel reference modules for targeted native adaptation (timeline/keyframe model, grading/effect math, audio processing), with tests and license notices.
6. Add AI-assisted features as separately scoped modules; do not label a feature AI-powered until its model/runtime, offline/online behavior, privacy, performance and tests are implemented.

## 9. Change-control rules

- Keep `main` stable; all audit and feature work happens on a feature branch and is reviewed before merge.
- Do not change package identity, signing configuration, permissions, native dependencies or export pipeline without a specific requirement and a successful build/test.
- Do not add cloud uploads or analytics to local media workflows without an explicit product decision.
- Each feature must have: a clear user-facing behavior, implementation notes, test coverage, device acceptance steps and license review where applicable.
