# THEO VIDEO STUDIO

THEO VIDEO STUDIO is the branded Android build of this repository's CapCraft-based editing engine.

## Product identity

- App label: THEO VIDEO STUDIO
- Share-sheet label: Edit in THEO VIDEO STUDIO
- Android project name: THEO-VIDEO-STUDIO
- UI languages: Hindi and English (localization work is tracked separately)
- Base engine: Kotlin + Jetpack Compose Android UI with a Rust media engine
- Minimum Android version: Android 10 (API 29)
- Current build ABI: arm64-v8a
- License inherited from the upstream project: AGPL-3.0-or-later

## Important package identity decision

The current application ID and Kotlin namespace remain `io.github.artcraftandroid.capcraft`. Do not change the application ID casually: Android and Google Play use it as the app's unique identity, and changing it creates a separate app rather than an update. A later package migration must be planned as a new app identity.

## Build configuration

The Android Gradle project lives at:

`src/crates/capcraft-android/android`

The Rust workspace lives at:

`src`

The Android build requires the pinned Rust toolchain in `src/rust-toolchain.toml`, JDK 21 in CI, Android SDK, NDK `29.0.14206865`, `cargo-ndk` 4.1.2, CMake, and libclang. The native engine also needs FFmpeg and sherpa-onnx prepared by the repository scripts.

The CI build follows this sequence:

1. Install the pinned Rust toolchain and Android target.
2. Install cargo-ndk and Android NDK.
3. Build/cache FFmpeg for `aarch64-linux-android`.
4. Fetch sherpa-onnx runtime libraries.
5. Source `src/scripts/mobile-env.sh aarch64-linux-android`.
6. Run `./gradlew assembleRelease -Pcapcraft.abis=arm64-v8a`.
7. Verify and upload the APK artifact.

## Build an APK with GitHub Actions

The workflow is `.github/workflows/release.yml`.

- Pushes to `feature/theo-video-studio-branding` start a build.
- After this branch is merged into the repository's default branch, a manual `workflow_dispatch` run is available from Actions and builds the version in Android `gradle.properties`.
- Successful runs upload an artifact named `theo-video-studio-apk`.
- Version-tag builds require a configured release keystore and create a draft GitHub release.
- Manual branch builds without release secrets use the debug signing key. They are for testing, not public release.

### Download the APK on Android

1. Open this repository on GitHub and tap **Actions**.
2. Open the latest **THEO VIDEO STUDIO - Android APK** run.
3. Wait for the APK job to finish successfully.
4. In **Artifacts**, download `theo-video-studio-apk`.
5. Extract the downloaded ZIP and install the APK.

## Signing secrets for public releases

Configure these repository Actions secrets before creating a version tag:

- `CAPCRAFT_KEYSTORE_BASE64`
- `CAPCRAFT_KEYSTORE_PASSWORD`
- `CAPCRAFT_KEY_ALIAS`
- `CAPCRAFT_KEY_PASSWORD`

Never commit a keystore, passwords, API keys, or signing credentials to the repository.

## Current scope

The upstream editing engine already contains a substantial set of editing features, including a timeline, clip operations, effects, transitions, captions, audio tools and export options. This branding commit does not claim that new AI generative features have been implemented. Any additional AI tools must be built and tested as separate features.
