*[Version française : [README.md](README.md)]*

# XPackager

Builds **macOS installers** — `.pkg` files that are signed, notarized and stapled — from a
project you set up in a graphical interface, without writing a line of `pkgbuild` or
`productbuild`.

Targets **macOS 15** and later. Free — [supported by your donations](#support-the-project).

This repository carries the **documentation** and the **released builds**. The application's
source is not here.

---

## Download

Everything is in the [latest release](../../releases/latest):

| File | What it is |
|---|---|
| `XPackager-x.y.z.pkg` | The application and, optionally, its command line tool |

The package installs with a double click. It is itself built by XPackager, signed and
notarized — it is its own demonstration.

## What it does

- **The payload** is assembled by dragging from the Finder: an `.app`, a folder with its
  whole tree, a command line tool. Permissions can be set per item.
- **Several components** produce a custom installation, with its checkboxes, its
  preselected choices and its pre- and post-install scripts.
- **Requirements** block installation outside their range: minimum macOS version, allowed
  architectures, presence of a file, available memory.
- **The installer's four screens** — Welcome, Read Me, License, Conclusion — are written in
  a rich text editor, or imported from an existing `.rtf`. Background image included.
- **A multilingual installer** is described in the same project: every screen, the title and
  the choice labels are translated language by language, with a reference language used as
  a fallback and a translation progress indicator.
- **Signing and notarization** follow the build. The payload is re-signed with the Hardened
  Runtime on a copy — your originals are untouched — then the package goes to Apple and
  comes back with its ticket stapled.
- **A command line tool**, `xpackagerbuild`, builds the same package from the same project
  file, for a delivery script or continuous integration.

## Getting started

The [guide](GUIDE.en.md) starts from nothing: the application has just been downloaded and
never opened. It covers the first-launch environment check, obtaining the certificates from
Apple — the long part, and the one people get wrong most often —, storing a notarization
profile, then a complete worked example up to the verified package.

## Support the project

XPackager is **free**. A donation encourages what comes next if the tool serves you well —
€10, €20 or €50, or whatever you choose.

[![Donate with PayPal](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/donate/?hosted_button_id=J5GEC6T9ZX2XE)

Or by scanning this code:

<img src="assets/donate-qr.png" width="150" alt="PayPal donation QR code">

The button is in the application too: **About XPackager**, and in the **Help ▸ Donate…**
menu.

## What it is made with

XPackager is written in **Xojo**, on top of
**[VDSTools](https://github.com/Valdemar-VDSC/VDSTools-dist)** — native macOS window chrome,
sidebar, toolbar and controls, in pure Xojo.

## Getting help

A question, a problem: **support@vdsc.fr**, or this repository's [issues](../../issues).
