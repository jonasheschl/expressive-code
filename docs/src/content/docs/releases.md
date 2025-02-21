---
# Warning: This file is generated automatically. Do not edit!
title: Release Notes
---

This page combines all release notes of the Expressive Code monorepo.
You can find the source changelogs on GitHub in the subfolders of
[`packages`](https://github.com/expressive-code/expressive-code/tree/main/packages).

## 0.30.2

- Fixes missing CSS output for `uiFontWeight` and `codeFontWeight` style settings. Changed font weights are now properly respected. Thank you [@depatchedmode](https://github.com/depatchedmode)!

- Individual code blocks can now be switched to the base theme while an alternate theme is selected on the page level.

  Expressive Code differentiates between your base theme (= the first theme in `themes`) and your alternate themes (= any other entries in `themes`). Previously, as soon as an alternate theme was selected on the page level, e.g. by using `<html data-theme="some-theme-name">`, it wasn't possible to switch individual code blocks to the base theme anymore because of selector specificity issues. This has been resolved and block-level overrides should work as expected now.

- Fixes unexpected `InlineStyleAnnotation` behaviors to improve DX for plugin authors.

  - Inline styles now use `:where()` in selectors to reduce specificity and make them easier to override.
  - When applying multiple overlapping inline styles to the same line, render phases are now properly respected and later styles override earlier ones.
  - The `styleVariantIndex` property is no longer required. Inline styles without an index now apply to all style variants.
  - The default `InlineStyleAnnotation` render phase is now `normal`. The previous default setting `earliest` is now explicitly applied by `plugin-shiki` instead. This improves the API while still rendering syntax highlighting in the `earliest` phase to allow other annotations to wrap and modify the highlighted code.

- Themes that use transparency in unexpected places (e.g. the `rose-pine` themes) are now displayed correctly.

## 0.30.1

- Fixes parallel execution of multiple syntax highlighter creations and tasks.

  The Shiki plugin now ensures that async tasks like creating syntax highlighters, loading themes or languages are never started multiple times in parallel. This improves performance, reduces memory usage and prevents build errors on large sites.

## 0.30.0

- Potentially breaking: Increases minimum supported Astro version to 3.3.0 (when Astro switched to Shikiji).

- Changes the syntax highlighter used by `plugin-shiki` to Shikiji. Adds a `shiki: { langs: [...] }` option for loading custom languages.

  This change should not cause any differences in HTML output as all rendering is done by Expressive Code. The new `langs` option allows registering custom TextMate grammars in JSON form.

## 0.29.4

- Unknown code block languages now log a warning and render as plaintext instead of throwing an error.

- Adds the config option `useStyleReset`.

  This option determines if code blocks should be protected against influence from site-wide styles. This protection was always enabled before this release and could not be turned off.

  When enabled, Expressive Code uses the declaration `all: revert` to revert all CSS properties to the values they would have had without any site-wide styles. This ensures the most predictable results out of the box.

  You can now set this to `false` if you want your site-wide styles to influence the code blocks.

- Set `prerender = true` for injected routes to improve adapter support.

## 0.29.3

- Fix warning in Astro 4 due to renamed "entryPoint" property. Add Astro 4 to allowed peer dependencies.

## 0.29.2

- Comments like `// ...` are now no longer incorrectly detected as file names. Thanks [@kdheepak](https://github.com/kdheepak)!

## 0.29.1

- Fix asset URLs when using non-default Astro config options for `base`, `build.assets` or `build.assetsPrefix`.

## 0.29.0

- Update default fonts to match Tailwind CSS.

  The previous set of default fonts could result in very thin character line widths on iOS devices. This is now fixed by using the same widely tested set of fonts that Tailwind CSS uses.

- Clean up frontmatter after file name comment extraction.

  If a file name comment gets extracted from a code block without a `title` attribute, additional cleanup work is now performed on the surrounding lines:

  - If the code block's language supports frontmatter, and the comment was located in a frontmatter block that has now become empty, the empty frontmatter block gets removed.
  - If the line following the removed comment (or removed frontmatter block) is empty, it gets removed as well.

## 0.28.2

- Use `import type` in route handlers to avoid potential `APIRoute` warning.

## 0.28.1

- Add missing `files` entry to make `emitExternalStylesheet` option work.

  Sadly, this bug didn't occur before actually publishing the package - it worked fine when linking the package locally. Sorry about that!

## 0.28.0

- Add `emitExternalStylesheet` option.

  Determines if the styles required to display code blocks should be emitted into a separate CSS file rather than being inlined into the rendered HTML of the first code block per page. The generated URL `_astro/ec.{hash}.css` includes a content hash and can be cached indefinitely by browsers.

  This is recommended for sites containing multiple pages with code blocks, as it will reduce the overall footprint of the site when navigating between pages.

  **Important**: To actually benefit from caching, please ensure that your hosting provider serves the contents of the `_astro` directory as immutable files with a long cache lifetime, e.g. `Cache-Control: public,max-age=31536000,immutable`.

  Defaults to `true`.

## 0.27.1

- Fix missing `styleOverrides.collapsibleSections` declaration even after importing `@expressive-code/plugin-collapsible-sections`. Thanks [@fflaten](https://github.com/fflaten)!

## 0.27.0

- Add `useDarkModeMediaQuery` config option.

  This new option determines if CSS code is generated that uses a `prefers-color-scheme` media query to automatically switch between light and dark themes based on the user's system preferences.

  Defaults to `true` if your `themes` option is set to one dark and one light theme (which is the default), and `false` otherwise.

- Rendering multiple themes no longer generates duplicate CSS and HTML output.

  In previous versions, a full set of CSS styles was generated for each individual theme, and each code block was rendered multiple times to include the HTML for each theme.

  In this version, the CSS output has been changed to a single static set of base styles that uses CSS variables to allow efficient switching between themes.

  Also, the HTML output for code blocks is now generated only once, and theme-dependent styles are applied using CSS variables.

  These changes significantly reduce page size when using multiple themes, especially on pages with many code blocks.

  If you have added CSS code to your site that relies on the old output (e.g. by selectively hiding or showing theme-specific code blocks based on their class name), you will need to update it to work with the new output.

  > **Note**: Before writing new custom CSS, please consider if you can achieve your desired result out of the box now. For example, if your `themes` option contains one dark and one light theme, the `useDarkModeMediaQuery` option will generate a `prefers-color-scheme` media query for you by default.

- Add `minSyntaxHighlightingColorContrast` config option.

  This new option determines if Expressive Code should process the syntax highlighting colors of all themes to ensure an accessible minimum contrast ratio between foreground and background colors.

  Defaults to `5.5`, which ensures a contrast ratio of at least 5.5:1. You can change the desired contrast ratio by providing another value, or turn the feature off by setting this option to `0`.

- Config option `textMarkers` can no longer be an object.

  In previous versions, the `textMarkers` config option could be an object containing plugin options. This is no longer supported, as the only option that was available (`styleOverrides`) has been nested into the top-level `styleOverrides` object now.

  ```diff lang="js"
    /** @type {import('remark-expressive-code').RemarkExpressiveCodeOptions} */
    const remarkExpressiveCodeOptions = {
  -   textMarkers: {
  -     styleOverrides: {
  -       markHue: '310',
  -     },
  -   },
  +   styleOverrides: {
  +     textMarkers: {
  +       markHue: '310',
  +     },
  +     // You could override other plugin styles here as well:
  +     // frames: { ... },
  +   },
    },
  ```

- Move all plugin styles into nested sub-objects of top-level config option `styleOverrides`.

  In previous versions, there could be multiple `styleOverrides` scattered through the configuration (one per plugin with configurable style settings). This has been simplified to a single top-level `styleOverrides` object that contains all style overrides.

  Plugins can contribute their own style settings to this object as well by nesting them inside under their plugin name.

  ```diff lang="js"
    /** @type {import('remark-expressive-code').RemarkExpressiveCodeOptions} */
    const remarkExpressiveCodeOptions = {
      frames: {
        showCopyToClipboardButton: false,
  -     styleOverrides: {
  -       shadowColor: '#124',
  -     },
      },
  +   styleOverrides: {
  +     frames: {
  +       shadowColor: '#124',
  +     },
  +     // You could override other plugin styles here as well:
  +     // textMarkers: { ... },
  +   },
    },
  ```

- Rename config option `theme` to `themes`.

  Efficient multi-theme support using CSS variables is now a core feature, so the `theme` option was deprecated in favor of the new array `themes`.

  Please migrate your existing config to use `themes` and ensure it is an array. If you only need a single theme, your `themes` array can contain just this one theme. However, please consider the benefits of providing multiple themes.

  ```diff lang="js"
    /** @type {import('remark-expressive-code').RemarkExpressiveCodeOptions} */
    const remarkExpressiveCodeOptions = {
  -   theme: 'dracula',
  +   // Rename to `themes` and ensure it is an array
  +   // (also consider adding a light theme for accessibility)
  +   themes: ['dracula'],
    },
  ```

- Add `cascadeLayer` config option.

  This new option allows to specify a CSS cascade layer name that should be used for all generated CSS styles.

  If you are using [cascade layers](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_layers) on your site to control the order in which CSS rules are applied, set this option to a non-empty string, and Expressive Code will wrap all of its generated CSS styles in a `@layer` rule with the given name.

- Remove engine properties `configClassName` and `themeClassName`.

  The `configClassName` property was previously used to add a config-dependent class name to the CSS selectors used to style code blocks.

  As this property was automatically calculated by hashing the configuration object, it introduced a level of unpredictability, which has now been removed in favor of static base styles.

  The `themeClassName` property was previously used to add a theme-dependent class name to code blocks. Its format was `ec-theme-<name>`, where `<name>` was the kebab-cased name of the theme.

  As code blocks are now styled using CSS variables instead of generating multiple blocks for all themes and attaching class names to them, this property is no longer needed.

## 0.26.2

- Fix multiple different inline marker types on the same line. Thanks [@7c78](https://github.com/7c78)!

  The logic inside `flattenInlineMarkerRanges` had a flaw that caused various combinations of `mark`, `ins` and `del` inline markers on the same line to fail. This was fixed and more tests were added.

## 0.26.1

- Re-initialize copy to clipboard buttons after Astro view transitions.

## 0.26.0

- Themed selection now keeps the code foreground color intact (like VS Code).

  Color overlays no longer prevent text from being selectable.

- Make code blocks accessible by keyboard

- Improve caching logic to respect theme contents in addition to name

## 0.25.0

- Improve theme loading by allowing to pass more theme types directly.

  The `theme` config option now supports the following value types:

  - any theme object compatible with VS Code or Shiki (e.g. imported from an NPM theme package)
  - any ExpressiveCodeTheme instance (e.g. using `ExpressiveCodeTheme.fromJSONString(...)`
    to load a custom JSON/JSONC theme file yourself)
  - if you are using a higher-level integration like `remark-expressive-code` or `astro-expressive-code`:
    - any theme name bundled with Shiki (e.g. `dracula`)
  - any combination of the above in an array

- Add more colors to `ExpressiveCodeTheme.applyHueAndChromaAdjustments`, allow chaining.

  The `applyHueAndChromaAdjustments()` function now also adjusts `titleBar.activeBackground` and `titleBar.border` properly. Also, it returns the `ExpressiveCodeTheme` instance to allow chaining.

## 0.24.0

- Render frame borders on top of background, add `editorActiveTabHighlightHeight` style setting.

  Previously, borders were rendered around the editor / terminal window, which could lead to unwanted empty margins between the window background and the drop shadow (e.g. in theme `nord`). Now, the border is rendered on top of the background to resolve this issue, making fully transparent borders act like padding instead.

  Additionally, the `editorActiveTabHighlightHeight` style setting was introduced, which allows customizing the colorful line that highlights the active editor tab. It defaults to `borderWidth`.

- Migrate i18n functions to string templates with plural support.

  Translated texts including dynamic parts (e.g. a line count) previously used a function syntax. This was convenient to use during plugin development, but made it impossible to use the popular JSON file format as a source of translated texts. To make it easier to integrate Expressive Code, this release gets rid of the function syntax and adds a `formatTemplate` function that understands a simple string template syntax including placeholders and plural support.

  Simple placeholders are written as variable names in curly brackets, e.g. `{variableName}`.

  You can also use conditional placeholders by separating multiple choices with semicolons and optionally adding a condition before each choice, e.g. `{itemCount;1=item;items}` or `{variableName; 0=zero; >0=positive; negative}`.

- Pass global `styleOverrides` to plugin style resolver functions.

  This allows plugins to access their individual `styleOverrides` extensions even when values were defined at the global config level.

## 0.23.0

- Add config option `customizeTheme`.

  This optional function is called once per theme during engine initialization with the loaded theme as its only argument.

  It allows customizing the loaded theme and can be used for various purposes:

  - You can change a theme's `name` property to influence its generated CSS class name (e.g. `theme.name = 'dark'` will result in code blocks having the class `ec-theme-dark`).
  - You can create color variations of themes by using `theme.applyHueAndChromaAdjustments()`.

- Add plugin styles to the `styleOverrides` config option.

  So far, this object only contained core styles like colors, fonts, paddings and more. Now, plugins also contribute their own style settings to this object.

  For example, if the `frames` plugin is installed, you can now override its `shadowColor` by setting `styleOverrides.frames.shadowColor` to a color value.

- Add `applyHueAndChromaAdjustments` function to `ExpressiveCodeTheme`.

  You can now apply chromatic adjustments to entire groups of theme colors while keeping their relative lightness and alpha components intact. This can be used to quickly create theme variants that fit the color scheme of any website or brand.

  Adjustments can either be defined as hue and chroma values in the OKLCH color space (range 0–360 for hue, 0–0.4 for chroma), or these values can be extracted from hex color strings (e.g. `#3b82f6`).

  You can target predefined groups of theme colors (e.g. `backgrounds`, `accents`) and/or use the `custom` property to define your own groups of theme colors to be adjusted.

- Add outer wrapper when rendering multiple themes.

  When the `theme` option is set to an array containing multiple themes, the rendered code block groups are now wrapped inside `<div class="ec-themes-wrapper">...</div>`. This encapsulates all rendered themes in a single element and thereby ensures their consistent positioning on sites that would otherwise add margins between them due to adjacent sibling combinators.

- Add `styleOverrides` to `ExpressiveCodeTheme`.

  Themes can now provide their own `styleOverrides`, which take precedence over global `styleOverrides` and the default styles.

- Add support for extracting file names from CSS file comments.

## 0.22.2

- Hide summary marker on Safari for collapsible section

## 0.22.1

- Fix shifted collapsible sections when other plugins add or remove lines

## 0.22.0

- Implements the plugin-collapsible-sections plugin, which adds support for collapsed sections of code. These sections hide a number of code lines until the user chooses to expand them. Thanks [@birjj](https://github.com/birjj) for the contribution!

## 0.21.0

- Add multi-theme support to the `theme` config option.

  You can now pass an array of themes to the `theme` config option of `remark-expressive-code` and `astro-expressive-code`.

  This allows you to render each code block in your markdown/MDX documents using multiple themes, e.g. to support light and dark modes on your site.

  **Note**: If you use this feature, you will also need to add custom CSS code to your site to ensure that only one theme is visible at any time.

  To allow targeting all code blocks of a given theme through CSS, the theme property `name` is used to generate kebap-cased class names in the format `ec-theme-${name}`. For example, `theme: ['monokai', 'slack-ochin']` will render every code block twice, once with the class `ec-theme-monokai`, and once with `ec-theme-slack-ochin`.

## 0.20.0

- Add `removeCommentsWhenCopyingTerminalFrames` config option to `plugin-frames`. Thanks [@AkashRajpurohit](https://github.com/AkashRajpurohit)!

  If `true` (which is the default), the "Copy to clipboard" button of terminal window frames will remove comment lines starting with `#` from the copied text.

  This is useful to reduce the copied text to the actual commands users need to run, instead of also copying explanatory comments or instructions.

## 0.19.2

- Add support for Astro 3.0.0 incl. prereleases

## 0.19.1

- Add support for CSS variables to option `styleOverrides.terminalTitlebarDotsForeground`. Thanks [@delucis](https://github.com/delucis)!

## 0.19.0

- Add support for `diff`-like syntax and `lang` meta attribute. Thanks for the idea [@hirasso](https://github.com/hirasso)!

  To mark lines as inserted or deleted, you can now use the widely supported `diff` language as an alternative to adding line numbers to the opening code fence.

  You can even specify a separate syntax highlighting language by adding a `lang="..."` attribute to the opening fence. See [README.md](https://github.com/expressive-code/expressive-code/blob/main/packages/%40expressive-code/plugin-text-markers/README.md) for more details.

## 0.18.1

- Fix possible `querySelectorAll is not a function` issue on page content changes

## 0.18.0

- Add support for ANSI formatted code blocks. Thanks [@fflaten](https://github.com/fflaten)!

  You can now use the new language `ansi` to render code blocks containing ANSI escape sequences. This allows you to render colorful terminal output.

## 0.17.0

- Add support for Windows drive letters and typical path patterns to file name comment detection. Thanks [@fflaten](https://github.com/fflaten)!

## 0.16.0

- Improve file type support when extracting file names from comments. Thanks [@fflaten](https://github.com/fflaten)!

  - Adds more file types to the `LanguageGroups` object
  - Exports `LanguageGroups` to allow external modification
  - Extends automatic detection of frame type to differentiate between shell scripts and terminal sessions based on file name and/or shebang (if any)

## 0.15.0

- Synchronize package versions to prevent future dependency issues

## 0.14.0

- Add support to override frame types per code block. Thanks [@Princesseuh](https://github.com/Princesseuh)!

  By default, the plugin will automatically select the frame type (code editor or terminal) based on the language identifier in your code block's opening fence.

  You can override this behavior and force a specific frame type by adding an optional `frame="..."` attribute after the language identifier.

  The supported values for this attribute are `code`, `terminal`, `none` and `auto`. The default value is `auto`.

## 0.13.0

- Add config options `useThemedScrollbars` and `useThemedSelectionColors`. Thanks [@Princesseuh](https://github.com/Princesseuh)!

  Both options default to `true`. Set any of them to `false` to prevent themes from customizing their appearance and render them using the browser's default style.

## 0.12.2

- Fix non-working copy buttons in dynamically loaded content

## 0.12.1

- Make marked text selectable (#15). Thanks [@hirasso](https://github.com/hirasso)!

## 0.12.0

- Fix copy button on Firefox (still missing `:has()` support)

## 0.11.0

- Add default export for `astro add` support

- Reduce potential of unexpected changes through site-wide CSS

## 0.10.0

- Add RTL support (ensure that code lines are always LTR)

## 0.9.1

- Improve mobile core and copy button styles

- Enable stricter TypeScript checks (exactOptionalPropertyTypes), improve types

## 0.9.0

- Add tabWidth option to normalize tabs to spaces (default: 2)

## 0.8.4

- Fix feedback tooltip on mobile Safari

## 0.8.3

- Improve mobile core and copy button styles

## 0.8.2

- Fix CSS inconsistencies due to box-sizing

## 0.8.1

- Make `astro` peer dependency more tolerant

## 0.8.0

- Add support for localized texts, add German to frames plugin

## 0.7.0

- First working version of Astro integration

- Add custom renderer support

## 0.6.0

- Allow plugins to add JS modules

- Add copy to clipboard button

## 0.5.0

- Automatically trim whitespace at the end of lines, and remove empty lines at the beginning & end of code blocks

## 0.4.2

- Turn off explanations to improve Shiki performance

## 0.4.1

- Fix issues with color transforms

## 0.4.0

- Provide access to all expressive-code exports

- Change base font size unit to rem

- Make tab margins configurable, improve defaults

- Fix incorrect highlighting of terminal placeholders

## 0.3.0

- Synchronize package versions

## 0.2.0

- Initial release
