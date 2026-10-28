# Local macOS build

Build Rubidium in Release on the Apple Silicon laptop using the sibling Sneeze checkout.

- [x] Install missing build prerequisites and prepare universal Rust targets.
- [x] Build Sneeze dependencies and its Release library.
- [x] Restore executable permissions on the macOS build script and its helper.
- [x] Build Rubidium dependencies and its Release app.
- [x] Validate the app bundle and verify startup from process state and logs.

Use the existing `scripts/build-macos.sh --all` scripts in Sneeze, then Rubidium.
Build output and dependency sources remain in their ignored directories.

Prerequisites: installed Homebrew `molten-vk` 1.4.2 and `nasm` 3.02. Rust's
`aarch64-apple-darwin` and `x86_64-apple-darwin` targets were already installed.
Builds use `CMAKE_BUILD_PARALLEL_LEVEL=12 CARGO_BUILD_JOBS=12`.

GitHub issue creation was attempted, but `advatar/Rubidium` has issues disabled
(GitHub returned HTTP 410). This file tracks the local build instead.

Sneeze built after correcting OpenXR's system JsonCpp selection and disabling
host Vulkan discovery for Apple OpenXR/Vox builds in the sibling repository.
Wasm, Chrono, SPIR-V, UI, compute, Vox, and console tests passed. The JWS suite
aborted on malformed input; the storage suite crashed at startup. glTF reported
167 passed and 2 failed checks because sample GLB files were missing. These
failures are recorded separately from the successful build; this was not a
fully passing test run.

Validated on 2026-10-28:

- `CMAKE_BUILD_PARALLEL_LEVEL=12 CARGO_BUILD_JOBS=12 ./scripts/build-macos.sh --all`
  completed successfully after building Sneeze in the sibling checkout.
- `Rubidium.app/Contents/MacOS/Rubidium` contains arm64 and x86_64 slices.
- `codesign --verify --deep --strict` and `plutil -lint` passed for the app.
- The app launched and remained running. The launch log reported the Metal
  backend, selected Apple M5 Max, renderer initialization, WASM compilation and
  instantiation, and glTF scene uploads.
- Visual inspection was unavailable because the computer-use native service
  failed to start, including after a reset/retry.

App: `builds/macos-universal/install/release/bin/Rubidium.app`.
Build/startup logs: `builds/macos-universal/logs/`.
Sneeze logs: `../Sneeze/builds/macos-universal/logs/`.

This is a local Mac build: the bundled Homebrew MoltenVK is arm64-only, although
the main executable is universal. A distributable Intel-compatible bundle still
requires the documented universal MoltenVK build. The updater also logged a
release-notes manifest fetch warning during startup.
