---
name: readme-i18n
description: Create and maintain bilingual (English + Simplified Chinese) READMEs with a language switcher, and make Gitee auto-select the localized README while GitHub keeps showing English. Use when asked to add a Chinese README, add a README language toggle, explain how Gitee/GitHub choose localized READMEs, or fix README language selection.
---

# Bilingual README (English + Simplified Chinese)

## Platform behavior (authoritative)

- **Gitee auto-selects by the viewer's browser language.** Put `README.md` (English)
  and `README.zh-CN.md` (Chinese) in the repo root:
  - Chinese browser → `README.zh-CN.md`
  - English browser → `README.md` (fallback when no language-specific file matches)
- **GitHub does NOT auto-select.** GitHub always renders `README.md`; there is no
  built-in multi-language README switching. Keep `README.md` in English and provide a
  link to the Chinese version.

## Recommended file layout

```
README.md          # English: GitHub default + Gitee English fallback
README.zh-CN.md    # Simplified Chinese
```

Do **not** add `README.en.md` unless you explicitly want Gitee to prefer it for English
browsers; `README.md` already serves as the English fallback.

## Language switcher (required in both files)

Place it immediately after the title / badges, near the top:

- `README.md`:
  ```markdown
  **English** | [简体中文](README.zh-CN.md)
  ```
- `README.zh-CN.md`:
  ```markdown
  [English](README.md) | **简体中文**
  ```

## Gitee localization rules (priority order)

1. `*.osc.md` wins (e.g. `README.osc.md`).
2. Language suffix:
   - Simplified Chinese: `zh`, `cn`, `zh-cn`, `zh_cn`
   - Traditional Chinese: `zh-hk`, `zh-tw`, `zh-yue`, `zh_hk`, `zh_tw`, `zh_yue`
   - English: `en`
3. Otherwise extension priority (`mdown`, `md`, `markdown`, `rst`, …).

Matching is case-insensitive, so `README.zh-CN.md` matches `zh-cn`.
The Gitee page footer also has a manual language switch.

## GitHub behavior

- Always renders `README.md`. `README.zh-CN.md` is not used as the repo landing page.
- The switcher link is the only way for users to reach the Chinese version.
- Do not claim GitHub can auto-switch; it cannot.

## Translation rules

- Keep the same structure, headings, order, code blocks, tables and links.
- Do not translate: commands, code identifiers, file paths, env vars, URLs, badge
  URLs, directory trees, or the ASCII architecture diagram.
- Translate prose and table descriptions. Keep proper nouns as-is (Tauri, Rust,
  WebView2, MediaPipe, aniface, three.js, GitHub, Gitee).
- Use plain, specific wording; avoid marketing fluff and emojis.
- Keep the language switcher accurate in both files.
- Update both files together when features change (do not let them drift).

## Verification

- Both files start with a switcher that links to the other file.
- `README.zh-CN.md` is valid UTF-8 and renders Chinese (no mojibake).
- Push to the Gitee remote, then:
  - Chinese browser language → shows `README.zh-CN.md`
  - English browser language → shows `README.md`
- On GitHub: `README.md` is shown; the “简体中文” link opens the Chinese README.

## Gotchas

- A single `README.md` cannot render differently per platform.
- Gitee selects by the **viewer's browser language** (and footer toggle), not by repo
  language settings.
- If you add `README.en.md`, keep it in sync with `README.md` (Gitee prefers `.en.md`
  for English browsers).
- Remember to push to both remotes for Gitee to see the new files.
