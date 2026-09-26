# Changelog

## 1.0.0 (2026-09-26)

First release of Iconic Theme, forked from `luxury_dashboard` in [ruudmens/home-assistant-dashboard](https://github.com/ruudmens/home-assistant-dashboard). Developed in an AI-accelerated style with Claude Code.

### Typography
- Replaced Poppins with Apple's SF family through the system font stack: SF Pro Text for body text, SF Pro Display for headings and titles, SF Mono for code (previously Poppins).
- Added the current `ha-font-family-body`, `-heading`, `-longform` and `-code` variables alongside the legacy `paper-font-*` ones.
- Removed the Google Fonts `@import` from `card-mod-root`, so the theme makes no external requests.

### Accessibility (WCAG 2.2 AA)
| Issue in upstream theme | Fix | Result |
|---|---|---|
| Colours at the top level, so HA rendered the theme as **light** and kept light form, dropdown, menu and dialog surfaces under white text (invisible labels and pull-downs) | Moved all colours under `modes: dark:` (dark-only theme) | Text up to 14.41:1 |
| `text-primary-color: #ffffff` on gold fills | Dark `#211f1f` text on gold | 1.65:1 → 9.96:1 |
| `switch-*` variables no longer read by the frontend; switches, buttons, links and focus used HA blue | Full gold `ha-color-primary-*` ramp, loud fills in gold with dark text | Links 6.81:1, focus 6.81:1 |
| `paper-slider-*` no longer read; slider track `#444` | `ha-slider-thumb/indicator/track-color` | Track 1.48:1 → 3.82:1 |
| Off-state and on-state icons both gold | Off `#c2bbb3`, on `#eac578` | Off 7.58:1, on 8.75:1 |
| Secondary text gold, the same as links and active states | Warm grey `#c2bbb3` | 7.58:1 |
| Input fill identical to card | `#363233` fill | Label 6.65:1 |
| Selected sidebar item barely distinguishable | `#3a3536` background, gold text | 7.32:1 |

### Tooling
- `scripts/contrast_check.py`: 30 WCAG checks read from the theme file; non-zero exit on failure.
- `scripts/deploy.sh`: contrast gate, backup, copy, config check, theme reload and load verification over SSH.
- `scripts/verify_theme.py`: websocket check on the HA host that the theme loaded with dark mode.
- `hacs.json` for installation as a HACS custom repository.
