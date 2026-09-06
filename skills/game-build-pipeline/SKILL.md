---
name: game-build-pipeline
group: Engineering
description: >-
  Automate platform builds, asset cooking, console certification prep, and Steam or itch.io depot
  publishing. Use when automating game builds, cooking, packaging, or Steam uploads.
---

# game-build-pipeline

## Core Philosophy
Manually building a game—clicking "Build" in the Unity or Unreal editor, waiting 45 minutes on a workstation, manually zipping files, and uploading them through a web browser to Steam—is an amateur operational hazard. A game build pipeline is an automated, continuous software deployment factory. High-performance game studios automate multi-platform cross-compilation, asset cooking, console certification checks, automated smoke testing, and direct SteamPipe/itch.io depot deployment via headless CI/CD.

---

## 4-Step Automated Game Build & Deployment Pipeline

### Step 1: Headless Engine Compilation & Asset Cooking
1. **Headless Engine Invocation**:
   - Execute builds from command-line without GUI overhead:
     - *Unity*:
       ```bash
       Unity.exe -batchmode -nographics -quit -projectPath ./ -executeMethod BuildScript.PerformWindowsBuild -logFile build.log
       ```
     - *Unreal Engine*:
       ```bash
       RunUAT.bat BuildCookRun -project=Game.uproject -noP4 -platform=Win64 -clientconfig=Shipping -cook -build -stage -pak -archive
       ```
2. **Asset Cooking Optimization**:
   - Separate code compilation from asset cooking. Use persistent shared DDC (Derived Data Cache) in Unreal or Cache Server in Unity to reduce 2-hour asset cooking runs to under 8 minutes.

### Step 2: Multi-Platform Target Matrix
1. **Target Artifact Packaging**:
   - *Windows (Win64)*: Monolithic executable + packed asset bundles (`.pak` / `.pck`).
   - *macOS (Apple Silicon / Intel)*: Universal binary `.app` bundle, notarized via Apple Developer CLI (`xcrun notarytool`).
   - *Linux (Steam Deck / Proton)*: Standalone x86_64 ELF binary validated against SteamOS runtime.

### Step 3: Automated Platform Smoke Testing
1. **The Automated Headless Boot Test**:
   - Run the packaged game executable in an automated CI container:
     - Boot executable with `-autotest -benchmark -duration=30`.
     - Assert that process does not crash, loads the splash scene, initializes audio/graphics drivers, and exits cleanly with exit code 0.

### Step 4: SteamPipe & Itch.io Depot Publishing
1. **Automated Steam Publishing (`steamcmd`)**:
   - Script SteamPipe depot uploads using manifest scripts (`app_build.vdf`):
     ```bash
     steamcmd.exe +login "$STEAM_BUILD_USER" "$STEAM_BUILD_TOKEN" +run_app_build app_build.vdf +quit
     ```
   - Automatically push nightlies to internal Steam branch `internal-qa` with automated Slack notifications to testers.
2. **Itch.io Deployment (`butler`)**:
   - Push binary diffs instantly: `butler push ./dist/win64 developer/game:windows-beta`.

---

## Deliverable Format: Game CI/CD Pipeline Specification (`BUILD-PIPELINE.md`)

```markdown
# Automated Game Build & Depot Pipeline: [Game Title]

## 1. Build Runner & Infrastructure
- **CI Runner**: Self-hosted Windows 11 Enterprise worker (64GB RAM, 16-Core AMD Ryzen, NVMe SSD)
- **Build Trigger**: Git tags (`v*.*.*`) or scheduled nightly at 02:00 UTC
- **Engines Supported**: Unity 6 (6000.0) / Unreal Engine 5.4

## 2. Multi-Platform Build Targets
| Target Platform | Architecture | Output Packaging | Target Storefront |
|---|---|---|---|
| Windows | Win64 | Single Folder + `.pak` | Steam (Default branch) |
| Steam Deck / Linux| x86_64 | Native Linux + SteamOS | Steam (`steamdeck` branch) |
| macOS | Universal (ARM64) | Signed `.app` + DMG | Itch.io / Mac Store |

## 3. SteamPipe App Build Configuration (`app_build_12345.vdf`)
```vdf
"appbuild"
{
  "appid" "1234560"
  "desc" "Nightly Automated QA Build"
  "buildoutput" ".\steam_output"
  "contentroot" ".uild\Win64"
  "setlive" "qa-nightly"
  "depots"
  {
    "1234561" "depot_win64.vdf"
  }
}
```

## 4. Pipeline Execution Sequence
1. Git checkout clean tag.
2. Execute headless build script (`BuildScript.BuildWin64`).
3. Run automated 60-second headless boot smoke test.
4. If smoke test passes, execute `steamcmd` deployment to `qa-nightly`.
5. Post build artifact link and commit changelog to Slack `#game-builds`.
```

---

## Worked Example: Slashing Build Times from 90 to 14 Minutes

- **Challenge**: Studio team spent 1.5 hours waiting for daily builds on local machines; QA frequently received broken builds.
- **Solution**: Set up a dedicated self-hosted CI runner with an NVMe SSD raid array and shared shader/DDC caching. Automated build, test, and SteamPipe upload on every git push to `staging`.
- **Outcome**: Build turnaround dropped to 14 minutes; zero broken packages reached QA.

---

## Verification Checklist

- [ ] Builds execute headlessly from CLI with zero GUI dependency.
- [ ] Shared asset and shader cache enabled to optimize incremental cook times.
- [ ] Automated smoke test validates engine boots and terminates cleanly before store upload.
- [ ] SteamPipe uploads automated via `steamcmd` script without manual web intervention.
- [ ] macOS builds signed and notarized via Apple Developer tools.

---

## Anti-Patterns

- **Building on Developer Laptops**: Freezing an engineer's machine for 2 hours every time QA needs a test build.
- **Skipping Smoke Tests**: Uploading a build to Steam that crashes on launch due to a missing DLL or shader variant.
- **Hardcoding Steam Passwords**: Storing plaintext credentials in git rather than using ephemeral tokens or environment secrets.
