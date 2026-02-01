# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a userscript project called "twOpenOriginalImage" (Twitter 原寸びゅー) that enables viewing and downloading original-size images on Twitter/X. The script runs in userscript managers like Tampermonkey, ScriptCat, or Violentmonkey.

## Architecture

### Single-File Structure

This project uses a **single-file architecture** - all code resides in `twOpenOriginalImage.user.js` (~265KB, ~5700+ lines). There is no build process, bundler, or compilation step. The file is distributed directly to users.

### Key Architectural Components

1. **Platform Detection** (lines ~174-198): Determines the execution environment
   - `is_twitter()`, `is_tweetdeck()`, `is_media_url()` - URL pattern matching
   - `is_react_page()`, `is_react_twitter()`, `is_react_tweetdeck()` - React-based UI detection
   - `is_legacy_twitter()`, `is_legacy_tweetdeck()` - Legacy UI detection

2. **Initialization Flow** (line 1959):
   - `initialize(user_options)` - Main entry point that:
     - Merges user options with defaults
     - Checks if script should run (OPERATION flag, TweetDeck settings)
     - Calls `initialize_download_helper()` (line 1703)
     - Sets up URL validation
     - Initializes button injection and click handlers

3. **Configuration System** (line 128):
   - `OPTIONS` object contains all behavioral flags
   - Uses `GM_config` library for settings UI (registered via `GM.registerMenuCommand`)
   - Settings stored via `GM.getValue`/`GM.setValue`

4. **External Dependencies** (required at lines 19-21):
   - JSZip 3.9.1 - for ZIP file creation
   - FileSaver.js 2.0.5 - for file downloads
   - GM_config - for settings UI

5. **Core Functionality Areas**:
   - **URL Manipulation**: `normalize_img_url()`, `get_img_url()`, `get_img_url_orig()` - converts Twitter image URLs to original size
   - **Download System**: `download_zip()` (line 1449), `save_blob()` (line 1382) - handles individual and batch downloads
   - **Button Injection**: `add_open_button()` (line 2014) - adds UI buttons to tweets
   - **Overlay Display**: Handles in-page image viewing with custom controls
   - **Click Override**: Intercepts thumbnail clicks to show original images

### Localization

The `_locales/` directory contains message files (en, ja, ko) but these appear to be legacy from a browser extension version. The userscript itself has embedded localization strings.

## Development

### No Build System

This project has **no build process**. Edit `twOpenOriginalImage.user.js` directly:
- No transpilation
- No bundling
- No minification
- No package.json dependencies

The script uses plain ES5/ES6 JavaScript compatible with userscript managers.

### Testing

Install the script locally in your userscript manager:
1. Open the userscript manager dashboard
2. Create a new script or import `twOpenOriginalImage.user.js`
3. Navigate to Twitter/X to test changes
4. Reload the page after making edits

### Version Bumping

Update the version in the userscript header (line 6):
```javascript
// @version         0.1.21
```

### Configuration Flags

Key `OPTIONS` to understand (lines 128-156):
- `OVERRIDE_CLICK_EVENT` - Controls thumbnail click interception
- `DISPLAY_OVERLAY` - Opens images in-page vs new tabs
- `DOWNLOAD_HELPER_SCRIPT_IS_VALID` / `DOWNLOAD_ZIP_IS_VALID` - Download features
- `SHOW_IN_DETAIL_PAGE` / `SHOW_IN_TIMELINE` - Where script activates
- `ENABLED_ON_TWEETDECK` - TweetDeck support toggle

## Important Notes

- **Format On Save is Disabled**: `.vscode/settings.json` explicitly disables `formatOnSave`
- **Multiple Twitter UI Versions**: The script handles both legacy and React-based Twitter/TweetDeck UIs
- **Cross-Domain Permissions**: Script has `@connect` permissions for `twitter.com`, `x.com`, `twimg.com`
- **External CDN Dependencies**: JSZip and FileSaver.js are loaded from CDNs at runtime
- **License**: MIT (see header comments for full text and attribution to original author furyu)

## User Installation

Users install via: https://github.com/Coxxs/twOpenOriginalImage/raw/main/twOpenOriginalImage.user.js

The script auto-updates via the `@updateURL` directive in the userscript header.
