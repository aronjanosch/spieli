# Concept (as of 2026-10-23)

## Idea

A desktop library for mouse and keyboard that reads ES-DE's data. Like the Steam library next to
Big Picture mode: ES-DE stays the controller interface, Spieli is the desktop view. It works on its
own with any ES-DE setup; Heimspiel (github.com/aronjanosch/heimspiel) ships Spieli and therefore
needs no library view of its own.

## Why a separate project

No desktop frontend reads ES-DE's data (research from 2026-10-23, details in heimspiel
`docs/research/`):

- Pegasus reads ES gamelists but is controller-first; last release October 2024.
- Playnite is Windows only; a Linux version is announced but not released.
- LaunchBox is Windows only; Linux not before 2027.
- Retrom (Tauri client + Rust server): no API to create games, launching only through emulator
  profiles, requires a server with Postgres. Useful as UI inspiration
  (`packages/client-web/src/routes/_fullscreenLayout.tsx`).
- gamelistedit, gamelist-media-editor: editors, not a library to play from.

## Technology

Tauri + Rust, web frontend. Linux and Windows.

## Data sources (ES-DE 3.5, references point into the ES-DE repo)

- Systems: bundled `resources/systems/<os>/es_systems.xml` plus `~/ES-DE/custom_systems/es_systems.xml`
  (takes precedence; bundled systems with the same name are replaced, `SystemData.cpp:1324-1340`).
- Games: files in `<ROMs>/<system>/` with a matching extension; metadata from
  `~/ES-DE/gamelists/<system>/gamelist.xml` (fields in `MetaData.cpp:23-48`).
- Media: `<MediaDirectory>/<system>/<type>/<file stem>.<ext>` (`FileData.cpp:347`); types include
  covers, screenshots, titlescreens, marquees, 3dboxes, fanart, videos, manuals.
- Collections: `~/ES-DE/collections/custom-<name>.cfg`.
- Settings: `~/ES-DE/settings/es_settings.xml` (ROM directory, media directory, emulator choice).

## The hard part: launching like ES-DE

The interface is manageable. The work is in launching games exactly the way ES-DE does:

- `<command>` from `es_systems.xml` with variables such as `%EMULATOR_…%`, `%CORE_…%`, `%ROM%`,
  `%ROMRAW%`, `%RUNINBACKGROUND%`, `%ENABLESHORTCUTS%`.
- Emulator lookup through `es_find_rules.xml` (systempath, staticpath, corepath).
- Alternative emulators per system (`es_settings.xml`) and per game (`<altemulator>` in the gamelist).
- `.desktop` files: ES-DE runs the `Exec` line itself.
- Differences between Linux and Windows.

Goal: resolve commands the same way ES-DE does, tested against ES-DE's bundled files.

## Writing

The first version is read-only. ES-DE reads `gamelist.xml` at startup and writes changed entries
back; outside changes are only safe while ES-DE is not running. Favorites, playtime and metadata
editing come later, with a check that ES-DE is not running.

## Name

"Spieli": German slang for *Spielplatz*, the playground.
