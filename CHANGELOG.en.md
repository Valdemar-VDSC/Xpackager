*[Version française : [CHANGELOG.md](CHANGELOG.md)]*

# Changelog

## 1.2.1

Thirty-two labels had no translation at all and showed up in French whatever the system
language: the **File** menu, the five project templates, the save-on-close question, the
three panels explaining which certificate to choose, and a few labels in the settings
window. They now speak the application's six languages.

The getting-started guide is also available in English.

## 1.2.0

First public release.

### The essentials

- The payload is assembled by dragging from the Finder, with permissions set per item.
- Custom installation with several components: checkboxes, preselected choices, pre- and
  post-install scripts.
- Installation requirements: minimum macOS version, allowed architectures, presence of a
  file, available memory.
- The installer's four screens in a rich text editor, background image included, with
  import of `.rtf` and plain text.
- **Multilingual installer**: screens, title and choice labels translated language by
  language, with a mandatory reference language as a fallback, a translation progress
  indicator and normalized regional codes — `pt-BR`, `zh-Hans`.
- `Developer ID Installer` signature, payload re-signed with the Hardened Runtime,
  notarization and stapled ticket.
- `xpackagerbuild` command line tool: the same engine, the same project file, the same
  package.

### First launch

An environment check opens on first launch and stays available from **Help ▸ Check
environment…**. It inspects Apple's tools, both Developer ID certificates and the
notarization profile — everything that lives outside the application and would otherwise
only show up at the end of a failed build.

A development certificate gets its own message: it looks like a Developer ID in a signing
menu, and Apple refuses to notarize with it.

### Donationware

XPackager is free. A donation button sits in the **About** window and in the **Help ▸
Donate…** menu.

### Notes

- Requires **macOS 15**.
- This version's package is built by XPackager itself.
