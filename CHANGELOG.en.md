*[Version française : [CHANGELOG.md](CHANGELOG.md)]*

# Changelog

## 1.5.0

### Packages a third smaller

Components are now compressed with `pkgbuild`'s latest algorithm. XPackager's own package
drops from 22.0 to 15.1 MB, for identical contents. The option is on by default and lives in
the requirements. It needs a valid minimum macOS version above it: without one `pkgbuild`
refuses to build, and XPackager then leaves compression alone rather than failing the build.
The caption under the switch says so.

### Applications to quit

Each component accepts a list of **applications to quit**: the Installer asks for them to be
closed before installing. This is the answer to replacing a running application — every
update. The **Choose…** button opens a picker and reads the identifier out of the application
you point at, so you need not know it. The Installer only acts on it if that component is
selected.

### An uninstall script

**File ▸ Generate an uninstaller script…** writes a `.sh` script to ship alongside your
package. macOS offers no uninstaller, but it keeps an exact record of what each package laid
down: the script reads those receipts, removes files and symlinks, then directories **only if
they are left empty**, and forgets the receipts. Shared directories — `/Applications`,
`/usr/local/bin`, `~/Library/Preferences` and their like — are spared even when empty. A dry
run shows what would go without touching anything, and the script refuses to run without
`sudo`.

### Apple's rejection, in plain words

When notarization fails, the build window used to show the raw JSON returned by `notarytool`.
It now shows a digest: the reason in one sentence, then each issue with its locations and the
link to the Apple page that explains it. Identical issues are grouped — Apple repeats the same
sentence for every binary and every architecture, which turned a single fault into two hundred
lines.

### Downloads marked as quarantined

XPackager now sets the quarantine flag on the package it downloads, as a browser does. It is
up to the downloading application to set it; without it, macOS examines nothing on opening.
So the package is checked twice: by XProtect analysis and Gatekeeper if you open it yourself
from the Finder, and by XPackager itself — signed, notarized, signed by the expected publisher
— before it is handed to the Installer. The flag does not travel through the package: the
installed application does not inherit it, so there is nothing to clean up afterwards.

Thanks to Arnaud Nicolet for raising it on the Xojo forum. The 1.4.0 notes presented the
internal check as the only available safeguard: setting the flag costs nothing, and macOS does
more than verify a signature.

### The detail pane scrolls

The pane's four pages — settings, components, requirements, presentation — scroll when they do
not fit the window. The window minimum drops from 780 to 600 points: enough to work on a
13-inch laptop without a field staying out of reach.

## 1.4.0

### Checking for updates

A **Check for Updates…** item in the application menu queries the latest published version,
shows the installed version alongside it, and lists what is new — rendered, not as raw
Markdown.

The **Download** button puts the package in your *Downloads* folder, with a progress bar.
**Before handing it to Apple's Installer, XPackager checks it itself**: a file downloaded by
an application does not carry the quarantine flag, so Gatekeeper would not examine it on
opening. XPackager verifies that it is signed, notarized, and signed by the expected
publisher. If any of the three is missing, it does not open it and shows what it found.

The check also runs at startup, **silently**: nothing appears when you are up to date. A
checkbox at the bottom of the window turns it off.

## 1.3.0

### Choice hierarchy

A component can now be a **choice group**: it installs nothing and gathers under a single
checkbox the components that name it in their **Under** menu. Groups nest, which produces
the multi-level lists of larger installers. A parentage that would loop back on itself is
discarded at build time rather than producing an unreadable installer.

### Owner and group per item

Every payload item accepts an **owner** and a **group**. Left empty — the normal case — it
belongs to `root:wheel`, as before. Filled in, they are applied at the end of the
installation, to the item and to everything inside it; an item placed below takes over for
its own subtree. Enough to ship a daemon, or a folder a service must be able to write to.

### Install on the startup volume only

A checkbox in the requirements forbids installing on another disk or into the user's home
folder. Required for anything living in `/Library`.

### Fixes

- Switching component refreshed the page's values without laying it out again: rows from the
  previous component stayed on screen.
- The "Choice group" switch was not clickable — it belonged to the wrong box.
- Two empty lists were written as `{}` instead of `[]` in the project file.

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
