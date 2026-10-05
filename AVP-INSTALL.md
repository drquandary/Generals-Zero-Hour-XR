# Installing Generals: Zero Hour XR on Apple Vision Pro

This is a native visionOS port of *Command & Conquer: Generals – Zero Hour*, built on [Cesarus85's Meta Quest tabletop edition](https://github.com/Cesarus85/Generals-Zero-Hour-XR). The original engine runs natively, rendering through OpenGL ES 3 on Metal (ANGLE), and draws the battlefield in stereo as a 1.1 m square miniature standing in your room, with passthrough around it. You command it with look and pinch, and Ground View lets you walk the battlefield at human scale. It is early: the first campaign mission plays; skirmish setup, save/load and multiplayer aren't in yet.

## What you need

- Apple Vision Pro in Developer Mode, registered to your Apple Developer account and trusted by your Mac. The app targets visionOS 26.0.
- A Mac with Apple silicon and Xcode 27 with the visionOS SDK (Settings > Components)
- CMake 3.28 or newer, Ninja and pkg-config (`brew install cmake ninja pkg-config`), XcodeGen (`brew install xcodegen`), git and python3
- A full (not shallow), bootstrapped clone of [vcpkg](https://github.com/microsoft/vcpkg) at a commit that supports visionOS
- Your own installed copies of both *Command & Conquer: Generals* and *Zero Hour*

## Your game files

No game data ships with this project, in the repository or in the app. Zero Hour is an expansion, so the app needs **both** games: the Zero Hour archives (`INIZH.big` and the rest) and the base Generals archives (`Terrain.big` and the rest). [docs/GAME_DATA_SETUP.md](docs/GAME_DATA_SETUP.md) lists every required file and the folder layouts the app accepts (Steam, installed from disc, merged, or separate folders). Disc images, `.cab` files and installers aren't supported: install the games on a computer first and copy the installed folders. Expect about 2.7 GB.

Get the folders onto the headset in either of these ways:

- **Pick a folder (recommended).** Copy the installed game folders to iCloud Drive (make them available offline first), AirDrop them to the headset, or put them on an external drive. In the app, choose **Choose game folder...** and pick the folder. The app checks it, then copies it into its own storage; the copy is resumable if it's interrupted.
- **Drop folder.** In the Files app, copy the game folders into the `GameData` folder under *On My Apple Vision Pro > Generals ZH XR*. The app finds them on launch, when you return to it, or when you press **Check again**, and uses them where they are.

When the data passes the checks, the launcher shows **Ready** and unlocks **Enter Tabletop**.

## Build from source

Start from a checkout of this repository. The build keeps its large dependencies outside the repository, under `GX_DEPS_ROOT` (default `$HOME/CandC/deps`); vcpkg is expected at `$GX_DEPS_ROOT/vcpkg` unless you set `VCPKG_ROOT`. If you don't have it yet:

```sh
git clone https://github.com/microsoft/vcpkg.git "$HOME/CandC/deps/vcpkg"
"$HOME/CandC/deps/vcpkg/bootstrap-vcpkg.sh" -disableMetrics
```

`source scripts/build/visionos/env.sh` checks the build prerequisites (CMake, Ninja, pkg-config, the visionOS SDKs and vcpkg) and says what's missing.

Then, from the repository root:

```sh
# 1. ANGLE (OpenGL ES 3 on Metal) for the device, once per machine: about 10 to 15 minutes
scripts/build/visionos/build-angle.sh device      # or "all" to add the Simulator slice

# 2. A signed device build: builds the engine library first if it's missing (15+ minutes),
#    generates visionos/GeneralsZHXR.xcodeproj with XcodeGen, and prints the .app path last
DEVELOPMENT_TEAM=YOUR_TEAM_ID GX_DEVICE_ID=YOUR_HEADSET_UDID \
  scripts/build/visionos/build-shell.sh device

# 3. Install it (find the UDID with: xcrun devicectl list devices)
xcrun devicectl device install app --device YOUR_HEADSET_UDID <path printed by build-shell.sh>
```

Without `DEVELOPMENT_TEAM`, `build-shell.sh device` makes an unsigned compile check that can't be installed. `GX_DEVICE_ID` lets Xcode register the headset in your team's provisioning profile.

To use Xcode instead: after `build-shell.sh` has run once, open `visionos/GeneralsZHXR.xcodeproj`, select the `GeneralsZHXR` target, choose your own team under Signing & Capabilities, pick your Vision Pro as the run destination and press Run. The bundle identifier is `com.generalsx.zerohour.xr.vision`; if your team needs a different one, change `PRODUCT_BUNDLE_IDENTIFIER` in `visionos/project.yml`, since the Xcode project is regenerated from it. Don't commit team identifiers or provisioning files.

[docs/BUILD/VISIONOS.md](docs/BUILD/VISIONOS.md) covers the engine build, its environment variables and troubleshooting; [README_VISIONOS.md](README_VISIONOS.md) covers the Simulator and its test hooks.

## Notes

- **Controls:** look at a unit and pinch to select it; look at the ground or an enemy and pinch to move or attack. Pinch and drag on the map to box select, drag the table edge to pan, and spread both hands to zoom. The grab bar in front of the table moves, turns and resizes it. The game's own control bar lies in front of the table; a small controls strip beside it has Ground View, Recenter, Pause, Menu and common orders. The full table is in the [README](README.md#how-the-controls-work).
- Don't use `devicectl device copy to ... --remove-existing-content true` on the app's container: it removes all app data, including game data you copied.
- Known issues and next steps are in [README_VISIONOS.md](README_VISIONOS.md#possible-improvements-from-the-first-headset-sessions); [docs/VISIONOS_TEST_MATRIX.md](docs/VISIONOS_TEST_MATRIX.md) lists every feature and how it was tested.
- *Command & Conquer*, *Generals* and *Zero Hour* are trademarks of their owners. This is an unofficial community project, not made, endorsed or supported by Electronic Arts or Westwood Studios.
