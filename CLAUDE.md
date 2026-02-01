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
   - **Tweet Detection**: `check_tweets(node)` (around line 4954) - identifies and processes tweets, ideal place to add per-tweet features
   - **Dynamic Content Handling**: `start_mutation_observer()` (around line 5260) - monitors DOM changes for React's dynamic rendering

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

### Adding New Features

To add a new feature to the script, follow these steps:

1. **Add Configuration Option** (around line 128 in `OPTIONS` object):
   ```javascript
   ,   NEW_FEATURE_NAME : false // true: feature description
   ```

2. **Add Localization Strings** (around lines 5720-5830):
   - Japanese case (around line 5720): Add message to Japanese Object.assign()
   - English default case (around line 5770): Add message to English Object.assign()
   ```javascript
   "NEW_FEATURE_NAME": "Feature description in language",
   ```

3. **Add GM_config Field** (around line 5830 in `GM_config.init()`):
   ```javascript
   NEW_FEATURE_NAME : {
       label : messages.NEW_FEATURE_NAME,
       type : 'checkbox',  // or 'text', 'select', etc.
       default : OPTIONS.NEW_FEATURE_NAME,
   },
   ```

4. **Implement Feature Logic**:
   - Create a new function for the feature (before `check_tweets()` around line 4950)
   - Call the function in appropriate places:
     - In `check_tweets()` for per-tweet operations
     - In `main()` for initial page load (around line 5740)
     - In MutationObserver if needed for dynamic content (around line 5260)

5. **Update Version Number** (line 7):
   ```javascript
   // @version         0.1.XX
   ```

### Example: Adding UI Hiding Feature

For features that hide Twitter UI elements:
- Create a function like `hide_unwanted_elements(node)`
- Use `querySelectorAll()` to find elements by:
  - `data-testid` attributes
  - `aria-label` attributes
  - Text content matching
  - SVG path patterns
- **Important for proper alignment**: Don't just hide the button itself
  - Find the button's container (usually a direct child of `div[role="group"]`)
  - Hide the container instead of the button to maintain flex layout
  - Traverse up the DOM tree to find `role="group"` parent
  - Hide the direct child of the group to prevent alignment issues
- Set `element.style.display = 'none'` to hide
- **Adjust parent flex layout**: Set `justifyContent: 'space-between'` on button groups
- Call from `check_tweets()` when `OPTIONS.YOUR_FEATURE` is enabled
- Ensure it works with React's dynamic rendering via MutationObserver

**Layout Fix Example**:
```javascript
// Bad - causes misalignment
button.style.display = 'none';

// Good - maintains alignment
var container = button;
while (container && container.parentElement?.getAttribute('role') !== 'group') {
    container = container.parentElement;
}
container.style.display = 'none'; // Hide the group's direct child

// Also adjust the group's flex properties
group.style.justifyContent = 'space-between';
```

## Important Notes

- **Format On Save is Disabled**: `.vscode/settings.json` explicitly disables `formatOnSave`
- **Multiple Twitter UI Versions**: The script handles both legacy and React-based Twitter/TweetDeck UIs
- **Cross-Domain Permissions**: Script has `@connect` permissions for `twitter.com`, `x.com`, `twimg.com`
- **External CDN Dependencies**: JSZip and FileSaver.js are loaded from CDNs at runtime
- **License**: MIT (see header comments for full text and attribution to original author furyu)
- **Extending Functionality**: When adding features unrelated to images:
  - Keep configuration options in the `OPTIONS` object
  - Add localization for all supported languages (ja, en)
  - Make features optional via GM_config checkboxes
  - Use `is_react_page()` to handle React vs Legacy Twitter
  - Test with MutationObserver for dynamic content
  - Avoid breaking existing image-related functionality

## User Installation

Users install via: https://github.com/blurSong/twOpenOriginalImage/raw/main/twOpenOriginalImage.user.js

The script auto-updates via the `@updateURL` directive in the userscript header.

## Fork Features

This fork adds:
- **Wallpaper Buttons**: "Download", "Save W", "Save P" buttons on timeline tweets
- **Keyboard Shortcuts**: `[w]`/`[p]` in overlay to save as wallpaper
- **Resolution Display**: Shows image dimensions in overlay status
- **Custom Paths**: `DOWNLOAD_WALLPAPER_PATH`, `DOWNLOAD_PHONE_WALLPAPER_PATH` options

Key additions:
- `save_to_wallpaper()` / `save_to_wallpaper_via_xhr()` - wallpaper download functions
- `download_button_template`, `wallpaper_button_template`, `phone_wallpaper_button_template` - button templates
- CSS styling in `set_user_css()` for 3 distinct button styles
