# o2cloud_login_wizard — Wizard (2-step login) 📧

A **Gmail-style two-step** login skin for RoundCube Webmail. Visitors enter their e-mail first; once the format is validated, a dynamic avatar appears and the password field reveals itself — only the login page changes, the rest of your webmail keeps the standard **Elastic** look and feel.

![Démo](https://img.shields.io/badge/Démo-en%20ligne-1a73e8?style=for-the-badge) ← [Voir la démo](https://o2cloud.fr/demos/demo-wizard-2steps.html)

## ✨ Features

- **Two-step login flow** : e-mail first, password second — just like a Google account sign-in
- **Live e-mail format validation** : inline error message if the format is invalid, no page reload
- **Dynamic avatar** : shows the first letter of the entered e-mail once validated
- **"Modifier" / back link** : go back and edit the e-mail at any time
- **Smart auto-skip** : if a password manager has already filled both fields on page load, the wizard jumps straight to step 2
- **Material-style floating labels**, automatic light/dark mode (`prefers-color-scheme`)
- **Show/hide password** toggle
- **Chrome autofill fix**
- **Zero impact on the rest of the webmail** : only the login page is touched
- **No build step** : plain CSS + vanilla JS

## 🚀 Installation

### Prerequisites

- RoundCube Webmail ≥ 1.6 (skin inheritance via `meta.json` `extends` requires this)
- The **Elastic** skin must remain installed (this skin extends it, it does not replace it)
- Outbound HTTPS access from the server (Google Fonts + FontAwesome CDN)

### Quick install

```bash
# 1. Extract the skin into RoundCube's skins/ directory
unzip o2cloud_login_wizard.zip -d /var/www/roundcube/skins/

# 2. Fix ownership/permissions
chown -R www-data:www-data /var/www/roundcube/skins/o2cloud_login_wizard

# 3. Activate it in config.inc.php
echo "\$config['skin'] = 'o2cloud_login_wizard';" >> /var/www/roundcube/config/config.inc.php

# 4. Clear RoundCube's compiled-template cache
rm -rf /var/www/roundcube/temp/*
```

Hard-refresh your browser (Ctrl+Shift+R) and you're done.

> ⚠️ **Important** : this skin is a genuine progressive-disclosure UI, not a fake two-step form. The real RoundCube `_user` and `_pass` fields stay inside the same `<form>` at all times — they are only moved between two visual containers (step 1 / step 2). The final submit always sends both values exactly as RoundCube expects; no authentication behavior is changed.

---

## ⚙️ Customization

All visual settings are CSS variables at the top of `styles/login.css` :

```css
:root {
    --gs-primary: #1a73e8;
    --gs-error: #d93025;
    --gs-radius-card: 24px;
}
```

The e-mail format check and step texts are in `scripts/login.js` :

```js
var EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
// ...
showError('Veuillez saisir une adresse e-mail valide (ex. nom@domaine.fr)');
```

---

## 📁 File structure

| Path | Description |
|---|---|
| `meta.json` | Skin declaration — `"extends": "elastic"` |
| `templates/login.html` | Login page template (overrides Elastic's, adds our CSS/JS) |
| `styles/login.css` | All visual styling, scoped to `body.task-login` |
| `scripts/login.js` | Builds the 2-step UI and validates the e-mail format client-side — never touches auth logic |

No other file is provided or modified: every other page of RoundCube keeps rendering through the standard Elastic skin.

---

## 🔍 Troubleshooting

**Page doesn't change after install**
```bash
grep "skin" config/config.inc.php   # must show $config['skin'] = 'o2cloud_login_wizard';
rm -rf temp/*
```
Then hard-refresh (Ctrl+Shift+R).

**Step 2 never appears, "Suivant" seems to do nothing**
Open the browser console — if `EMAIL_RE` rejects the value you typed, an inline red message should appear under the field instead. Check that you're testing with a `nom@domaine.tld`-shaped address.

**The "remember me" checkbox disappears**
`scripts/login.js` looks for `label[for="rcmloginsavepass"]` and relocates it into step 2. If your RoundCube config doesn't render that checkbox at all (e.g. `$config['login_rememberme'] = false`), there is simply nothing to relocate — this is expected.

**CSS/JS return 404**
Don't prefix asset paths with the skin folder name — RoundCube already resolves root-relative paths against the active skin folder.

---

## 🛠️ Technical details

- **Skin inheritance** : declared via `"extends": "elastic"` in `meta.json`. Only `templates/login.html` is overridden.
- **Real fields, fake steps** : the script wraps RoundCube's native `name="_user"` / `name="_pass"` inputs into two purpose-built containers (`.gs-step`) and toggles their visibility with a CSS class — no DOM removal, no detached inputs, no risk to form submission.
- **Original table markup is hidden, not deleted** : after relocating the real inputs out of RoundCube's generated `<table>`, the now-empty table is set to `display: none` rather than removed, in case a future RoundCube version adds extra fields to it (multi-host selector, etc.) that this skin doesn't yet know to relocate.
- **Auto-skip detection** : on load, if `passInput.value` is already non-empty (typical of password-manager autofill) and the e-mail looks valid, the skin jumps straight to step 2 — no extra click needed.
- **Field selection** : by `name` attribute rather than `id`, for robustness across RoundCube versions/configs.

---

## ⚠️ Security notes

- This skin contains no credentials and no server-side code — pure front-end (CSS/JS) layered on top of RoundCube's own login form.
- The e-mail format check is **cosmetic only** (better UX, instant feedback) — it is not a security control. RoundCube's server-side authentication is completely unaffected and still performs its own validation.
- It depends on the Google Fonts and FontAwesome CDNs. For air-gapped or high-security environments, self-host these assets.
- Always review third-party skins before deploying to production.

---

## 📄 Changelog

| Version | Date | Changes |
|---|---|---|
| v1.0.0 | June 2026 | Initial release — 2-step Gmail-style flow, e-mail format validation, dynamic avatar, auto-skip on password-manager autofill, automatic light/dark mode, Chrome autofill fix |

---

## 🤝 Contributing

1. Fork the project
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add my feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## 👤 Author

**o2Cloud**
- GitHub: [@o2Cloud-fr](https://github.com/o2Cloud-fr)
- Email: github@o2cloud.fr

---

[![RoundCube](https://img.shields.io/badge/RoundCube-%E2%89%A5%201.6-1a82e2?style=for-the-badge)](https://roundcube.net)
[![Elastic skin required](https://img.shields.io/badge/Requires-Elastic%20skin-1a73e8?style=for-the-badge)](https://github.com/roundcube/roundcubemail)
[![2-step login](https://img.shields.io/badge/UX-2--step%20login-34A853?style=for-the-badge)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

---

⭐ **If this skin helps you, feel free to give it a star!**

> **Note**: This skin only replaces the RoundCube login page. It does not modify authentication logic, account data, or any other part of the webmail.
