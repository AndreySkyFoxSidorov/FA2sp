# FinalAlert 2 workspace rules

## Repository layout

- `FA2sp/`, `FA2pp/`, `MFC42/`, and `FA2sp.sln` are the legacy FA2sp extension for the vanilla FinalAlert 2 1.02 executable.
- `FinalAlert2YR-src/` is the separate official-source editor with the in-process FinalAlert MCP server.
- The runnable MCP-enabled editor package is `build/FinalAlert2MCP/FinalAlert2YR.exe`.
- Never inject `FA2sp.dll` with Syringe into the rebuilt editor. FA2sp relies on hard-coded addresses from vanilla 1.02 and is not binary-compatible with the rebuilt executable.
- Keep the legacy FA2sp output and `build/FinalAlert2MCP/` separate.

## FinalAlert MCP connection

- This trusted project configures the `finalalert2` Streamable HTTP MCP server in `.codex/config.toml`.
- MCP endpoint: `http://127.0.0.1:7462/mcp`.
- Health endpoint: `http://127.0.0.1:7462/health`.
- The server listens only while `build/FinalAlert2MCP/FinalAlert2YR.exe` is running. If it is unavailable, start that executable with `build/FinalAlert2MCP` as its working directory, wait for `/health` to return `status: ok`, and then retry MCP discovery.
- Keep `BindAddress=127.0.0.1` in `build/FinalAlert2MCP/FinalAlertMCP.ini`. Never expose port 7462 directly to a LAN or the Internet.
- Close modal editor dialogs before MCP calls. If a tool returns `editor_busy`, close the dialog and retry the same operation with the same `operationId`.
- The full protocol, tool catalog, semantic-image palette, and examples are documented in `build/FinalAlert2MCP/README-MCP-RU.md`.

## MCP map workflow

1. Start with `finalalert.get_status`, then inspect the loaded map with `finalalert.get_map_info`.
2. Query `finalalert.list_catalog` before choosing tiles, overlays, houses, units, buildings, infantry, aircraft, or terrain objects. Do not invent type IDs.
3. For image-driven generation, use `finalalert.import_semantic_image`. Treat the BMP/PNG/JPEG as a semantic mask, not as a literal texture. Follow the exact palette and classifier rules in `README-MCP-RU.md`.
4. Prefer `finalalert.apply_map_plan` for coordinated batches. Use `set_cells` or `paint_region` for precise terrain and height corrections.
5. Place resources, waypoints, player spawns, and catalog-validated objects only after the base terrain is stable.
6. Run the full RA2/YR validation checklist below; `finalalert.validate_map` by itself is not sufficient.
7. Generate and inspect the preview with `finalalert.get_preview`, then save with `includePreview: true`.
8. Save deliverable multiplayer maps directly under `E:\Games\Red Alert 2\` with a lowercase filename containing no spaces. For RA2 multiplayer maps, use a filename matching `[a-z0-9_-]+\.mpr`.
9. Save first under a new filename. If replacing an existing map, create a recoverable backup whose extension is not `.mpr`, `.map`, or `.yrm`, so the game does not scan the backup.

## Mutation and recovery rules

- Every mutating tool call must contain a non-empty, unique `operationId`; use a UUID.
- Reuse an `operationId` only when retrying the exact same request after a timeout or connection error. Never reuse it for different arguments.
- A client timeout does not prove that the editor failed to mutate the map. Inspect status/state before retrying, then retry with the original `operationId`.
- Terrain undo/redo covers tiles, heights, and overlays only. It does not cover objects, waypoints, or arbitrary INI changes.
- Use absolute paths for map and semantic-image operations. Put disposable validation maps outside the game install and do not overwrite shipped game data.

## Map compatibility

- Set `targetGame` explicitly to `ra2` or `yr` when creating/importing a map.
- Do not use the YR-only theaters `NEWURBAN`, `LUNAR`, or `DESERT` for Red Alert 2 maps.
- Internal map coordinates are isometric cell coordinates, not raw preview pixels. Read map/cell data before applying coordinate-sensitive edits.
- Validate player count, reachable spawns, buildable ground, resources, and symmetry for multiplayer maps before saving.
- For vanilla RA2 multiplayer maps, set `GameMode` to the complete supported mode list unless the user explicitly requests a narrower set: `standard, meatgrind, navalwar, nukewar, airwar, megawealth, duel, cooperative, teamgame`.
- `Basic.HomeCell` and `Basic.AltHomeCell` must reference existing waypoints. The MCP semantic-image and player-spawn paths may remove waypoints `98` and `99`; restore them before saving. For a newly generated map, use the editor defaults at the map center unless the map design requires another valid playable location.

## Mandatory final map validation

Do not describe a map as validated for RA2 merely because `finalalert.validate_map` reports zero issues. Before delivering or installing every generated or edited RA2 map, complete all of the following checks:

1. Run `finalalert.get_status`, `finalalert.get_map_info`, `finalalert.validate_map`, and `finalalert.get_preview`.
2. Confirm `targetGame=ra2`, an RA2-compatible theater, the requested map/player sizes, multiplayer metadata, and continuous player waypoints `0..N-1`.
3. Read `Basic`, `Map`, `Header`, and `Waypoints`. Confirm every referenced waypoint exists, especially `HomeCell=98` and `AltHomeCell=99`, and confirm `Header.NumberStartingPoints` and header waypoint coordinates agree with the player spawns.
4. Confirm every spawn is playable, free of resource overlays and blocking objects, and surrounded by sufficient buildable ground.
5. Prove that every spawn can reach the intended shared land/center using passable cells; do not infer reachability from the preview alone.
6. Check resource totals and per-player distribution. Report both the total and material asymmetry between starts.
7. Validate every used tile and subtile against the target RA2 theater catalog, reject YR-only tile ranges/content, validate overlay IDs and overlay-data values, and confirm no YR-only objects or theaters are present.
8. Validate the saved packed sections, not only the live editor state: `IsoMapPack5` must decode into unique valid cell records; `OverlayPack` and `OverlayDataPack` must each decode to exactly 262144 bytes; `PreviewPack` must decode to exactly `previewWidth * previewHeight * 3` bytes; numeric pack line keys and chunk boundaries must be contiguous and valid.
9. Save with `includePreview: true`, reopen the exact saved file through MCP, rerun validation, and render the preview again. Confirm the installed file hash matches the validated candidate.
10. When a local RA2 runtime is available, perform a real load test or obtain equivalent crash-log evidence. Check for new `GAME.EXE` exceptions in the Windows Application log and project diagnostics. If runtime loading was not performed, state that explicitly and do not claim runtime validation.

The required deliverable path is `E:\Games\Red Alert 2\`. Deliverable filenames must be lowercase and contain no spaces. Preview generation and embedding are mandatory. Keep disposable validation files outside the game installation.

## Build and verification

- Build the MCP editor from `FinalAlert2YR-src/MissionEditor.sln` with configuration `FinalAlertRelease YR|Win32` and the v143 x86 toolchain.
- Preserve the full runtime package, not only `FinalAlert2YR.exe`; it also needs its INI/data files and x86 dependency DLLs.
- After MCP/server changes, rebuild and run:
  `powershell -ExecutionPolicy Bypass -File build/FinalAlert2MCP/Test-FinalAlertMcp.ps1`
- The smoke test is read-only. For mutation tests, create and save a disposable map under a dedicated temporary directory, then verify the saved file and preview.
