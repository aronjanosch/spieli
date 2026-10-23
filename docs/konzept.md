# Konzept (Stand 2026-10-23)

## Idee

Desktop-Bibliothek für Maus und Tastatur, die die Daten von ES-DE liest. Wie die Bibliothek von
Steam neben dem Big-Picture-Modus: ES-DE bleibt die Controller-Oberfläche, Spieli ist die
Desktop-Ansicht. Eigenständig nutzbar mit jedem ES-DE-Setup; Heimspiel
(github.com/aronjanosch/heimspiel) liefert Spieli mit aus und braucht dadurch keine eigene
Bibliotheksansicht.

## Warum ein eigenes Projekt

Es gibt kein Desktop-Frontend, das ES-DE-Daten liest (Recherche 2026-10-23, Details in
heimspiel `docs/research/2026-10-23-landschaft.md`):

- Pegasus liest ES-gamelists, ist aber controller-first, letztes Release Oktober 2024.
- Playnite nur Windows; Linux-Version angekündigt, nicht erschienen.
- LaunchBox nur Windows, Linux frühestens 2027.
- Retrom (Tauri-Client + Rust-Server): keine API zum Anlegen von Spielen, Start nur über
  Emulator-Profile, Server mit Postgres Pflicht. Taugt als UI-Inspiration
  (`packages/client-web/src/routes/_fullscreenLayout.tsx`).
- gamelistedit, gamelist-media-editor: Editoren, keine Bibliothek zum Spielen.

## Technik

Tauri + Rust, Web-Frontend. Linux und Windows.

## Datenquellen (ES-DE 3.5, Belege im ES-DE-Repo)

- Systeme: Bundle `resources/systems/<os>/es_systems.xml` plus `~/ES-DE/custom_systems/es_systems.xml`
  (geht vor, gleichnamige Bundle-Systeme werden ersetzt, `SystemData.cpp:1324-1340`).
- Spiele: Dateien in `<ROMs>/<system>/` mit passender Endung; Metadaten aus
  `~/ES-DE/gamelists/<system>/gamelist.xml` (Felder `MetaData.cpp:23-48`).
- Medien: `<MediaDirectory>/<system>/<typ>/<Dateistamm>.<ext>` (`FileData.cpp:347`), Typen u. a.
  covers, screenshots, titlescreens, marquees, 3dboxes, fanart, videos, manuals.
- Sammlungen: `~/ES-DE/collections/custom-<Name>.cfg`.
- Einstellungen: `~/ES-DE/settings/es_settings.xml` (ROM-Verzeichnis, Medienverzeichnis, Emulatorwahl).

## Der schwierige Teil: Starten wie ES-DE

Die Oberfläche ist überschaubar. Die Arbeit steckt darin, Spiele genau so zu starten wie ES-DE:

- `<command>` aus `es_systems.xml` mit Variablen wie `%EMULATOR_…%`, `%CORE_…%`, `%ROM%`,
  `%ROMRAW%`, `%RUNINBACKGROUND%`, `%ENABLESHORTCUTS%`.
- Emulator-Suche über `es_find_rules.xml` (systempath, staticpath, corepath).
- Alternative Emulatoren pro System (`es_settings.xml`) und pro Spiel (`<altemulator>` in der gamelist).
- `.desktop`-Dateien: ES-DE führt die `Exec`-Zeile selbst aus.
- Plattformunterschiede Linux/Windows.

Ziel: dieselbe Auflösung wie ES-DE, mit Tests gegen die Bundle-Dateien von ES-DE.

## Schreiben

Erste Version nur lesend. ES-DE liest `gamelist.xml` beim Start und schreibt geänderte Einträge
zurück; Änderungen von außen sind nur sicher, wenn ES-DE nicht läuft. Favoriten, Spielzeit und
Metadaten bearbeiten kommen später, mit Prüfung, ob ES-DE läuft.

## Name

„Spieli“: umgangssprachlich für Spielplatz.
