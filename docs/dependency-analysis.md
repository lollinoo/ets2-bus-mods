# Dependency analysis: Preite livery for the ETS2 Coaches DLC

Status: **analysis only.** Nothing has been installed, downloaded or implemented. Implementation is blocked until you answer the [decisions](#decisions-needed-from-you).
Date: 2026-11-19. Scope: Euro Truck Simulator 2 build **1.62.3.3** (the build on the local PC) unless a line says otherwise.

How to read it: `[Sn]` is a source in [Sources](#sources), all accessed 2026-11-19. **UNVERIFIED** means a single third-party source, an outdated doc or an inference. Those items are checked read-only on the local install in milestone M4 ([Verification strategy](#verification-strategy)). "Kit" is the community Coach Livery Kit [S7]; "PJP" is Paint Job Packer [S8].

## Summary

- **The DLC** is *Euro Truck Simulator 2 - Coaches* (Steam app 3809270, released 2026-11-12, game 1.62.x). It adds 3 coaches in 5 bodies, stored in `dlc_bus.scs` [S1][S2].
- **There is no Setra in the DLC.** The reference photos show Setra coaches, so the livery must be re-fitted to the MAN Lion's Coach, Scania Touring and Volvo 9700 DD bodies [S2].
- **Windows and black trim cannot be painted** by the stock paint shop [S11]. The black glazing band of the photos cannot be reproduced exactly. The legal fallback is OEM glass plus black only on bordering paintable panels. The community overlay trick uses DLC geometry and is excluded (rule 3.1) [S22].
- **Mechanism:** a standard paint job (`accessory_paint_job_data` + mask `.tobj` + DDS) [S12]. All bus ids and paths come from community tools, not from SCS docs, so they are UNVERIFIED until M4 lists the local archive.
- **Only 3 new tools to install**, all portable (about 40-100 MB, about 5 min): texconv, resvg and the sk-zk Extractor. Node 26, PowerShell and 7-Zip are already on the PC. No Python, no GUI editor, no Blender.
- **Legal limits:** SCS forum rule 3.1 forbids DLC textures or art in shared mods, so the repo and the mod hold only our own artwork [S22]. The photos and the PREITE marks are third-party. The GitHub repo is public, so `.gitignore` keeps all of that out.
- **Distribution default:** an unpacked folder for development and `dist/preite.scs` for personal use: a stored (uncompressed) zip built with 7-Zip and named `.scs`, which SCS accepts for local mods [S13]. The original plan's `dist/*.scs` therefore needs no extra tool. The Workshop rejects a renamed zip but accepts the same file as a real `.zip` or a folder; `scs_packer` is needed only for a HashFS v2 `.scs` [S14][S20].
- **Nothing is built yet.** [Decisions needed from you](#decisions-needed-from-you) lists 10 choices with recommended defaults.

## Target DLC and bus models

| Item | Value |
|---|---|
| Name | Euro Truck Simulator 2 - Coaches (DLC of base game app 227300) [S1] |
| Steam app id | 3809270 [S1] |
| Release date | 2026-11-12, i.e. 7 days before this analysis [S1][S2] |
| Game version | 1.62 stable came out 2026-11-05 [S3]. SCS states no minimum version for the DLC; one news site lists "base game version 1.62" [S4]. Local build: 1.62.3.3 (Steam build id 26644872) |
| Archive | `dlc_bus.scs`: 429,718,656 B, 4,501 entries, HashFS v2 (header `SCS#`, version 2, hash `CITY`), mounted in `game.log.txt`. The mapping to the Coaches DLC is inferred from the exact byte match with the Steam depot size and from the Kit README [S7]. Contents **UNVERIFIED** until listed in M4 |
| Not the same DLC | `dlc_kassbohrer.scs` (name only, probably the Kassbohrer trailers; **UNVERIFIED**) |

Three licensed brands, no others [S2]. Body ids and cabin names below come from PJP's vehicle JSONs [S9][S10] and the Kit [S7]. Lengths and axles come from a third-party wiki [S5].

| Bus | Internal id (**UNVERIFIED**) | Body id | Cabin unit (**UNVERIFIED**) | Length / axles [S5] | Paint parts [S7] |
|---|---|---|---|---|---|
| MAN Lion's Coach | `man.lions_24` | `121h4` | `121h4.man.lions_24.cabin` | 12.1 m, 4x2 | body, f_bumper, r_bumper, r_panel, mirror, f_wnd_frame (6) |
| MAN Lion's Coach L | `man.lions_24` | `139h6` | `139h6.man.lions_24.cabin` | 13.9 m, 6x2*4 | same 6 |
| Scania Touring HD | `scania.touring_23` | `129h4` | `129h4.scania.touring_23.cabin` | 12.9 m, 4x2 | body, f_bumper, r_bumper, r_panel, mirror (5) |
| Scania Touring HD | `scania.touring_23` | `137h6` | `137h6.scania.touring_23.cabin` | 13.7 m, 6x2*4 | same 5 |
| Volvo 9700 DD (double-decker) | `volvo.9700_23` | `143d6` | `143d6.volvo.9700_23.cabin` | 14.3 m (third-party preview [S6]), 6x2*4. "DD 4.0 m" in PJP is the height | same 5 |

- 5 bodies, so 27 part textures if every part gets its own mask (6 + 6 + 5 + 5 + 5).
- The Preite reference vehicles are Setra S 515/516 HD and S 415. None of them exists in the DLC.
- Neoplan coaches are rumoured for later by secondary sites only. **UNVERIFIED**, out of scope.
- Multiplayer: players without the DLC see coaches as green placeholder boxes on 1.62.3.3. SCS staff call it intended [S42].

## How bus paint jobs work in the game

### Mechanism (official docs, trucks and trailers)

The wiki unit `accessory_paint_job_data` applies from ETS2 1.25 and was last edited 2022-05-05. It has no bus content, SCS modding guides stop at 1.54 [S16], and nothing official covers coach paint jobs [S12].

| Aspect | Rule |
|---|---|
| `airbrush: true` | RGB of `paint_job_mask` is blended onto `base_color`, with the **alpha** channel as blend factor. Needs a DDS with alpha |
| colormask (default) | R, G and B channels mark regions coloured by `mask_r/g/b_color`; red wins on overlap. Base plus 3 channels = 4 colours at most |
| `flipflake` | Metallic/pearlescent mode, cannot be combined with colormask. We leave it off |
| Other attributes | `paint_job_mask` (path to a `.tobj`), `base_color`, `base_color_locked`, `stock` (must be false) [S12]. From the generic accessory page: `price` (0 hides the item), `icon` (path under `/material/ui/accessory/`, no extension), `suitable_for[]`, `overrides[]` (since 1.27) [S49] |

**Our choice: airbrush.** The livery has 6 colours, which colormask cannot carry. The Kit warns that `base_color` and texture pixels render slightly differently, so the azure is painted fully opaque into the texture [S7].
The Kit's generator writes `airbrush: true` together with `flipflake: true` [S7]. We leave `flipflake` off (opaque azure is harmless either way). M4 only confirms what the vanilla coach paint jobs use.

### Bus layout (community sources only, **UNVERIFIED** on the local install)

```
def/vehicle/bus/<id>/paint_job/<stem>.sii            accessory_paint_job_data : <stem>.<id>.paint_job
                                                     suitable_for[]: "<body>.<id>.cabin", paint_job_mask = body mask
def/vehicle/bus/<id>/paint_job/accessory/<stem>.sii  simple_paint_job_data per non-body part: paint_job_mask + acc_list[]
vehicle/bus/upgrade/paintjob/<author>/<id>/<body>/<part>.tobj + .dds    (mask location is free, this is convention)
material/ui/accessory/<icon>.tobj + .dds + .mat      shop icon, 256x64
```

Sources: PJP commit "Add bus support and three coaches" (2026-11-13) [S9] and the Kit's generator [S7]. The `bus` folder name comes from tool sources, not from SCS. `simple_paint_job_data` is not on the SCS wiki and is community-observed only [S12][S7].

### Per body and per part: conflict and how it is settled

| Source | Claim |
|---|---|
| PJP JSON [S10] | `uses_accessories: false`, `alt_uv: false`: one mask per body |
| Kit, reads `dlc_bus.scs` [S7] | The body mask is `paint_job_mask` in `paint_job/<stem>.sii`. `paint_job/accessory/<stem>.sii` has one `simple_paint_job_data` per other part: f_bumper, r_bumper, r_panel, mirror (+ f_wnd_frame on MAN), each 4096x2048. The mirror carries two `acc_list` ids |

Decision: trust the Kit, since it is the standard accessory-paint mechanism. Verify in M4 by reading one vanilla `paint_job/*.sii` and its `accessory/*.sii`. The outcome sets the texture count:
- Plan A: 5 artwork body textures plus 2 shared tiny solid textures (azure, near-black for MAN f_wnd_frame): 7 DDS. This needs accessory parts to accept a small shared mask (**UNVERIFIED**). Single-source hint: the Kit itself writes blank or uniform parts as 512x256 DDS (`blank_part_size`) [S7].
- Plan B (fallback, same code path): one full-size texture per part, 27 DDS.

### Textures and tobj

**DDS format: conflict.**

| Source | Claim |
|---|---|
| PJP v2.1.3 (MIT, 2026-11-13) [S8] | DXT5, mipmaps, power-of-2 sizes, icon 256x64 |
| Kit v0.1.0 (single source) [S7] | DX10 header, uncompressed B8G8R8A8 sRGB, full mips, 4096x2048 per part. Its own comment says "what the coach mods use", i.e. community mods, not vanilla |

- **Default until settled: BC3/DXT5, legacy DX9 header, full mips** (about 10.7 MiB per 4096x2048 texture). It is the long-proven paint-mask format and carries the alpha that airbrush needs.
- Switch: BGRA8 sRGB DX10 (about 42.7 MiB per texture, 4x larger). One config flag.
- Settled by reading the header of one vanilla coach DDS (FourCC at byte 84, DXGI format at byte 128: 91 = BGRA8 sRGB, 87 = BGRA8 linear, 77/78 = BC3, 98/99 = BC7), then an in-game check of `game.log.txt`.
- BC7 is not used: it forces a DX10 header and no source says ETS2 accepts it. SCS docs give no DDS requirements for paint jobs [S12].

**tobj** is a binary wrapper around the DDS path. Layout from ConverterPIX `tobj.h` [S34] (offsets checked against the raw header):

| Offset | Content |
|---|---|
| +0 | `u32` magic `0x70B10A01` |
| +24 | `u8` type (2 = generic, 5 = cubic) |
| +26..+28 | mag / min / mip filters; +30..+32 address u, v, w |
| +33, +35, +38 | compress, anisotropy, colour space (0 = sRGB, 1 = linear) |
| 40 bytes | end of header, then `u32` path length, `u32` 0, then the absolute DDS path (leading `/`, no NUL) |

- PJP hand-writes the same fixed header [S8], so SCS Conversion Tools are not needed when the DDS already exists.
- The Kit writes the icon tobj without mip filtering [S7], while PJP uses the mask header for the icon too [S8]. **UNVERIFIED** which one vanilla uses; mask tobj files use mip filtering in both.
- In a HashFS v2 `.scs`, a tobj and its DDS files are stored as one object, so ship both together [S14][S15].
- Only the UI icon needs a `.mat` (text) [S7][S8].

### What cannot be painted

Windows and black trim are not reachable by the paint shop. PJP's author on the SCS forum, 2026-11-19: "the unpaintable parts are a limitation of how SCS made the models" [S11].
The Kit works around it with a transparent overlay accessory (`r_logo`, decal shader) built from DLC model geometry [S7]. Rule 3.1 forbids shipping that, so we do not use it ([Legal](#legal-and-clean-room-constraints)).

## Mod packaging and manifest

`manifest.sii` sits at the package root and holds one anonymous `mod_package` unit [S13]. Planned content (ids per the pipeline design):

```
SiiNunit
{
mod_package : .preite_mod
{
    package_version: "0.1.0"
    display_name: "Preite (fan-made)"
    author: "<author>"
    category[]: "paint_job"
    icon: "mod_icon.jpg"
    description_file: "mod_description.txt"
    dlc_dependencies[]: "dlc_bus"
}
}
```

| Field | Rule [S13] |
|---|---|
| `category[]` | Max 2 of 18 values are used. There is no bus category; `paint_job` fits |
| `icon` | `.jpg`, exactly **276x162** px, next to the manifest. Workshop: max 1 MB [S20] |
| `description_file` | UTF-8 `.txt` next to the manifest, max 1 MB on Workshop [S20] |
| `dlc_dependencies[]` | Since ETS2 1.26. Value = DLC archive name without `.scs`. The Mod Manager blocks activation without the DLC. `dlc_bus` is **UNVERIFIED** as the Coaches name: no SCS source maps it, the archive exists locally |
| `compatible_versions[]` | Omitted. If present and not matching, the mod cannot be used |
| `mp_mod_optional` | Not set. The Kit sets it [S7]; TruckersMP does not allow custom mods (Kit README) |
| unit names | Components are tokens of at most 12 characters [S17]: `preite_121h4` is exactly 12, `.preite_livery` would be 13, hence `.preite_mod` |

| Format | Status |
|---|---|
| Unpacked **folder** | Accepted. SCS recommends it for development [S13]. Default for dev |
| **Zip named `.scs`** | Accepted for local mods: zip archives work "with the .scs or .zip extension" [S13], and zip is still supported after 1.50 [S15]. **Default for `dist/preite.scs`**: the 7-Zip output, stored (no compression), forward-slash paths, no extra tool. The Workshop uploader rejects it [S20]. Both stored and deflated zips exist locally; whether a deflated zip loads on 1.62 is **UNVERIFIED**. M7 confirms in game that the renamed zip loads; if not, ship the same file as `.zip` |
| **`.zip`** | Accepted. Same file under the `.zip` name (fallback) |
| **HashFS v2 `.scs`** | Officially supported and preferred since 1.50, built with `scs_packer create archive.scs -root <dir>` [S14][S15]. Optional, even for the Workshop, which also accepts a real `.zip` or a folder [S20] |

**Workshop note** (out of scope unless decision 9 changes):
- Layout is `versions.sii` plus a `universal` folder or zip [S19].
- The uploader rejects a zip renamed to `.scs` and accepts only a real zip, a folder or a HashFS v2 `.scs` [S20].
- Coach tags were added in uploader 1.62.1 (2026-11-16) [S21].
- Workshop manifests omit `display_name` and `compatible_versions` [S19].

**Mod folder:** `%USERPROFILE%\Documents\Euro Truck Simulator 2\mod` [S18], which exists and is empty. Our mod adds new paths and overrides nothing, so load order only matters against mods that reuse the same ids.

## Required tools

One matrix. "Need" is mandatory / optional / not needed. Versions apply to the research date; winget lags upstream for resvg.

| Dependency | Purpose | Need | Version | License | Official URL | Install | Size | On this PC |
|---|---|---|---|---|---|---|---|---|
| ETS2 + Coaches DLC | Host game. `dlc_bus.scs` is the read-only source for ids, paths and the format check | Mandatory (owned) | 1.62.3.3 | SCS EULA, proprietary (text **UNVERIFIED**) | [Steam][S1] | none | `dlc_bus.scs` 410 MiB | yes |
| sk-zk Extractor | Lists and extracts `dlc_bus.scs` into a work dir outside the repo (M4) | **Mandatory** | tag `2026-07-29`, win-x64 zip | GPL-2.0 (never vendor its source) | [releases][S27] | portable zip, see block below (no winget package found) | 36.7 MB zip | no |
| texconv (DirectXTex) | PNG to DDS with full mips; DDS to PNG for local viewing | **Mandatory** | 2026.5.7 (tag `may2026`) | MIT | [DirectXTex][S28] | `winget install --id Microsoft.DirectXTex.Texconv -e` | 0.97 MB | no |
| resvg | Renders our own SVG art to PNG at each texture's exact size | **Mandatory** | 0.49.0 via winget (upstream 0.50.0) | Apache-2.0 OR MIT | [resvg][S30] | `winget install --id linebender.resvg -e` | 1.7 MB | no |
| VC++ 2015+ x64 runtime | Needed by texconv (its winget manifest declares it) | Mandatory (transitive) | 14.51.36247 | MS redistributable | [MS Learn][S46] | none needed | about 25 MB | yes |
| Node.js | All pipeline scripts, tobj/SII writers, validators, `node:test`. Zero npm packages | Mandatory (reused) | 26.13.1 (accept 22+) | MIT | [nodejs.org][S31] | none | 36.6 MB | yes |
| PowerShell | `tools/check-deps.ps1` bootstrap only | Mandatory (built in) | 5.1 or 7.x | OS / MIT (pwsh) | built into Windows | none | n/a | yes |
| 7-Zip | Packs `dist/preite.scs` (stored zip). Not on PATH, called by full path | Mandatory for dist only | 26.01 | GNU LGPL 2.1+ with BSD-2/3-clause parts and an unRAR restriction [S55] | [7-zip.org][S32] | none | 1.6 MB | yes |
| Official `scs_extractor` | Fallback if sk-zk fails (no path filter) | Not needed | 1.55+ build | not stated | [wiki][S33] | portable zip to `tools/bin/` | small | no |
| Official `scs_packer` | Builds a real HashFS v2 `.scs` (Workshop only; local `dist/preite.scs` is a stored zip) | Not needed (Workshop `.scs` only) | 1.55 (zip last modified 2026-05-27) | not stated | [download][S44] | portable zip to `tools/bin/` | small | no |
| Paint Job Packer | Reference only. Its `templates.zip` is a local-only UV-guide fallback | Not needed | v2.1.3 (2026-11-13) | tool MIT; templates unlicensed | [release][S8] | none | 68 MB + about 351 MB | no |
| Python 3 + Pillow | Would only run PJP or the Kit; Pillow 13.0.0 cannot write mips (`DdsImagePlugin` writes 0 mipmaps) | Not needed | 13.0.0 (Pillow) | PSF / MIT-CMU | [python.org][S50], [Pillow source][S51] | winget `Python.Python.3.13` | 29.9 MB | no (Store stub only) |
| Blender 3.6 + SCS Blender Tools + ConverterPIX | UV wireframes of the coach meshes, last resort | Not needed | 3.6.23 / 2.4.1 / n/a | GPL / GPL-2.0 / LGPL-3.0 | [Blender][S52], [BlenderTools][S53], [ConverterPIX][S54] | portable zips | about 300 MB+ | no |
| Inkscape | GUI editing of the SVG art (SVG can also be written as code) | Optional | 1.4.4 | GPL | [inkscape.org][S43] | `winget install --id Inkscape.Inkscape -e` | 170 MB | no |
| Saira Italic font | Approximating the wordmark while drawing. It is outlined in the SVG, so the build never needs a font | Optional | variable TTF | SIL OFL 1.1 | [google/fonts on GitHub][S35] | download TTF to git-ignored `tools/fonts/` | under 1 MB | no |
| texdiag, ImageMagick, Compressonator, GIMP, .NET SDK | n/a (our `src/dds.mjs` reads the DDS header) | Not needed | n/a | n/a | n/a | n/a | n/a | no |

**Run only after approval (decision 1). Nothing below has been run.**

```powershell
# 1) texconv  2) resvg   (portable winget packages)
winget install --id Microsoft.DirectXTex.Texconv -e
winget install --id linebender.resvg -e

# 3) sk-zk Extractor: portable zip into the git-ignored tools\bin\extractor\ (run from the repo root)
Invoke-WebRequest https://github.com/sk-zk/Extractor/releases/download/2026-07-29/extractor-2026-07-29-win-x64.zip -OutFile "$env:TEMP\extractor.zip"
(Get-FileHash "$env:TEMP\extractor.zip" -Algorithm SHA256).Hash   # expect D493DC7D1834854EFFE12B80F6598CD1A8D5E754B069D7321F8802F72D64B2F7
Expand-Archive -Path "$env:TEMP\extractor.zip" -DestinationPath .\tools\bin\extractor
# the exe may sit in a sub-folder of the zip: resolve it instead of assuming the root
(Get-ChildItem .\tools\bin\extractor -Recurse -Filter extractor.exe | Select-Object -First 1).FullName
```

- New installs: **3**, about 40-100 MB, about 5 minutes. The exe name `extractor.exe` is confirmed by the project file (`AssemblyName` at tag 2026-07-29); the folder depth inside the zip and the unpacked size are **UNVERIFIED** [S26][S27]. The README says a standalone executable is available on the Releases page, so no .NET 10 runtime is needed (this PC has only .NET 8).
- **Runtime: Node 26 for everything, PowerShell only for `tools/check-deps.ps1`, no Python.** Node's built-ins cover binary tobj/DDS handling, tests and BOM-free UTF-8 (Windows PowerShell 5.1 `Set-Content -Encoding UTF8` writes a BOM, and the SII page does not document the game's BOM handling, so **UNVERIFIED** [S17]). PowerShell stays for the bootstrap because it must still run when Node is missing.
- `check-deps.ps1` is read-only and installs nothing. Exit codes: 0 all mandatory OK, 1 something missing, 2 present but too old or broken, 3 script error.
- Extracted game files live **outside** the repo in `%USERPROFILE%\ets2-bus-work` (env `ETS2_WORK`).

## Local environment audit

Read-only audit on 2026-11-19 (Windows 11 x64, one NTFS drive). Nothing was installed or modified.

| Item | Result |
|---|---|
| Game | Installed in the default Steam library: `C:\Program Files (x86)\Steam\steamapps\common\Euro Truck Simulator 2`. Build **1.62.3.3**, fully installed |
| Coaches DLC | Owned. `dlc_bus.scs` detected (429,718,656 B, 4,501 entries, HashFS v2). All six archives checked (base, def, base_vehicle, dlc_bus, dlc_it, dlc_kassbohrer) are HashFS v2 |
| Free disk | about 419 GiB (450 GB) free of 930.4 GiB (999 GB) on `C:` |
| Mod folder | `%USERPROFILE%\Documents\Euro Truck Simulator 2\mod` exists, **empty**, not redirected to OneDrive. `game.log.txt` reports "Active 0 mods" |
| Dev flags | `g_developer` and `g_console` are both 0 in `config.cfg` |
| Present | git 2.55, gh 2.102 (authenticated), Node 26.13.1 + npm 11.20, winget 1.29, 7-Zip 26.01 (not on PATH), VC++ 14.51 x64 runtime, PowerShell 7.7 (Windows PowerShell 5.1 too), Windows `tar`, VS Code, Rust + MSVC build tools, .NET 8 runtimes (no SDK) |
| **Missing** | texconv, resvg, any SCS extractor, real Python (only the Store stub), ImageMagick, Compressonator, image editors (GIMP, Krita, Inkscape, Paint.NET), Blender, .NET SDK |

## Livery specification

Estimated from three low-resolution third-party reference photos, which stay local and are never committed. Letter height in the photos is only 8-18 px, so colours and shapes are estimates (blurred, lit, compressed). The livery is redrawn as vectors and no photo pixel enters any texture.

### Palette (design values)

| Element | Hex | Notes |
|---|---|---|
| Body azure | `#2E86BD` | Photo hue is stable at 202-205 deg; observed range (shaded to lit) about `#1C6DA0`-`#3D8EC3` |
| Wordmark red (also swoosh oval) | `#C8283C` | Estimate: blurred and dulled by the surrounding black or blue |
| Blue accent (stripes, underline, swoosh block) | `#2A5CAB` | Darker and bluer than the body (hue about 220) |
| Swoosh yellow-lime | `#E3EA4F` | Estimate, with a pale edge line |
| Swoosh violet arc | `#3A2E7A` | Estimate |
| Front black band | `#0E1019` | Near-black |
| Glazing | `#0A0F1A` | Glossy tinted glass; **not paintable** on the DLC buses |

Hub caps are body colour; tyres, chrome and lights are OEM.
Note: an early team-channel post gave `#2A5CB0` and `#E6EB4A`; the values above are the later design values from the spec report.

### Elements

1. **Body base:** flat azure over the whole lower body, no gradient.
2. **PREITE wordmark, both sides,** below the window line, forward of the middle door, reading normally from outside. Heavy extended forward-slanted sans capitals (about 15-20 deg) in red, with blue slanted bars, a leading group of 2-3 slanted blue bars flowing into a blue underline.
3. **Swoosh, both sides,** above the rear axles under the sill: a yellow-lime crescent, a thin violet arc, a cobalt block with a red oval. The tail points rearward.
4. **Front black band with a centred wordmark** on the front face (Setra S 515/516). The S 415 has no band; its front wordmark sits directly on body colour.
5. **Black glazing band** along the sides (Setra-shaped, with a dip at the tail).

### Mapping onto the DLC bodies

| Part | MAN 121h4 / 139h6 | Scania 129h4 / 137h6 | Volvo 143d6 (DD) |
|---|---|---|---|
| `body` | Azure. Wordmark on both sides, swoosh above the rear axle (spans the tandem on 139h6). Front band and wordmark only if the front face is on this UV. Rear plain | Same as MAN | Azure. Wordmark on the belt panel between the decks, swoosh at skirt level above the rear axles, upper deck plain |
| `f_bumper`, `r_bumper`, `r_panel`, `mirror` | solid azure | solid azure | solid azure |
| `f_wnd_frame` | solid near-black (the one paintable part that echoes the black glazing) | n/a | n/a |

Everything is a **design assumption** until M4 shows the real UV layout: **UNVERIFIED** which part holds the front and rear faces, and whether left and right sides are separate UV regions. If they share or mirror a region, text can read correctly on one side only and you decide which.

### Approximated or omitted

- **Glazing band:** OEM glass stays as is. Near-black only on paintable panels that clearly border the glass, plus MAN `f_wnd_frame`. The band follows each bus's own windows, not the Setra outline.
- **Wordmark letterforms:** an approximation (Saira Italic outline, hand-adjusted). The photos cannot resolve the exact typeface, the stripes inside letters, or the tiny tagline, which is left out. A clean logo from you would replace it.
- **Omitted by default:** the small "REGIONE CALABRIA" text, a rear wordmark, the pale tick marks near the swoosh, any SETRA badge.
- **Roof, skirt, rear:** plain azure; hatches and AC units stay OEM.

## Legal and clean-room constraints

This is a reading of public documents, not legal advice.

- **SCS forum rule 3.1:** "Posting or sharing mods that include any material/texture/art content from DLC packages in whole or in part is not allowed" [S22]. Moderators add that even modified DLC textures count and that linking to DLC assets is fine [S23][S24]. The SCS EULA text could not be loaded (**UNVERIFIED**).
- **Consequence:** the repo and the mod hold only our own textures plus defs that reference DLC assets, and `dlc_dependencies[]` declares the DLC. No extracted or edited DLC texture, model, material, UV guide or template goes in either place. We do not override DLC base textures. Extractions stay outside the repo.
- **Templates and tools:** No official SCS paint templates were found; the paint-job wiki page mentions none [S12]. Community template packs (Carsmaniac [S48], PJP `templates.zip` [S8]) state no licence, so they are local-only. UV guides derived locally from extracted masks stay local too. sk-zk Extractor is GPL-2.0, so we do not vendor its source. The Kit's overlay and `templates/` are not copied.
- **Real-world branding:** Preite is a real operator, and its name, logo and livery are its trade dress (trademark register search not done: **UNVERIFIED**). Italian trademark law targets use "in the course of trade" [S37], so risk rises from private use (lowest), to a free public release (takedown possible), to anything monetised (worst; SCS forbids payware too [S22]). A Workshop livery with real-company lettering was removed for a guidelines violation, reason not stated [S45]. Never add the SETRA badge: the DLC's licences cover only Scania, Volvo and MAN [S2]. Use wording like "Unofficial fan-made project, not affiliated with or endorsed by SCS, Scania, Volvo, MAN, Setra or Preite".
- **Photos:** assume protected by copyright or the photographer's exclusive right of reproduction under Italian law (Capo V, arts. 87-92) [S38]. Never commit or publish them, use them only as private reference, never paste or trace pixels, never publish crops showing a number plate.
- **Fonts:** SIL OFL fonts such as Saira can be used for artwork and the artwork carries no OFL obligation [S36]. Prefer not committing font files; if one is committed it goes under `fonts/ofl/` with its `OFL.txt`.
- **Licence recommendation** (repo has no LICENSE yet): code under MIT; original docs and non-branded art under CC BY 4.0 or CC0 [S41]; no open licence on the Preite marks or the livery reproduction.
- **Public repo:** pushing makes content public, and removal needs a history rewrite and a GitHub Support request, with forks out of our reach [S40]. Publishing to the Workshop adds a Steam rights warranty [S39].
- **What `.gitignore` protects:** `.herdr-swarm/`; the reference photos (`/preite*.*`); game archives and model files (`*.scs *.pmd *.pmg *.pmc *.pma`); textures and raster art (`*.dds *.tobj *.png *.jpg *.jpeg *.webp *.psd *.xcf`); archives (`*.zip *.7z`); local folders `work/ dist/ templates/ tools/bin/ tools/fonts/ node_modules/`; the Preite brand art `art/brand/` (local until decision 8 is taken). A hygiene test (`test/hygiene.test.mjs`, run by `node --test`) backs it up; no CI workflow is planned.

## Proposed pipeline and repo layout

Designed now, built only after approval.

**Pipeline** (`node scripts/build.mjs`):
1. Load and validate `config/`; find `resvg` and `texconv` in `tools/bin/` or on PATH.
2. M4 `scripts/extract.mjs`, run once: Extractor lists `dlc_bus.scs`, extracts only defs and `vehicle/bus` into `ETS2_WORK`, prints ids, bodies, paint-job paths and one DDS header, and makes UV guides there.
3. Compose our SVG layers (`art/`) per body into `work/svg/`; `resvg` renders PNG at each part's exact size.
4. `texconv` writes DDS with full mips. Default profile `dxt5`: `-f BC3_UNORM -m 0 -dx9 --ignore-srgb`. Alternative `bgra8srgb`: `-f B8G8R8A8_UNORM_SRGB -m 0 -dx10 -srgb`. Flags are verified against the texconv docs [S29], game behaviour is **UNVERIFIED**.
5. Write tobj (`src/tobj.mjs`), SII and mat files (`src/defs.mjs`), manifest, description; `mod_icon.jpg` 276x162 (resvg renders a PNG, then `texconv -ft jpg` converts it).
6. Stage in `work/stage/`, then `& "$env:ProgramFiles\7-Zip\7z.exe" a -tzip -mx=0 dist/preite.scs` with explicit top-level entries (a stored zip named `.scs`, see [Mod packaging](#mod-packaging-and-manifest)).
7. Validate (below), then the manual in-game check.

Size: about 54 MiB at DXT5 or about 214 MiB at BGRA8 (Plan A); about 289 MiB or 1.1 GiB for 27 per-part textures (Plan B). The mod holds about 30 files.

**Tracked files:**
```
.gitignore  LICENSE  README.md  package.json (scripts only, no dependencies)
docs/dependency-analysis.md
config/mod.json  config/buses.json     user choices; ids, bodies, parts from M4
art/brand/symbols.svg                  own-drawn wordmark + swoosh: git-ignored under the decision 8 default (local only);
                                       tracked SVGs may only reference it with <use>, never copy its paths
art/bodies/{121h4,139h6,129h4,137h6,143d6}.svg   art/icon-shop.svg  art/icon-mod.svg
mod/mod_description.txt
src/{contract,sii,defs,tobj,dds}.mjs   scripts/{build,extract,check-log}.mjs
tools/check-deps.ps1
test/{sii,tobj,dds,package,check-log,hygiene}.test.mjs
```

**Git-ignored** (see `.gitignore`): `work/`, `dist/`, `templates/`, `tools/bin/`, `tools/fonts/`, `node_modules/`, the photos, and all texture, model, archive and raster formats listed above. `art/brand/` is already in `.gitignore` (the Preite brand art stays local until decision 8 is taken); delete that line only if you decide the brand art may be public.

**Milestones** (from the original plan; M3-M7 stay blocked until you approve):
- **M3** Toolchain bootstrap: `tools/check-deps.ps1` exits 0 when every mandatory dependency is present at an accepted version; install steps in the README.
- **M4** Base asset discovery: a script lists the buses, bodies and paint-job def paths from the local install into the git-ignored work dir.
- **M5** Artwork and textures: own vector sources render onto each body's layout and are encoded to DDS; a validator checks size, format and mips.
- **M6** Defs and packaging: `manifest.sii`, icon, description, paint-job SII per body and textures built and validated. **`dist/*.scs` stays**: the same 7-Zip stored zip, named `dist/preite.scs`, loads as a local mod [S13]. For the Workshop, the same file must be named `.zip` (or be a folder); `scs_packer` is optional [S20].
- **M7** README and in-game verification: install instructions; a log-check script exits 0 after you load the mod in game.

## Verification strategy

### Automated (`node --test`, built-ins only)

| Test | Checks |
|---|---|
| sii | Syntax: `SiiNunit` header, balanced braces, no BOM, name components at most 12 chars, no quotes or newlines in strings. Manifest: one anonymous `mod_package`, at most 2 valid categories, `dlc_dependencies` set, icon and description present [S17] |
| tobj | Magic, offsets, path length, absolute path, round trip; byte-compare with a vanilla tobj when `ETS2_WORK` is set |
| dds | `DDS ` magic, power-of-2 size matching config, full mip count, FourCC or DXGI format per profile, exact file size |
| package | Every path in SII, tobj and mat exists in the package; zip listing matches the staging tree with forward slashes; no unreferenced `.jpg` or `.txt`; icon is 276x162 and under 1 MB [S20] |
| check-log | Pass and fail fixtures for the `game.log.txt` scanner. The fixtures are synthetic: the real log contains the Windows user name |
| hygiene | `git ls-files` has no photos, DDS, tobj, scs, zip, png, jpg or psd, no file over 1 MB, and no `C:\Users\` path or user-profile name in any tracked file |

### Read-only checks on the local install (settle the UNVERIFIED items in M4)

All write only to `%USERPROFILE%\ets2-bus-work`. Extractor flags per its README [S26]; the exe name is `extractor.exe`, its folder depth inside the zip is **UNVERIFIED**.

```powershell
$g = 'C:\Program Files (x86)\Steam\steamapps\common\Euro Truck Simulator 2'; $w = "$env:USERPROFILE\ets2-bus-work"
$x = (Get-ChildItem .\tools\bin\extractor -Recurse -Filter extractor.exe | Select-Object -First 1).FullName
& $x "$g\dlc_bus.scs" --list --partial=/def/vehicle/bus | Select-String '^/?def/vehicle/bus/'   # ids, cabin names (--partial because the root may not list)
& $x "$g\dlc_bus.scs" --partial=/def/vehicle/bus,/vehicle/bus --dest "$w\dlc_bus"    # defs + textures
# then read one DDS header (fourcc at byte 84, dxgiFormat at 128, mip count at 28) and one vanilla paint_job/*.sii with its accessory/*.sii
```

| Open item | Settled by |
|---|---|
| Folder `def/vehicle/bus/<id>/`, ids, `<body>.<id>.cabin` names | The `--list` above |
| Per-part accessory files vs one mask per body | Reading vanilla `paint_job/*.sii` and `accessory/*.sii` (27 vs 5 textures) |
| `acc_list` ids, `overrides[]`, airbrush and flake flags | The same vanilla files |
| `dlc_bus` as dependency name | A `*.manifest.sii` at the archive root; the Mod Manager must allow activation in game |
| DXT5 vs BGRA8, size | Header of one vanilla coach paint-job DDS |
| tobj colour-space and filter bytes | Byte-compare with a vanilla tobj. The Extractor source writes the tobj plus a sibling `.dds` for HashFS v2 textures [S56]. The tobj bytes may be reconstructed by the library rather than stored verbatim (**UNVERIFIED**), so a byte-compare may check a rebuilt header. Else PJP's header |
| Front and rear faces, left vs right UV | UV guide made from a vanilla DDS with `texconv -ft png`, kept local |
| `base_color` colour space | In-game comparison of an opaque texture region with base-colour-only areas |
| Extractor opens `dlc_bus.scs` | The run itself; fallback is the official `scs_extractor` |

### Manual in-game (you), then the log check

1. Copy `dist/preite.scs` (or the folder) into the mod folder; start the game with `-unlimitedlog -error_overlay` [S16].
2. In the Mod Manager check name, icon and description, that the DLC requirement is met, and activate the mod.
3. In the bus upgrade shop open Paint job: "Preite" must appear only for the matching bodies. Apply it.
4. Inspect: no white stock parts, wordmark readable on both sides, swoosh over the rear axles, no seams or black fringes at distance.
5. Optionally start the game once with `-validate` to detect missing textures [S16] (same check as the console `validate` command; the console is disabled on this PC).
6. Quit, then `node scripts/check-log.mjs` reads `%USERPROFILE%\Documents\Euro Truck Simulator 2\game.log.txt` [S25]. It requires the package mounted and `Active N mods` of at least 1, and fails on any `<ERROR>` or `<WARNING>` line naming `preite` or our texture folder. Exit codes: 0 OK, 1 errors, 2 log not found.

Exact log wording for SII syntax errors and malformed DDS on 1.62 is **UNVERIFIED**; a deliberate bad-file test settles it later. Bodies you do not own can only be checked through the log, unless the dealer offers paint jobs (**UNVERIFIED**).

## Risks and open questions

| Risk | Impact | Mitigation |
|---|---|---|
| Ids, paths and unit names are community-sourced (**UNVERIFIED**) | Wrong paths, mod does not load | M4 read-only listing; ids kept in one `config/buses.json` table |
| The Extractor was never reported running on `dlc_bus.scs`. Its source does write a tobj plus a sibling `.dds` for HashFS v2 textures [S56] | M4 blocked if it fails on this archive | Official `scs_extractor` fallback (430 MB is fine) |
| DXT5 vs BGRA8 mismatch | Rejected or wrong-looking textures | One-flag profile switch; header check and in-game A/B |
| Shared small solid masks may be refused (Plan A) | 5x more textures | Plan B on the same code path |
| Black band cannot follow real windows | Look differs from the photos | Decision 3; OEM glass accepted |
| Left/right UV shared or mirrored | Wordmark readable on one side only | Check in M4; you pick the side |
| 1.60 Improved Material System, selected trucks only so far [S47] | Effect on coach paint masks **UNVERIFIED** | In-game test on the pilot body |
| Wiki guides stop at 1.54; the paint-job page dates from 2022 | Some facts stale | Treat as hints, verify locally |
| Multiplayer | TruckersMP bans custom mods (Kit README); non-owners see green boxes [S42] | Personal single-player use |
| Real-brand livery in a public repo or on the Workshop | Takedown | Decisions 8 and 9 |
| Photo resolution is poor | Approximate wordmark | Decision 7 (clean logo) |

### Source conflicts and how they are settled

| Topic | Conflict | Settlement |
|---|---|---|
| Texture format | DXT5 (PJP) vs BGRA8 (Kit) | Default DXT5; vanilla header decides (M4) |
| Parts per body | PJP `uses_accessories: false` vs Kit accessory files | Trust the Kit; verify; 7 or 27 DDS |
| Package | Plan M6 `dist/*.scs` vs toolchain advice (`.zip`, `scs_packer` needed) | [S13] accepts a zip named `.scs` for local mods: `dist/preite.scs` = 7-Zip stored zip, no `scs_packer`; the Workshop takes a real `.zip`, a folder or a HashFS v2 `.scs` [S20] |
| Zip packer | Windows `tar.exe` vs 7-Zip | 7-Zip, `tar.exe` as fallback |
| texdiag | Recommended by the image scout | Not needed, `src/dds.mjs` reads the header |
| Extract location | Possibly inside the repo | Outside, in `%USERPROFILE%\ets2-bus-work` |
| Texture count | 27 per-part vs 7 shared | 7 (Plan A) until proven wrong |
| Kit `airbrush` | Research notes disagreed | Settled: the Kit writes `airbrush: true` and `flipflake: true` [S7] |

Open questions for later: whether coach paint jobs are offered at purchase or only in the service shop; whether AI-traffic coaches should wear the livery (**UNVERIFIED** claim that coaches joined traffic); trademark status of PREITE.

## Decisions needed from you

Recommended defaults are in **bold**. Reply "defaults" to accept them all.

1. **Approve the toolchain:** 3 portable installs (texconv, resvg, sk-zk Extractor) plus reuse of Node 26, PowerShell and 7-Zip. **Approve.**
2. **Bus scope:** **pilot on one body (MAN 121h4, unless you tell me which bus you own), then all 5 bodies.**
3. **Windows and black trim are not paintable:** **accept OEM glass; black only on bordering panels and MAN `f_wnd_frame`; no overlay geometry.**
4. **Front black band and front wordmark:** **only where the front face lies on a paintable UV.**
5. **Volvo 9700 DD placement:** **wordmark on the belt between the decks, swoosh above the rear axles, upper deck plain.**
6. **Rear wordmark and "REGIONE CALABRIA":** **omitted.**
7. **Wordmark source:** **hand-drawn approximation starting from Saira Italic (OFL).** Better: you send a clean logo or close-up.
8. **Brand art in the public repo:** **keep `art/brand/` git-ignored and local until Preite gives permission.** Alternative: make the repo private.
9. **Distribution:** **personal use only, `dist/preite.scs` (a stored zip with the `.scs` extension, built with 7-Zip; the game loads it locally); no Workshop, no GitHub release.** A Workshop upload would need the same zip named `.zip` (or a folder), or a HashFS v2 `.scs` built with `scs_packer` [S20].
10. **Names and price:** **display name "Preite (fan-made)", author = your GitHub handle (`lollinoo`), price 1000, unlock 0, base colour locked, flake off.**

Not asked, settled by M4 checks: texture profile (DXT5 default), 7 vs 27 textures, the `dlc_bus` dependency name.

## Sources

All accessed 2026-11-19.

- S1 https://store.steampowered.com/app/3809270 - DLC name, release, 3 coaches, paint jobs
- S2 https://blog.scssoft.com/2026/11/coaches-release.html - release post, 3 licensed brands
- S3 https://blog.scssoft.com/2026/11/euro-truck-simulator-2-162-update.html - 1.62 stable (2026-11-05)
- S4 https://racinggames.gg/article/euro-truck-simulator-2-coaches-dlc-launches-on-steam-breaking-all-time-player-records - game 1.62 (secondary)
- S5 https://wikiwiki.jp/ets2steam/DLC/Coaches - variants, lengths (third-party)
- S6 https://www.onlineracedriver.com/2026/02/10/volvo-9700-coach-revealed-for-euro-truck-simulator-2/ - Volvo length (third-party)
- S7 https://github.com/3TW2527/ets2-coach-livery-kit - Coach Livery Kit v0.1.0 (MIT, community)
- S8 https://github.com/Carsmaniac/paintjob-packer/releases/tag/v2.1.3 - Paint Job Packer v2.1.3 (MIT)
- S9 https://github.com/Carsmaniac/paintjob-packer/commit/08d541dc376d06fc07b666b0d6c930542f906d5d - PJP bus support commit
- S10 https://github.com/Carsmaniac/paintjob-packer/tree/master/vehicles/ets/scs - PJP coach ids and bodies
- S11 https://forum.scssoft.com/viewtopic.php?p=2174252#p2174252 - PJP thread, post 2174252: unpaintable parts
- S12 https://modding.scssoft.com/wiki/Documentation/Engine/Units/accessory_paint_job_data - paint job unit
- S13 https://modding.scssoft.com/wiki/Documentation/Engine/Mod_manager - manifest, formats
- S14 https://modding.scssoft.com/wiki/Documentation/Tools/Game_Archive_Packer - HashFS v2, scs_packer
- S15 https://modding.scssoft.com/wiki/Games/ETS2/Modding_guides/1.50 - HashFS in mods, textures
- S16 https://modding.scssoft.com/wiki/Games/ETS2/Modding_guides - guide index, launch options
- S17 https://modding.scssoft.com/wiki/Documentation/Engine/Units - SII syntax
- S18 https://modding.scssoft.com/wiki/Documentation/Engine/Game_user_path - mod folder
- S19 https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Workshop_mod_structure - Workshop layout
- S20 https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Validation_errors_and_solutions_for_them - validation rules
- S21 https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Changelog - uploader 1.62.1
- S22 https://forum.scssoft.com/viewtopic.php?t=300666 - SCS forum rules, rule 3.1
- S23 https://forum.scssoft.com/viewtopic.php?p=2007629 - moderator statement, linking to DLC assets
- S24 https://forum.scssoft.com/viewtopic.php?p=1287799 - moderator statement, modified DLC textures
- S25 https://forum.scssoft.com/viewtopic.php?p=35183 - `game.log.txt` location
- S26 https://github.com/sk-zk/Extractor - Extractor README (GPL-2.0, CLI)
- S27 https://github.com/sk-zk/Extractor/releases - release 2026-07-29 (asset `extractor-2026-07-29-win-x64.zip`, used in the install block)
- S28 https://github.com/microsoft/DirectXTex - texconv (MIT)
- S29 https://github.com/microsoft/DirectXTex/wiki/Texconv - texconv flags
- S30 https://github.com/linebender/resvg - resvg
- S31 https://nodejs.org - Node.js
- S32 https://www.7-zip.org - 7-Zip
- S33 https://modding.scssoft.com/wiki/Documentation/Tools/Game_Archive_Extractor - official extractor
- S34 https://github.com/mwl4/ConverterPIX/blob/master/src/structs/tobj.h - tobj layout
- S35 https://github.com/google/fonts/tree/main/ofl/saira - Saira (OFL)
- S36 https://openfontlicense.org/ofl-faq/ - OFL FAQ
- S37 https://www.normattiva.it/uri-res/N2Ls?urn:nir:stato:decreto.legislativo:2005-02-10;30~art20 - Italian CPI art. 20
- S38 https://www.brocardi.it/legge-diritto-autore/titolo-ii/capo-v/art88.html - photographer's exclusive rights, Italian copyright law (Capo V, arts. 87-92)
- S39 https://store.steampowered.com/subscriber_agreement/ - Steam Subscriber Agreement
- S40 https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository - removing data
- S41 https://creativecommons.org/faq/ - Creative Commons FAQ
- S42 https://forum.scssoft.com/viewtopic.php?p=2173490 - convoy without the DLC
- S43 https://inkscape.org - Inkscape
- S44 https://download.eurotrucksimulator2.com/scs_packer_1_55.zip - official scs_packer
- S45 https://steamcommunity.com/sharedfiles/filedetails/?id=3838583969 - removed Workshop livery (real companies)
- S46 https://learn.microsoft.com/cpp/windows/latest-supported-vc-redist - VC++ redistributable
- S47 https://blog.scssoft.com/2026/07/under-hood-improved-material-system.html - 1.60 Improved Material System (selected trucks)
- S48 https://forum.scssoft.com/viewtopic.php?t=272386 - Carsmaniac paint-job templates
- S49 https://modding.scssoft.com/wiki/Documentation/Engine/Units/accessory_data - generic accessory attributes (price, icon, overrides)
- S50 https://www.python.org - Python (winget `Python.Python.3.13`)
- S51 https://github.com/python-pillow/Pillow/blob/13.0.0/src/PIL/DdsImagePlugin.py - Pillow 13.0.0 DDS writer (0 mipmaps)
- S52 https://download.blender.org/release/Blender3.6/ - Blender 3.6 downloads
- S53 https://github.com/SCSSoftware/BlenderTools - SCS Blender Tools
- S54 https://github.com/mwl4/ConverterPIX - ConverterPIX
- S55 https://www.7-zip.org/license.txt - 7-Zip licence
- S56 https://github.com/sk-zk/Extractor/blob/2026-07-29/Extractor/HashFsExtractor.cs - Extractor source, tobj + sibling .dds output

[S1]: https://store.steampowered.com/app/3809270
[S2]: https://blog.scssoft.com/2026/11/coaches-release.html
[S3]: https://blog.scssoft.com/2026/11/euro-truck-simulator-2-162-update.html
[S4]: https://racinggames.gg/article/euro-truck-simulator-2-coaches-dlc-launches-on-steam-breaking-all-time-player-records
[S5]: https://wikiwiki.jp/ets2steam/DLC/Coaches
[S6]: https://www.onlineracedriver.com/2026/02/10/volvo-9700-coach-revealed-for-euro-truck-simulator-2/
[S7]: https://github.com/3TW2527/ets2-coach-livery-kit
[S8]: https://github.com/Carsmaniac/paintjob-packer/releases/tag/v2.1.3
[S9]: https://github.com/Carsmaniac/paintjob-packer/commit/08d541dc376d06fc07b666b0d6c930542f906d5d
[S10]: https://github.com/Carsmaniac/paintjob-packer/tree/master/vehicles/ets/scs
[S11]: https://forum.scssoft.com/viewtopic.php?p=2174252#p2174252
[S12]: https://modding.scssoft.com/wiki/Documentation/Engine/Units/accessory_paint_job_data
[S13]: https://modding.scssoft.com/wiki/Documentation/Engine/Mod_manager
[S14]: https://modding.scssoft.com/wiki/Documentation/Tools/Game_Archive_Packer
[S15]: https://modding.scssoft.com/wiki/Games/ETS2/Modding_guides/1.50
[S16]: https://modding.scssoft.com/wiki/Games/ETS2/Modding_guides
[S17]: https://modding.scssoft.com/wiki/Documentation/Engine/Units
[S18]: https://modding.scssoft.com/wiki/Documentation/Engine/Game_user_path
[S19]: https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Workshop_mod_structure
[S20]: https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Validation_errors_and_solutions_for_them
[S21]: https://modding.scssoft.com/wiki/Documentation/Tools/SCS_Workshop_Uploader/Changelog
[S22]: https://forum.scssoft.com/viewtopic.php?t=300666
[S23]: https://forum.scssoft.com/viewtopic.php?p=2007629
[S24]: https://forum.scssoft.com/viewtopic.php?p=1287799
[S25]: https://forum.scssoft.com/viewtopic.php?p=35183
[S26]: https://github.com/sk-zk/Extractor
[S27]: https://github.com/sk-zk/Extractor/releases
[S28]: https://github.com/microsoft/DirectXTex
[S29]: https://github.com/microsoft/DirectXTex/wiki/Texconv
[S30]: https://github.com/linebender/resvg
[S31]: https://nodejs.org
[S32]: https://www.7-zip.org
[S33]: https://modding.scssoft.com/wiki/Documentation/Tools/Game_Archive_Extractor
[S34]: https://github.com/mwl4/ConverterPIX/blob/master/src/structs/tobj.h
[S35]: https://github.com/google/fonts/tree/main/ofl/saira
[S36]: https://openfontlicense.org/ofl-faq/
[S37]: https://www.normattiva.it/uri-res/N2Ls?urn:nir:stato:decreto.legislativo:2005-02-10;30~art20
[S38]: https://www.brocardi.it/legge-diritto-autore/titolo-ii/capo-v/art88.html
[S39]: https://store.steampowered.com/subscriber_agreement/
[S40]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository
[S41]: https://creativecommons.org/faq/
[S42]: https://forum.scssoft.com/viewtopic.php?p=2173490
[S43]: https://inkscape.org
[S44]: https://download.eurotrucksimulator2.com/scs_packer_1_55.zip
[S45]: https://steamcommunity.com/sharedfiles/filedetails/?id=3838583969
[S46]: https://learn.microsoft.com/cpp/windows/latest-supported-vc-redist
[S47]: https://blog.scssoft.com/2026/07/under-hood-improved-material-system.html
[S48]: https://forum.scssoft.com/viewtopic.php?t=272386
[S49]: https://modding.scssoft.com/wiki/Documentation/Engine/Units/accessory_data
[S50]: https://www.python.org
[S51]: https://github.com/python-pillow/Pillow/blob/13.0.0/src/PIL/DdsImagePlugin.py
[S52]: https://download.blender.org/release/Blender3.6/
[S53]: https://github.com/SCSSoftware/BlenderTools
[S54]: https://github.com/mwl4/ConverterPIX
[S55]: https://www.7-zip.org/license.txt
[S56]: https://github.com/sk-zk/Extractor/blob/2026-07-29/Extractor/HashFsExtractor.cs
