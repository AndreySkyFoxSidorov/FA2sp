[![Github All Releases](https://img.shields.io/github/downloads/secsome/FA2sp/total.svg?label=Downloads&style=flat-square)](https://github.com/secsome/FA2sp/releases)
[![Workflow](https://img.shields.io/github/actions/workflow/status/secsome/FA2sp/nighty.yml?label=Nighty%20Build&style=flat-square)](https://github.com/secsome/FA2sp/actions)
[![license](https://img.shields.io/github/license/secsome/FA2sp?label=License&style=flat-square)](https://www.gnu.org/licenses/agpl-3.0.en.html)

> Upstream notice: the original legacy FA2sp project is no longer maintained. This fork also includes the separate source-based editor described below.

# FA2sp

## Fork workspace

This fork keeps the legacy FA2sp extension and the source-based FinalAlert editor
in separate projects. Initialize the nested repositories with
`git submodule update --init --recursive`.

- `FA2pp/` uses the [FA2pp fork](https://github.com/AndreySkyFoxSidorov/FA2pp).
- `FinalAlert2YR-src/` uses the [mission editor fork](https://github.com/AndreySkyFoxSidorov/CNC_TS_and_RA2_Mission_Editor).
  It adds an in-process MCP server, MCP settings in the Options menu, Russian,
  Ukrainian and Spanish UI translations, and legacy game-string decoding fixes.
- `MFC42/` continues to use its existing upstream repository.

Build the source-based editor from `FinalAlert2YR-src/MissionEditor.sln` using
`FinalAlertRelease YR|Win32` and the v143 x86 toolchain. Its local runtime package
is `build/FinalAlert2MCP/`; build artifacts and generated validation maps are
excluded from version control. Never inject the legacy `FA2sp.dll` into the
rebuilt editor, because its hard-coded addresses target the original executable.

## Changes in this fork

The source-based editor adds the following features:

- **MCP map automation:** 24 tools inspect and edit maps, import semantic terrain
  masks, place objects and player starts, validate results, render previews and
  save maps. Requests run through the editor's UI thread.
- **MCP settings in the menu:** **Options > MCP settings** controls server
  startup, port, endpoint, authentication, allowlists and request limits.
  Configuration is saved atomically and applied after restarting the editor.
- **Russian, Ukrainian and Spanish UI:** select the language under
  **Options > Settings**. Translation files are shipped with the editor.
- **Readable game object names:** legacy Russian CSF strings containing
  Windows-1251 bytes are detected and repaired in memory. Native Unicode strings
  remain readable; game archives and maps are not rewritten by this repair.
  UI language and names supplied by the installed game are independent.
- **Regression checks:** translation coverage, text decoding, configuration
  persistence and a read-only MCP protocol smoke script are included.

## MCP quick start

1. Clone this workspace with its nested repositories:

   ```powershell
   git clone --recurse-submodules https://github.com/AndreySkyFoxSidorov/FA2sp.git
   cd FA2sp
   ```

2. Build `FinalAlert2YR-src/MissionEditor.sln` with **FinalAlertRelease YR**,
   **Win32** and the **v143** toolchain. The output is
   `FinalAlert2YR-src/dist/FinalAlert2YR/`. Use an installed RA2/YR game for the
   required game data and complete the editor's initial game-path setup.
   The local `build/FinalAlert2MCP/` package is not downloaded by Git; it is a
   separately assembled runtime copy. Keep all runtime DLLs, INI files and
   translation data beside the executable when copying a build.
3. Start `FinalAlert2YR.exe` from the runtime directory and leave it open.
   Close modal dialogs before sending map commands. In **Options > MCP
   settings**, enable startup and keep the listen address at `127.0.0.1`.
   Restart the editor after changing server settings.
4. In a local MCP client, add a **Streamable HTTP** server with URL
   `http://127.0.0.1:7462/mcp`. If an authentication token is configured, send
   `Authorization: Bearer <your-token>`. `/health` requires the same token.
   The checked-in `.codex/config.toml` contains this workspace's default local
   endpoint without a token.
5. With the editor running at its default endpoint, verify the connection from
   the runtime directory:

   ```powershell
   Invoke-RestMethod http://127.0.0.1:7462/health
   powershell -ExecutionPolicy Bypass -File .\Test-FinalAlertMcp.ps1
   ```

   The example assumes the default empty token. For authentication, supply the
   same Bearer header to the health request and set `FINALALERT_MCP_TOKEN` in the
   smoke-test process environment. The smoke script checks `/mcp` and does not
   modify the map.
6. Discover `tools/list`, call `finalalert.get_status`, then inspect the loaded
   map with `finalalert.get_map_info`. Query `finalalert.list_catalog` before
   choosing object or tile IDs. Use a fresh UUID `operationId` for each mutation;
   retry only the identical request with its original ID after a timeout.
7. Validate and inspect the preview before saving under a new absolute path
   with `includePreview: true`. Reopen the saved file and validate it again.
   `finalalert.validate_map` does not replace a real RA2/YR load test or checks
   for playable, reachable and balanced multiplayer starts.

See the [English MCP guide in the editor README](https://github.com/AndreySkyFoxSidorov/CNC_TS_and_RA2_Mission_Editor/blob/main/README.md#using-mcp)
for configuration defaults, example tool arguments, the full tool list,
semantic-image colors and troubleshooting. Keep the listener on loopback;
another computer cannot connect to your machine's `127.0.0.1` directly.

## Original project

...is an engine extension project launched by secsome and aimed at providing a set of new features and fixes for **FinalAlert2** based on [FA2pp](https://github.com/secsome/FA2pp) and [Syringe](https://github.com/Ares-Developers/Syringe) to allow injecting code.

While FA2sp is independent of FA2Ext by AlexB, you cannot use FA2sp with using FA2Ext.

Currently, FA2sp uses `Visual Studio 2022 (v143)` with `/std:c++20` to build the lastest versions.

Due to the limited energy of developers, starting from version 1.6.0, FA2sp will no longer provide support for any version other than Yuri's Revenge 1.01 and mods based on it. Please understand.

You can also try the other modern map editors launched by the community, for example, [World Altering Editor](https://github.com/Rampastring/WorldAlteringEditor) .

Downloads
---------

You can choose one of the following:
- [Latest stable branch build](https://github.com/secsome/FA2sp/releases/latest)
- [Latest development branch builds](https://github.com/secsome/FA2sp/releases)
- [Latest development branch nightly](https://nightly.link/secsome/FA2sp/blob/develop/.github/workflows/nightly.yml)
- Individual new feature builds (for testing) can be found in [pull requests](https://github.com/secsome/FA2sp/pulls)

Changelog
---------

You can check the full changelog [here](./CHANGELOG.md).

Document
---------

You can check the document [here](./DOCUMENT.md).

[Unexplored](./UNEXPLORED.md)
---------
