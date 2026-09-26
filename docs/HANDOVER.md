# Handover: Iconic Theme

State of the project as of **2026-09-26**, for whoever picks it up next.

## 1. What this is

Iconic Theme is a Home Assistant frontend theme forked from `luxury_dashboard` in [ruudmens/home-assistant-dashboard](https://github.com/ruudmens/home-assistant-dashboard). It keeps the upstream look (charcoal with gold) but replaces the typography with Apple's SF family and reworks the colour model to pass WCAG 2.2 AA. Only the **theme** was forked: the upstream dashboards, automations and custom cards are not part of this repo.

Development was AI-accelerated with Claude Code: source research against the HA frontend, contrast maths, the deploy and verification scripts, and these docs.

## 2. Current status

| Item | Status |
|---|---|
| Theme file | Done; `themes/iconic_theme.yaml` |
| Contrast gate | 30/30 WCAG AA checks pass (`scripts/contrast_check.py`) |
| Deployed to the reference HA instance | Yes; loaded in dark-only mode with 50 colour variables (verified with `scripts/verify_theme.py`) |
| Visual check in a browser | **Not done yet** (see open items) |
| HACS | `hacs.json` present; installable as a custom repository, not submitted to HACS default |

Reference environment: Home Assistant OS, Core **2026.9.3**, frontend **20260826.7**.

## 3. How the theme works (read before editing)

These are the non-obvious facts, verified in the HA frontend source (`src/state/themes-mixin.ts`, `src/common/dom/apply_themes_on_element.ts`, `src/resources/theme/**`):

1. **Dark vs light is decided by `modes`.** A theme without `modes.dark` is always rendered as **light**, whatever the user or OS prefers. HA then keeps its light base (`--input-fill-color`, menu and dialog surfaces, `--ha-color-*` semantic tokens) and only overlays your variables. That was the cause of the invisible labels and pull-downs. With only `modes.dark` present, HA forces dark mode and applies its full dark base underneath.
2. **Top-level keys apply in every mode, and `modes.dark` keys win.** Fonts and radii live at the top level; every colour lives under `modes.dark`.
3. **Custom themes do not get a generated primary palette.** HA only derives the `ha-color-primary-05…95` ramp from `primary-color` for its *default* theme. Switches, filled buttons, links and focus rings read that ramp, so the theme must define all eleven steps itself.
4. **Dead variables.** `switch-checked-color`, `switch-unchecked-*` and `paper-slider-*` are no longer read by any component in 2026.9, so they were removed. Sliders use `ha-slider-thumb-color`, `ha-slider-indicator-color` and `ha-slider-track-color`.
5. **Filled (loud) buttons** use `ha-color-fill-primary-loud-*` with text in `ha-color-on-primary-loud`. HA's default is white text on primary-40; the theme uses gold primary-70 with dark text, which passes both text and boundary contrast.

## 4. Design decisions

| Decision | Why |
|---|---|
| Dark-only (no `modes.light`) | The brand is dark. A light mode would double the palette and testing surface for no current need. |
| System font stack instead of shipping SF | Apple's SF licence forbids web self-hosting. The stack gives real SF on Apple devices and native fonts elsewhere. |
| Secondary text and off-state icons in warm grey `#c2bbb3` instead of gold | In the upstream theme gold meant everything: secondary text, links, on and off. Now gold only means *active, link or selected*. |
| Gold fills with dark text | White on `#eac578` is 1.65:1; `#211f1f` on gold is 9.96:1. |
| Slider track `#8a827c` | The lowest warm grey that clears 3:1 (3.82:1) without competing with the gold indicator. |

## 5. Working on it

```bash
# 1. Edit colours under modes.dark in themes/iconic_theme.yaml
# 2. Check contrast
python3 scripts/contrast_check.py
# 3. Deploy to a test instance (backs up, validates, reloads, verifies)
HA_HOST=root@homeassistant.local ./scripts/deploy.sh
# 4. Hard-refresh the browser and check: Settings → Automations → New automation
#    (text fields and dropdowns), a dialog, the sidebar, a toggle, a slider
```

When adding a colour pair the UI actually renders, add a line to `checks` in `scripts/contrast_check.py`.

### SSH access to Home Assistant

`deploy.sh` needs the **Terminal & SSH** (or *Advanced SSH & Web Terminal*) add-on:

- **Configuration → Network**: set a host port (22). This section has its own Save button, and if it's left blank the add-on only works as the web terminal.
- **authorized_keys**: add your public key, then restart the add-on.
- If `homeassistant.local` resolves to several addresses, SSH to the IP directly.

Backups made by `deploy.sh` are stored on the host in `/config/backups_manual/iconic_theme.yaml.<timestamp>`. To roll back, copy one back to `/config/themes/iconic_theme.yaml` and run `frontend.reload_themes`.

## 6. Open items and next steps

1. **Visual QA in a browser.** The contrast figures are calculated from the theme values, but no one has screenshotted the rendered UI yet. Check form fields, dropdown menus, dialogs, the sidebar, toggles, sliders, the code editor and the history and energy charts on desktop and on the wall tablet. An automated axe-core scan of a few pages would make a good follow-up.
2. **Unthemed neutral surfaces.** Menus and dialogs use HA's dark neutral ramp (`ha-color-neutral-*`, cool grey `#202020`), while cards are warm `#2c292a`. This is accessible but slightly off-brand; defining a warm neutral ramp would unify it.
3. **Font on non-Apple devices.** Android wall tablets render Roboto. Consider self-hosting Inter as a fallback (instructions in the README).
4. **Upstream `luxury_dashboard` has the same accessibility bugs.** It could be fixed the same way and offered upstream as a PR.
5. **HACS default listing.** Submitting it needs a tagged GitHub release and screenshots in the README.
6. **Version floor.** Only tested on 2026.9.3. Establish the minimum supported HA version and add `"homeassistant"` to `hacs.json`.

## 7. Reference instance notes

These are unrelated to this repo, but they exist on the instance where the theme was developed. Don't be surprised by them:

- The upstream `luxury_dashboard` theme is also installed, together with its assets in `/config/www/assets/`.
- `configuration.yaml` has `frontend: extra_module_url: /local/assets/css/load-fonts.js`, a Poppins loader for `luxury_dashboard`. Iconic Theme doesn't need it, and it can be removed if `luxury_dashboard` is uninstalled (Core restart required).
- Pre-change backups are in `/config/backups_manual/`.
