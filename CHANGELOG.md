# WIM

## [3.15.1](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/tree/3.15.1) (2026-02-11)
[Full Changelog](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/compare/3.15.0...3.15.1) [Previous Releases](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/releases)

- Fix keep-focus behavior after sending messages.
- Persist and restore minimap icon position; save position on drag stop.
- Restore item link insertion in whisper message boxes.
- Guard Expose minimap menu integration when the menu is unavailable.
- Initialize LDB Notes to avoid nil errors.
- Fix MinimapIcon.lua syntax error.

## [3.15.0](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/tree/3.15.0) (2026-01-25)
[Full Changelog](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/compare/3.14.6...3.15.0) [Previous Releases](https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger/releases)

- Cleanup  
- Merge pull request #206 from Amadeus-/midnight  
    Midnight Expansion Support (12.0.0)  
- Merge branch 'master' into midnight  
- Update for WoW 12.0.0 (Midnight) API compatibility  
    - Migrate IsEncounterInProgress() to C\_InstanceEncounter namespace (3 locations)  
    - Add fallback for backward compatibility with pre-12.0 clients  
    - Update Interface version to include 120000, 120001  
    - Hardcode version to 3.14.5 (remove packager placeholder)  
    - Remove CurseForge packager files (.pkgmeta, .luacheckrc, .editorconfig)  
    - Bundle library dependencies directly for GitHub distribution  
- Add release workflow  
