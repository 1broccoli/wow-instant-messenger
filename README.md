# This project has moved

-   <https://github.com/Legacy-of-Sylvanaar/wow-instant-messenger>

# wow-instant-messenger
WIM (WoW Instant Messenger) is a World of Warcraft addon which brings an instant messenger feel to communication in game.

## Features
* Whispers in their own windows.
* Chat in their own windows.
* Tabbed windows
* Highly configurable.
* History
* Copy and paste as:
  * Raw Text
  * BBCode
  * Advanced, intellectual window behaviors & animations.
  * Skins
  * Emoticons
  * Clickable web URLS for easy viewing. No more retyping a long url a friend sends you.
  * Customizable sound options.
  * Expose - great way to clear your screen of windows when you are in combat.

## Addon Compatibility: (Always make sure you are running the latest versions.
* Prat
* DBM


https://github.com/1broccoli/wow-instant-messenger

# Fix: BNet whisper errors, cross-realm names, and Anniversary client compatibility

## Overview

This PR addresses a mix of long-standing BNet whisper edge cases, cross-realm name handling, and several Anniversary client API differences that were throwing runtime errors.

## Issues Fixed

- BNet whispers could throw `Usage: BNSendWhisper(id,text)` when a window was flagged as BNet without a valid BN ID.
- Cross-realm whispers using tab-complete (`Name-Server`) weren’t normalized and failed to send.
- Existing whisper windows were reused even if the connection type changed (BNet vs normal).
- Anniversary client API differences triggered runtime errors (`SetGradient`, `GetMouseFocus`, `ChatFrame_OnEvent`, `ChatFrame_GetMessageEventFilters`, `SetMinResize`, `SetParent`).
- `setfenv(1, WIM)` meant some Lua globals were missing unless pulled from `_G`.

- **BNSendWhisper API Error** - When whispering Battle.Net contacts OR regular WoW friends list members, the addon threw: `Usage: BNSendWhisper(id,text)` - This occurred because players were incorrectly detected as BNet contacts when they weren't

- **Cross-Realm Whisper Failures** - Whispers to cross-realm players using tab autocomplete (e.g., "PlayerName-ServerName") failed with: `Unable to whisper '%s': Blizzard services may be unavailable`

- **Window State Confusion** - Whisper windows could be incorrectly reused across different connection types (BNet vs regular), causing one to be used with the wrong protocol

- **False BNet Detection** - The `ChatFrame_SendBNetTell` hook was marking windows as BNet windows even when the target was NOT a BNet friend (no valid BNet ID) - commonly affected regular WoW friends list members

- **Anniversary Client API Mismatches** - Multiple UI API changes caused runtime errors, including `SetGradient` signature changes, missing `GetMouseFocus`, missing `ChatFrame_OnEvent`, missing `ChatFrame_GetMessageEventFilters`, missing `SetMinResize`, and stricter `SetParent` argument types

- **Missing Lua Globals in WIM Environment** - Some globals like `type` and `pcall` were nil in the WIM environment and caused runtime failures when used without `_G`.

## Root Causes (Short Version)

- BN IDs were not stored reliably and could be missing when needed.
- Window reuse didn’t account for connection type changes.
- Cross-realm names were only normalized for the old space-separated format.
- The Anniversary client swaps or removes some UI APIs used by WIM.

## Files Modified

- `Modules/WhisperEngine.lua`
- `Modules/MinimapIcon.lua`
- `Sources/ToolBox.lua`
- `Sources/WindowHandler.lua`
- `Sources/TabHandler.lua`
- `Sources/Skinner.lua`
- `Sources/Options/Options.lua`
- `Libs/LibChatHandler-1.0/LibChatHandler-1.0.lua` (Added or it was missing in my version)
- `WIM.lua`
- Updated .Toc for TBC 20505
- Added (CF) Pegga / (github) 1broccoli as contributor 

## Notable Changes

### 1) Cross-realm name normalization

**Before:**
```lua
user = string.gsub(user," ","") -- removes spaces only
user = FormatUserName(user);
```

**After:**
```lua
user = string.gsub(user," ","") -- removes spaces
user = string.gsub(user, "-[^-]+$", ""); -- removes server name suffix (e.g., "-ServerName")
user = FormatUserName(user);
```

### 2) Window reuse validation and BN ID updates

**Before:**
```lua
if(obj and obj.type == "whisper") then
    return obj;
end
```

**After:**
```lua
if(obj and obj.type == "whisper" and obj.isBN == isBN) then
    if(isBN and bnID) then
        obj.bn = obj.bn or {};
        obj.bn.id = bnID;
    end
    return obj;
end
```

### 3) Safe BNSendWhisper calls (split and single)

**Before:**
```lua
if(Windows[to] and Windows[to].isBN) then
    _G.BNSendWhisper(Windows[to].bn.id, chunk);
else
    _G.ChatThrottleLib:SendChatMessage(PRIORITY, HEADER, chunk, CHANNEL, EXTRA, to);
end
```

**After:**
```lua
local bnID = Windows[to] and Windows[to].bn and Windows[to].bn.id;
if(bnID and type(bnID) == "number" and bnID > 0) then
    _G.BNSendWhisper(bnID, chunk);
elseif(Windows[to]) then
    _G.ChatThrottleLib:SendChatMessage(PRIORITY, HEADER, chunk, CHANNEL, EXTRA, to);
end
```

### 4) Anniversary client shims

- `WIM.SetGradient` and `WIM.SetGradientRGB` handle both legacy and new `SetGradient` signatures.
- `WIM.GetMouseFocus` falls back to `GetMouseFoci` when needed.
- LibChatHandler falls back to `ChatFrame_MessageEventHandler` if `ChatFrame_OnEvent` is missing.
- Guard `ChatFrame_GetMessageEventFilters` if the API is absent.
- Use `SetResizeBounds` when `SetMinResize` does not exist.
- Use `_G.UIParent` instead of string names in `SetParent`.
- Import missing Lua globals from `_G` when required.


### Mouse Focus API Shim
`GetMouseFocus` is missing on the Anniversary client. A safe wrapper now tries `GetMouseFocus()` first and falls back to `GetMouseFoci()`.

### Chat Frame Event API Shim
`ChatFrame_OnEvent` is missing on the Anniversary client. LibChatHandler now falls back to `ChatFrame_MessageEventHandler` when needed.

### Chat Frame Filters Guard
`ChatFrame_GetMessageEventFilters` may be missing. WIM now skips filters safely when the API is absent.

### Resize Bounds Compatibility
`SetMinResize` is missing on the Anniversary client. WIM now uses `SetResizeBounds` when available and falls back to `SetMinResize`.

### SetParent Argument Fix
`SetParent` now uses the frame object (`_G.UIParent`) instead of string names.

### Environment Global Imports
Added explicit imports for missing Lua globals (e.g., `type`, `_G.pcall`) where needed.
