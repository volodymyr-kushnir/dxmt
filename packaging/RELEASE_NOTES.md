## Why this fork

A small number of Direct3D 11 fixes on top of [**3Shain/dxmt**](https://github.com/3Shain/dxmt), made while getting **Need for Speed Rivals** to run on macOS through Wine.

<sub>ℹ️ Everything else is the exact upstream at `@BASE@` — there are no game-specific hacks, no title detection and no behaviour switched on an executable name. Included are general D3D11 correctness fixes that happen to be what that game needed; other titles should be unaffected or better off. _Upstream is the better choice for anything other than the bugs these commits fix._</sub>

## Downloads

<table>
  <thead>
    <tr>
      <th>Asset</th>
      <th>Usage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>dxmt-@VERSION@-wine-builtin.zip</code></td>
      <td>Running Windows titles under Wine. Everything installs into the Wine build's <code>lib/wine/</code>, so every <em>new</em> prefix<sup title="A Wine prefix is a self-contained directory holding a virtual C: drive, the registry and per-application settings — Wine's equivalent of a Windows installation. Selected with $WINEPREFIX; defaults to ~/.wine.">?</sup> on that Wine gets DXMT.</td>
    </tr>
    <tr>
      <td><code>dxmt-@VERSION@-wine-native.zip</code></td>
      <td>Same, but the D3D DLLs install into one specific prefix's <code>system32</code>/<code>syswow64</code> and apply there alone. <code>winemetal</code> still goes to the Wine build, so that part remains shared.</td>
    </tr>
  </tbody>
</table>

<sub>ℹ️ The two `dxmt-@VERSION@-wine-…` zips are _not interchangeable_ — the D3D DLLs in the builtin zip are post-processed with `winebuild --builtin` and are built for loading with `=b`; the ones in the native zip are plain PE (portable executables) and are built for loading with `=n`.</sub>

## Contents

Both zips carry the same components, laid out differently: the builtin zip mirrors Wine's `lib/wine/` tree and includes ARM64EC, while the native zip places the D3D DLLs under `system32`/`syswow64` and covers x86_64 and x86 only. Both include `LICENSE` and `COPYING.LIB`.

<table>
  <thead>
    <tr>
      <th>File</th>
      <th>Description</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>d3d10core.dll</code></td>
      <td>Direct3D 10 core. Thin, forwards into <code>d3d11.dll</code>.</td>
      <td>Older titles, and D3D11 titles that touch D3D10 interop.</td>
    </tr>
    <tr>
      <td><code>d3d11.dll</code></td>
      <td>The Direct3D 11 implementation. Also contains the D3D10 device code.</td>
      <td>The main event — the DLL titles call into.</td>
    </tr>
    <tr>
      <td><code>dxgi.dll</code></td>
      <td>DXGI — adapter enumeration, swapchains, present, fullscreen transitions.</td>
      <td>A title that reaches Wine's DXGI instead of this one will not get a Metal swapchain. Always install alongside <code>d3d11.dll</code>.</td>
    </tr>
    <tr>
      <td><code>nvapi64.dll</code></td>
      <td>NVIDIA NVAPI shim (64-bit only).</td>
      <td><em>Optional.</em> For titles that probe NVAPI and misbehave in its absence; inert otherwise.</td>
    </tr>
    <tr>
      <td><code>nvngx.dll</code></td>
      <td>DLSS/NGX entry-point shim (64-bit only).</td>
      <td><em>Optional.</em> Satisfies presence checks; does not implement DLSS.</td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td>The PE half of DXMT's Wine unixlib pair. Marshals calls across the PE↔Unix boundary.</td>
      <td>Never called directly by a title; <code>d3d11.dll</code> loads it.</td>
    </tr>
    <tr>
      <td><code>winemetal.so</code></td>
      <td>The Unix half — a Mach-O library that talks to Metal.</td>
      <td>The only component that touches the GPU. Nothing renders without it.</td>
    </tr>
  </tbody>
</table>

<sub>ℹ️ `winemetal.so` is 64-bit only, by design. The 32-bit DLLs reach it through WoW64 — `meson` does not build a 32-bit unixlib at all (`src/winemetal/meson.build:60` gates it on `x86_64`/`aarch64`).</sub>

## Requirements

A system with **macOS 14.4 (Sonoma) or later**, **preferably an Apple-silicon GPU**, and **a patched Wine build** (e.g. [3Shain/wine](https://github.com/3Shain/wine), CrossOver's `wine-crossover`).

<sub>ℹ️ Every shader DXMT generates declares a minimum OS of 14.4 in its metallib header, and Metal refuses to load a metallib newer than the running system — so 14.0–14.3 fail at shader load, not at startup. On macOS 15+ DXMT targets Metal 3.2; below that — Metal 3.1. Non-Apple GPUs are attempted — DXMT logs `Experimental non-Apple GPU support` and drops to Metal 3.1. Unpatched Wine and Wine Staging have `winemac.so` that does not feature the symbols that DXMT hooks for Metal view creation and will yield `Failed to create metal view, it seems like your Wine has no exported symbols needed by DXMT.`.</sub>

## Backup

It is recommended to make copies of the files that will be overwritten during the installation.

## Installation

<details>
<summary><b>📦 Installation — Wine, builtin DLLs (Wine-wide)</b></summary>

<h4>Copying <sup>(overlaying)</sup></h4>

Zip's directory names already mirror Wine's own layout, so installation is a straight overlay onto the Wine installation's `lib/wine/`, plus a copy of `winemetal.dll` into each prefix:

<table>
  <thead>
    <tr>
      <th>Folder</th>
      <th>Files</th>
      <th>Destination</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>aarch64-unix/</code></td>
      <td><code>winemetal.so</code></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/aarch64-unix/</code></td>
    </tr>
    <tr>
      <td rowspan="2"><code>aarch64-windows/</code></td>
      <td><details><summary>All 6 files</summary><code>d3d11.dll</code>, <code>d3d10core.dll</code>, <code>dxgi.dll</code>, <code>winemetal.dll</code>, <code>nvapi64.dll</code>, <code>nvngx.dll</code></details></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/aarch64-windows/</code></td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/system32/winemetal.dll</code></td>
    </tr>
    <tr>
      <td rowspan="2"><code>i386-windows/</code></td>
      <td><details><summary>All 4 files</summary><code>d3d11.dll</code>, <code>d3d10core.dll</code>, <code>dxgi.dll</code>, <code>winemetal.dll</code></details></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/i386-windows/</code></td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/syswow64/winemetal.dll</code></td>
    </tr>
    <tr>
      <td><code>x86_64-unix/</code></td>
      <td><code>winemetal.so</code></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/x86_64-unix/</code></td>
    </tr>
    <tr>
      <td rowspan="2"><code>x86_64-windows/</code></td>
      <td><details><summary>All 6 files</summary><code>d3d11.dll</code>, <code>d3d10core.dll</code>, <code>dxgi.dll</code>, <code>winemetal.dll</code>, <code>nvapi64.dll</code>, <code>nvngx.dll</code></details></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/x86_64-windows/</code></td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/system32/winemetal.dll</code></td>
    </tr>
  </tbody>
</table>

<sub>⚠️ `x86_64-unix/` is required for 32-bit titles as well as 64-bit ones — the 32-bit DLLs reach Metal through the 64-bit unixlib. `aarch64-unix/` serves ARM64EC the same way.</sub>

<sub>ℹ️ A Wine build only carries the architectures it supports — a CrossOver tree may have `x86_64-windows`, `i386-windows` and `x86_64-unix`, and no `aarch64-*` at all. Only directories that already exist must be overwritten. The matching `*-unix/` directory is never optional and should always be present.</sub>

<h4>Updating</h4>

A prefix created before the overlay keeps its own copies of the builtins under `drive_c/windows/` — replacing the files in `lib/wine/` does not refresh them on its own — and therefore each affected prefix must be explicitly updated:

```sh
WINEPREFIX=/path/to/prefix wineboot -u
```

<sub>ℹ️ `wineboot -u` is documented only as _"Update the WINEPREFIX"_, so it's best to confirm the outcome rather than assume it — the version check under <strong>Troubleshooting</strong> reports which build a prefix is actually carrying. Where it has not taken effect, the DLLs from `lib/wine/` must be copied into that prefix's `system32`/`syswow64` by hand.</sub>

#### Running

```sh
WINEDLLOVERRIDES="d3d11,d3d10core,dxgi=b" wine Game.exe
```

<sub>ℹ️ Overwriting the files in `lib/wine/` makes DXMT the _builtin_, so a clean prefix loads it without any further configuration. However, the override matters if a _native_ (non-builtin) DLL of the same name is also present (e.g. in the prefix's `system32`/`syswow64`, or beside the executable), as Wine may resolve to that one instead, silently and with no error. Setting the override explicitly costs nothing and makes the configuration self-describing. `nvapi64` and `nvngx` belong in that list only if they were installed and a title requires them. `winemetal` needs no override at all — being builtin-only, nothing native can compete with it.</sub>

</details>

------

<details>
<summary><b>📦 Installation — Wine, native DLLs (per-prefix D3D)</b><br /><sub>Appropriate where DXMT should apply to a single prefix while Wine's own D3D stays in use everywhere else.</sub></summary>

<h4>Copying <sup>(overlaying)</sup></h4>

<table>
  <thead>
    <tr>
      <th>Folder</th>
      <th>Files</th>
      <th>Destination</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><code>i386-windows/</code></td>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/i386-windows/</code></td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/syswow64/winemetal.dll</code></td>
    </tr>
    <tr>
      <td><code>system32/</code></td>
      <td><details><summary>All 4 files</summary><code>d3d11.dll</code>, <code>d3d10core.dll</code>, <code>dxgi.dll</code>, <code>nvapi64.dll</code></details></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/system32/</code></td>
    </tr>
    <tr>
      <td><code>syswow64/</code></td>
      <td><details><summary>All 3 files</summary><code>d3d11.dll</code>, <code>d3d10core.dll</code>, <code>dxgi.dll</code></details></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/syswow64/</code></td>
    </tr>
    <tr>
      <td><code>x86_64-unix/</code></td>
      <td><code>winemetal.so</code></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/x86_64-unix/</code></td>
    </tr>
    <tr>
      <td rowspan="2"><code>x86_64-windows/</code></td>
      <td><code>winemetal.dll</code>, <code>nvngx.dll</code></td>
      <td><code><strong>&lt;wine&gt;</strong>/lib/wine/x86_64-windows/</code></td>
    </tr>
    <tr>
      <td><code>winemetal.dll</code></td>
      <td><code><strong>$WINEPREFIX</strong>/drive_c/windows/system32/winemetal.dll</code></td>
    </tr>
  </tbody>
</table>

<sub>⚠️ `x86_64-unix/` is required for 32-bit titles as well as 64-bit ones — the 32-bit DLLs reach Metal through the 64-bit unixlib.</sub>

<sub>ℹ️ **Only the D3D DLLs become native. `winemetal` is always a Wine builtin** — it is half of a unixlib pair and has to sit in the Wine tree beside its `.so`. This zip therefore installs _into two locations_, and the prefix scoping applies to the D3D DLLs alone. Because `winemetal` lands in the Wine tree, that part of the installation is shared by every prefix using that Wine build.</sub>

<h4>Updating</h4>

The D3D DLLs are written straight into the prefix and replaced in place, so they never go stale. `winemetal.dll` is the exception as it is installed into both the Wine tree and the prefix, so a later DXMT update applied to `lib/wine/` leaves the prefix's copy on the previous build. Each affected prefix must be explicitly updated:

```sh
WINEPREFIX=/path/to/prefix wineboot -u
```

<sub>ℹ️ `wineboot -u` is documented only as _"Update the WINEPREFIX"_, so confirm the outcome rather than assuming it — the version check under <strong>Troubleshooting</strong> reports which build a prefix is actually carrying. Where it has not taken effect, copy `winemetal.dll` from `lib/wine/` into that prefix's `system32`/`syswow64` by hand.</sub>

#### Running

```sh
WINEDLLOVERRIDES="dxgi,d3d11,d3d10core=n,b;" wine Game.exe
```

<sub>ℹ️ `n,b` prefers the native D3D DLLs just installed and falls back to Wine's builtin if one fails to load. `winemetal` is deliberately absent — being builtin-only, it needs no override once it is present in both the Wine tree and the prefix.</sub>

</details>

------

<details>
<summary><b>🔧 Troubleshooting</b></summary>

<h4>Logging<br /><sub>ℹ️ Under Wine, DXMT logs to <code>stderr</code> by default.</sub></h4>

```sh
DXMT_LOG_LEVEL=debug          # Or "none", "error", "warn", "info"
DXMT_LOG_PATH=/some/directory # Writes app_d3d11.log, app_dxgi.log, …
DXMT_SHADER_CACHE=0           # Disables the internal shader cache
```

#### Verifying the installed version

Every binary carries the version it was built from, so a log identifies precisely which release is loaded — and the same string can be read straight off a file, which is the quickest way to confirm which copy a prefix is actually using:

```sh
# builtin install — the Wine tree
strings -a <wine>/lib/wine/x86_64-windows/d3d11.dll | grep -m1 'v0\.'

# native install, and any prefix Wine has populated
strings -a "$WINEPREFIX/drive_c/windows/system32/d3d11.dll" | grep -m1 'v0\.'
```

<sub>ℹ️ Only `d3d11.dll` carries the string; `dxgi.dll` and `winemetal.dll` do not. After a builtin install, running both commands is the direct test for the staleness described under <strong>Updating</strong> — matching output means the prefix is current, differing output means it is not.</sub>

#### Common issues

<table>
  <thead>
    <tr>
      <th>Symptom</th>
      <th>Cause</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Failed to create metal view…</code></td>
      <td>Wine is not patched. See <strong>Requirements</strong>.</td>
    </tr>
    <tr>
      <td>Installed into <code>lib/wine/</code>, nothing changed</td>
      <td>An existing prefix holds its own copies of the builtins in <code>system32</code>/<code>syswow64</code>, which updating <code>lib/wine/</code> does not refresh.</td>
    </tr>
    <tr>
      <td>Title runs, performance unchanged</td>
      <td><code>WINEDLLOVERRIDES</code> missing or misspelled, or another <code>d3d11.dll</code>/<code>d3d9.dll</code> (typically a stale DXVK) has won the search order.</td>
    </tr>
    <tr>
      <td>DLL fails to load / initialisation error</td>
      <td>Builtin zip installed but loaded as native (<code>=n</code>), or native zip loaded as builtin (<code>=b</code>) — the two are built for different load paths.</td>
    </tr>
    <tr>
      <td>64-bit title works, 32-bit does not</td>
      <td><code>x86_64-unix/winemetal.so</code> not installed — the 32-bit DLLs cannot function without it.</td>
    </tr>
    <tr>
      <td>Nothing renders at all</td>
      <td><code>winemetal.so</code> missing, or placed in the wrong <code>*-unix</code> directory.</td>
    </tr>
  </tbody>
</table>

</details>
