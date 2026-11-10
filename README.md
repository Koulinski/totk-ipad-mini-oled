# Tears of the Kingdom 1.2.1 on iPad mini OLED

An experimental Apple silicon port built locally from your own Switch files. This guide covers the 1.2.1 OLED configuration. It uses the MacNativeGames build tools and the revision below.

This source patch is based on [MacNativeGames revision `4fdda415`](https://github.com/ly05010419/MacNativeGames/tree/4fdda41562a1c783fb1aacd18891ad4f48b8c360). It includes the OLED graphics presets, Stats, shader precompilation, controller support, and ten built-in cheats for 1.2.1. The private cheat catalog, captured motion-replay data, and device-specific shader caches stay out of the release. Performance below was measured on earlier iPad builds. See [what has been tested](#what-has-been-tested) for the scope of those results.

![Illustration of the graphics settings and the two output framings](images/ipad-mini-oled-settings.svg)

*Original framing illustration. Both modes keep the same horizontal field of view. Centered 16:9 has black bars above and below the 2316 × 1303 game picture. Full Height extends the vertical view across the 2316 × 1512 display. Game screenshots stay private.*

Keep the game, update, console keys, fonts, saves, and generated shader data on your own devices. A locally built IPA can contain rewritten game code even when the game image is transferred separately. Keep that IPA and the signed app private as well.

## Build and install

1. Prepare an Apple silicon Mac with macOS 26 or newer, Homebrew, and full Xcode 26 or newer with support for the iPad's installed iPadOS version. The earlier OLED device build used Xcode 27. The app targets iPadOS 26 or newer. Plan for roughly 80 GB free on the Mac and about 25 GB on the iPad. Keep the work folder and IPA on the same APFS volume. The game image alone takes about 17 GB. [MacNativeGames requirements](https://github.com/ly05010419/MacNativeGames/blob/main/docs/install.md)
2. Gather your uncompressed Tears of the Kingdom 1.0.0 base game as an XCI or NSP, the separate cumulative 1.2.1 update as an NSP or NSZ, and your console's `prod.keys`. Keep these outside both Git checkouts. An XCI can supply the system fonts. With an NSP base, supply your own unpacked firmware directory or extracted shared fonts.
3. Open Xcode once and finish its first-run setup, including installation of the iOS platform. Then open Terminal and install the build tools. Keep the same Xcode selected for building and signing. Adjust the example path if your Xcode is installed elsewhere.

   ```sh
   brew install opus libvpx python@3.14 xcodegen cmake ninja
   export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
   xcodebuild -version
   xcrun --sdk iphoneos --show-sdk-version
   xcodebuild -downloadComponent MetalToolchain
   xcrun --sdk iphoneos metal --version
   ```

4. Run this block together in Terminal from a folder where both repositories can be cloned as siblings. It checks out the reviewed patch commit and verifies the patch's SHA-256 before applying it to the pinned upstream revision. If a check fails, stop and use a fresh checkout from the listed GitHub URLs.

   ```sh
   git clone https://github.com/Koulinski/totk-ipad-mini-oled.git &&
   git -C totk-ipad-mini-oled switch --detach 06bb3bebad7f7dc7522219e481efff50999cfa90 &&
   printf '%s  %s\n' \
     'e8e58eeeda90dee5a700a65fe0174985425af69ba2b0ff4afc192d727f0c590d' \
     'totk-ipad-mini-oled/ipad-mini-oled.patch' | shasum -a 256 -c &&
   git clone https://github.com/ly05010419/MacNativeGames.git &&
   cd MacNativeGames &&
   git switch --detach 4fdda41562a1c783fb1aacd18891ad4f48b8c360 &&
   git apply --check ../totk-ipad-mini-oled/ipad-mini-oled.patch &&
   git apply ../totk-ipad-mini-oled/ipad-mini-oled.patch
   ```

5. In **Xcode → Settings → Accounts**, sign in with your Apple Account and choose your development team. If you need a local certificate, open **Manage Certificates**, click **+**, and choose **Apple Development**. The certificate and its private key must be available in this Mac's Keychain. The installer asks Xcode for a profile that includes both **Increased Memory Limit** and **Extended Virtual Addressing**. Use a team that can provision both capabilities. [Apple's certificate guide](https://developer.apple.com/documentation/xcode/sharing-your-teams-signing-certificates) · [Apple's Xcode signing guide](https://developer.apple.com/documentation/xcode/running-your-app-on-simulated-or-physical-devices)
6. Connect the iPad with a USB data cable, unlock it, and accept the trust prompt. Pair it in Xcode's device window. On iPad, enable **Settings → Privacy & Security → Developer Mode**, restart, and confirm the setting when prompted. Pairing makes this menu available. Run `xcrun devicectl list devices` and note the iPad's exact name. [Apple's device setup guide](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device)
7. Build the local IPA from the patched **MacNativeGames** checkout. Replace the quoted paths with your own. The first build extracts game data and builds shaders. Wait for each step to finish.

   ```sh
   base="$HOME/Games/your-base-game.xci"
   update="$HOME/Games/your-update-1.2.1.nsp"
   keys="$HOME/Games/prod.keys"
   work="$HOME/Games/TOTK-OLED-Local"
   ipa="$work/TOTK-1.2.1-local.ipa"

   python3.14 build_app.py "$base" --keys "$keys" --update "$update" \
     --platform ios --work "$work" --out "$ipa" --no-shader-reminder
   ```

   Use the actual `.nsp` or `.nsz` update filename. For an NSP base, add one of `--firmware "/path/to/your/unpacked-firmware"` or `--shared-fonts "/path/to/your/extracted-fonts"` to the build command. On a fresh build, the tool reports the update version from verified ExeFS hashes. Confirm it identifies **1.2.1**. The builder makes the Full Height overlay and shader seed locally. Check that the IPA exists at the printed output path.

8. Sign and install only the IPA you built locally in step 7. Skip prebuilt IPAs and third-party installers. Install over USB with the iPad unlocked. Before updating, quit the app and [back up its Documents](#back-up-and-switch-save-profiles). Keep your team and bundle identifier the same to retain those files.

   ```sh
   python3.14 games/totk/ios/tools/install_device.py "$ipa" --device "Your iPad name"
   ```

   Run this in the same Terminal session as the build, or replace `"$ipa"` with the full path to your IPA. If the helper lists multiple teams, rerun with `--team` and the listed 10-character team ID. It chooses a stable team-specific bundle ID by default and prints it before signing. If you choose `--bundle-id`, reuse that exact value for every update. The helper checks the memory entitlements and device membership in the profile, verifies the signature, and prints the provisioning expiration. Run the same command to renew. If iPadOS asks you to trust the developer, use **Settings → General → VPN & Device Management**.

9. Open the app and let initial data verification and shader preparation finish. Open **Settings → Apps → Tears of the Kingdom → Graphics**, choose a preset, then quit the game from the App Switcher and reopen it. For the tested 60 FPS profile, choose **Balanced (900p, Real 60 FPS)**.

## Graphics and controls

| Render preset | Scene resolution | Output on the OLED mini | Frame target |
| --- | --- | --- | --- |
| Performance (720p) | 1280 × 720 | Centered 16:9 | 30 real FPS, with optional frame generation |
| Balanced (900p) | 1600 × 900 | Centered 16:9 | 30 real FPS, with optional frame generation |
| Balanced (900p, Real 60 FPS) | 1600 × 900 | Centered 16:9 | 60 real FPS |
| Quality (1080p) | 1920 × 1080 | Centered 16:9 | 30 real FPS, with optional frame generation |
| Full Height (1600 × 1044, Real 60 FPS) | 1600 × 1044 | Full 2316 × 1512 display | 60 real FPS, experimental |

MetalFX spatial scaling is enabled for these presets. The centered presets use a 2316 × 1303 game picture inside the 2316 × 1512 display, with black bars above and below. Full Height extends the camera's vertical view while keeping the horizontal view. These are render targets. Frame rate depends on the scene, shader cache, and device conditions.

**Frame Generation (60 FPS Cap)** starts off. With a compatible 30 FPS preset, it can insert frames between real frames for up to 60 displayed FPS. The two Real 60 FPS presets turn it off. Selecting Full Height on an unrecognized device falls back to Balanced (900p, Real 60 FPS).

**Saves** has a **Use Copied Mac Saves** switch. If a separate Mac save profile has already been copied into the app, this switch selects it on the next launch. The iPad profile retains its own progress. The switch alone does not import a save.

**Game language** can follow the system or use Traditional Chinese, Simplified Chinese, English, Japanese, German, Spanish, French, Italian, Dutch, Russian, or Korean. Relaunch after changing it.

Tap the gameplay view to reveal the toolbar. The buttons hide again after about four seconds of inactivity. An open menu or active shader build keeps its controls available.

| Toolbar button | Options |
| --- | --- |
| Shaders | Start or stop **Build Shaders** and view its progress. You can hide the panel while work continues. This precompiles known recipes. New areas can still encounter new pipelines. |
| Stats | Toggle FPS, resolution and MetalFX, display frame time (P95), CPU submit time, GPU render time (P95), and app memory. The selections persist. Clear all six to hide the readout. |
| Cheats | Toggle individual 1.2.1 cheats for the current session. |

Touch controls are available. A DualSense was tested on the OLED mini, including gyro aiming. USB-C GameSir gyro still needs a device test.

## Cheats

1. [Back up the save profile](#back-up-and-switch-save-profiles) before enabling a cheat for the first time. Try changes on a copy of your save.
2. Load the save, tap the game view to reveal the toolbar, then tap **Cheats**.
3. Turn on the switches you want and tap **Done**. To stop an effect, reopen **Cheats**, turn its switch off, and tap **Done**.

All switches start off at launch and apply only to the current session. Items, rupees, or other effects already written to a save can persist after a switch is turned off.

| Switch | Effect |
| --- | --- |
| Refill Hearts | Refills the current hearts up to the existing maximum. |
| Refill Stamina | Restores the current stamina meter up to its existing maximum. |
| Refill Battery | Refills the current energy cells up to their existing maximum. |
| Refill Weapon Durability | Refills the durability of the equipped, usable weapon. |
| Refill Bow Durability | Refills the durability of the equipped, usable bow. |
| Refill Shield Durability | Refills the durability of the equipped, usable shield. |
| Keep 888 Arrows | Tops up arrows to at least 888. Higher counts stay as they are. |
| Keep 999,998 Rupees | Tops up rupees to at least 999,998. Higher counts stay as they are. |
| Moon Jump (Hold X) | Adds upward movement while holding the game's X button. |
| Item Pickup Multiplier (8x) | Makes a one-unit pickup of a stackable material add eight units. |

An earlier OLED build verified stamina refill in gameplay. A separate copied-save test verified the pickup switch added eight with the switch on and one with it off. The public version passed local switch and ARM64 tests. Broader cheat gameplay testing on the current build is still open.

## Back up and switch save profiles

- **Back up before updates or cheats.** Quit the game. In Files on iPad, open **On My iPad → Tears of the Kingdom**. Copy the `save` folder, any `save-mac-benchmark` folder, and `shader-cache` to a separate backup location.
- **Use a copied Mac profile.** With both apps closed, copy your existing Mac profile folder to the iPad app's Documents as `save-mac-benchmark`, retaining its `account.uid` and `save` subfolder. The default Mac profile path is `~/Library/Application Support/MacNativeGames/TOTK/profile`. In **Settings → Apps → Tears of the Kingdom → Saves**, enable **Use Copied Mac Saves** and relaunch. Turn the switch off and relaunch to return to the iPad profile.

## What has been tested

| Feature | Validation |
| --- | --- |
| Build and signing | The public source patch compiled for iPadOS with locally prepared inputs. Earlier builds installed on the OLED mini. A clean first build and physical-device run of the current public patch are pending. |
| Centered 16:9 and Full Height | Both displayed on the OLED mini in earlier builds. The scope and opening-film framing changes were checked on Mac and compiled for iPad. Projected world labels and the transition after cutscenes still need device checks. |
| Balanced 900p at 60 real FPS | Measured on an earlier OLED build. See [the Depths benchmark below](#measured-on-the-ipad-mini-oled). Full Height still needs a matched device benchmark. |
| Controls and audio | Earlier OLED checks covered DualSense gyro and opening cutscene audio sync. USB-C GameSir gyro remains unverified. |
| Stats and shaders | Earlier OLED builds ran shader preparation. The newer Stats panel, its persistence, and its corner clearance still need a live check. |
| Cheats | The 8× hook passed 64 native ARM64 cases. Local tests exercised all ten switches and the backup check. Earlier iPad builds verified stamina refill and pickup on and off. Testing the full public set during gameplay remains open. |

## Measured on the iPad mini OLED

The earlier 1.2.1 OLED test used the Balanced (900p, Real 60 FPS) preset, MetalFX, and frame generation off. The Depths route was walked by hand, so the samples cover different lengths. P95 is the display interval at or below which 95% of intervals fell.

| Depths route sample | Displayed FPS | Display interval P95 | Display gaps over 50 ms | Peak sampled app memory |
| --- | ---: | ---: | ---: | ---: |
| 40.6 seconds after building shaders | 59.30 | 16.67 ms | 7 | 3.25 GiB |
| 53.0 seconds on a repeat in the same session | 59.73 | 16.67 ms | 3 | 3.27 GiB |

The longest gaps were about 83 ms in the first sample and 133 ms on the repeat. Six of the first run's seven gaps and all three gaps on the repeat coincided with shader or pipeline creation. In a separate 900p comparison, the tester preferred real 60 FPS to 30 FPS with frame generation. An earlier higher-resolution real-60 test ran around 36–45 FPS in parts of its route and occasionally dropped to around 20 FPS. Full Height gameplay was checked on the OLED mini. Its sustained frame rate and memory use still need a matched device run.

## Problems found and work remaining

- **Cold shaders.** The first visit to a scene may hitch while Metal builds a new pipeline. A repeat route was smoother in the same session. Cache behavior after relaunch and across more areas still needs testing.
- **Rendering.** The reported waterfall flicker and unexpected red patches in the Depths were fixed at the tested locations. Full Height still needs checks for projected world labels, scope framing, and the transition after cutscenes. The scope and opening-film framing fixes were checked on Mac and compiled for iPad. They still need an iPad run.
- **Frame generation.** Equipment can shimmer or ghost during movement. The public patch caps generated output at 60 FPS. The motion-replay experiment remains separate and uses locally captured data. Pacing and quality during walking, jumps, attacks, and fast camera turns remain open.
- **Performance.** A separate Mac trace found a mesh decode job that used about 63 ms of CPU time. Two small decoder changes showed no measurable gain. Test more scenes on iPad, including long sessions, thermals, and memory pressure.
- **Device validation.** Opening audio sync, DualSense gyro, and Full Height gameplay were checked on the OLED mini. The newer Stats menu, its persistence, and overlay clearance around display corners still need a live check. HDR and ray-traced reflections are experiments.

## Troubleshooting

| If you see | Check |
| --- | --- |
| No iPhoneOS SDK or Metal toolchain | Open full Xcode and finish installing its iOS platform and Metal toolchain. Check `xcodebuild -version`, `xcrun --sdk iphoneos --show-sdk-version`, and `xcrun --sdk iphoneos metal --version` in the same Terminal session. |
| Patch fails to apply | Confirm `git rev-parse HEAD` matches the pinned revision and `git status --short` is empty before applying the patch. Use a fresh clone if the checkout already has changes you want to keep. |
| Module hashes or shared fonts fail verification | Check the actual base and update versions and make sure your copies are complete. For an NSP base, check the firmware or font path. The build prints the failing step and saves detailed logs under `$work/logs`. |
| No device found | Check the USB cable, trust prompt, unlock state, and Developer Mode. Confirm the iPad appears in `xcrun devicectl list devices`. |
| Signing or provisioning fails | Check Xcode Accounts, the Apple Development certificate and its private key, the team, device registration, and both required memory entitlements. The helper prints the location of its Xcode log if provisioning fails. |
| Cheats button is missing | Confirm you built the 1.2.1 update, load a save, and tap the game view to reveal the toolbar. A refused switch shows a message in the Cheats panel. |
| Build or install runs out of space | Check free space on both devices. Keep the work folder and IPA on one APFS volume so the builder can reuse data by cloning it. |
| First launch takes time or a new scene stutters | Give initial verification and shader work time to finish. Try **Shaders → Build Shaders** and compare the same route again. When reporting a problem, include the preset, area, shader-cache state, and frame generation setting. The app writes `log.txt` in its Documents. |

Before posting a log excerpt or screenshot, redact identifying paths, usernames, email addresses, device names and IDs, and Apple account or team details. Share only the lines needed to reproduce the issue. Never post `prod.keys`, other keys, passwords, access tokens, provisioning profiles, signed apps, or game files.

The patch is distributed under the MIT license with upstream notices preserved. Tears of the Kingdom and Nintendo Switch are trademarks of Nintendo. This project is independent of Nintendo.
