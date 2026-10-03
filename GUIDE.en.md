*[Version française : [GUIDE.md](GUIDE.md)]*

# Getting started with XPackager

This guide starts from nothing: you have just downloaded XPackager, you have never opened
it, and what you want in the end is a `.pkg` that anyone can install with a double click
without macOS getting in the way.

Read it in order. The long part is not XPackager — it is Apple.

---

## 1. Install and open

Double-click `XPackager.pkg`. The installer puts the application in `/Applications` and
offers you, optionally, the `xpackagerbuild` command line tool. Take it: it comes in handy
later, to build without opening the interface.

On first launch, XPackager does not show its empty project straight away. It opens a sheet
first: **Environment check**.

It exists because everything that can block you is **outside** XPackager — Apple's tools,
the certificates in your keychain, your notarization profile — and because those gaps,
otherwise, only show up after the fact: the build starts, runs for two minutes, and fails
right at the end on a refusal from Apple.

Five lines:

| Check | What it looks at |
|---|---|
| Xcode tools | is `notarytool` installed |
| Package building tools | `pkgbuild`, `productbuild`, `installer` |
| Developer ID Application certificate | signs the software |
| Developer ID Installer certificate | signs the package |
| Notarization profile | your Apple credentials stored in the keychain |

On a fresh machine, expect five crosses. That is normal. The sections below turn them into
ticks, in that order.

The sheet stays available at any time from **Help ▸ Check environment…**, and the **Check
again** button re-runs it without leaving the application. Keep it open throughout what
follows: every step you clear shows up there immediately.

---

## 2. Apple's tools

If the first line is red, install **Xcode** from the Mac App Store. It is big (several
gigabytes) but it is the safe route: `notarytool` lives inside it.

The command line tools alone are sometimes enough:

```bash
xcode-select --install
```

After installing, **Check again**. The first two lines should turn green. `pkgbuild`,
`productbuild` and `installer` ship with macOS and should already be there.

---

## 3. The certificates

This is the longest step, and the one people get wrong most often. Take your time.

### 3.1 The account

You need a paid **Apple Developer Program** account, renewed every year. A free Apple ID
does not entitle you to « Developer ID » certificates.

A second condition, often forgotten: only the **Account Holder** can create a Developer ID
certificate. If you are a member of a team without that role, the button is there but the
certificate type will not be offered to you.

### 3.2 Four certificates that look alike

This is where half of all notarization failures are decided.

| Certificate | What it does | Notarizable |
|---|---|---|
| **Developer ID Application** | signs an `.app`, a binary, a dylib | **yes** — this is the one you need |
| **Developer ID Installer** | signs a `.pkg` distributed outside the App Store | **yes** — you need this one too |
| Apple Development | local debugging on your own machines | **no** |
| 3rd Party Mac Developer Installer | Mac App Store submission | not applicable |

The trap: in a signing menu, `Apple Development: Your Name (XXXXXXXXXX)` and
`Developer ID Application: Your Name (XXXXXXXXXX)` look very much alike. The first produces
software that Apple **will refuse** to notarize. The environment check says so explicitly
when all it finds is a development certificate.

You need **both** Developer IDs: one for the software, one for the package.

### 3.3 Creating them — through Xcode

The shortest route:

1. Xcode ▸ **Settings…** ▸ **Accounts**
2. Add your Apple ID if it is not there
3. Select your team, then **Manage Certificates…**
4. The **+** button at the bottom left → **Developer ID Application**
5. Do it again: **+** → **Developer ID Installer**

Xcode generates the private key, asks Apple for the certificate and installs it in your
login keychain. Nothing else to do.

### 3.4 Creating them — through the portal

If you would rather go through the website, or if Xcode refuses:

1. **Keychain Access** ▸ menu *Keychain Access* ▸ *Certificate Assistant* ▸ *Request a
   Certificate From a Certificate Authority…*
2. Enter your email address, leave the authority's address empty, tick **Saved to disk**.
   You get a `.certSigningRequest` file.
3. On [developer.apple.com](https://developer.apple.com/account/resources/certificates),
   **Certificates** section, **+** button, choose **Developer ID Application**, upload the
   `.certSigningRequest`, download the `.cer` you get back and double-click it.
4. Do it again for **Developer ID Installer**.

Two warnings:

- **The private key never leaves the machine that produced the request.** A certificate
  downloaded on another Mac will be of no use. To work on several machines, export the
  certificate + key pair as a `.p12` from Keychain Access.
- The number of Developer ID certificates your account may hold is **limited**, and
  revoking a certificate invalidates the signatures made with it. Do not create them
  lightly and do not revoke anything just to « tidy up ».

### 3.5 Checking

In XPackager: **Check again**. The two certificate lines turn green and show the full name
found.

In the Terminal, if you want the raw list — this is exactly what XPackager reads:

```bash
security find-identity -v
```

You should see two lines containing `Developer ID Application:` and
`Developer ID Installer:`, followed by your name and your team identifier in parentheses.
Note that ten-character identifier: it is your **Team ID**, and the next step needs it.

---

## 4. The notarization profile

Signing is no longer enough. Since macOS 10.15, downloaded software must also be
**notarized**: sent to Apple, analysed, then given a ticket. Without that, your installer
opens on a warning that puts most people off.

`notarytool` needs your credentials. You do not type them every time: you store them once
and for all in the keychain, under a name — the **profile**.

### 4.1 An app-specific password

Your real Apple password will not do. You need a dedicated one:

1. Open [account.apple.com](https://account.apple.com/account/manage) — XPackager offers
   the **Generate an app-specific password…** link in its Settings.
2. **Sign-In and Security** section ▸ **App-Specific Passwords**
3. Create one, name it « XPackager » for instance
4. **Copy it right away**: it is shown only once, in the form `abcd-efgh-ijkl-mnop`.

### 4.2 Storing the profile

In XPackager: **Settings** (⌘,) ▸ **Notarization** tab.

| Field | What goes in it |
|---|---|
| Profile name | whatever you like — « XPackager » by default |
| Apple ID | the address of your developer account |
| Team ID | the ten characters in parentheses from your certificates |
| App-specific password | the one you have just created |

**Create / update the profile** button. XPackager calls
`xcrun notarytool store-credentials`: the password goes into the keychain and is **not**
kept by the application.

The **Verify** button asks Apple to confirm that the profile works.

The profile name is the one you will give in every project. Check the environment sheet
again: the fifth line turns green.

---

## 5. A worked example: shipping an application

Everything is in place. Let us make a real installer, the one for an application that
belongs in `/Applications`.

### 5.1 Starting from a template

**File ▸ New from Template…** then **Application in /Applications**.

Five templates ship with the application: *Empty package*, *Application in /Applications*,
*Two-component distribution*, *Command line tool in /usr/local/bin*, *Plug-in in /Library*.
A template only prefills — everything stays editable.

The window opens on four pages, in the sidebar: **Settings**, **Components**,
**Requirements**, **Presentation**.

### 5.2 Settings page

| Field | Example |
|---|---|
| Package name | `MyApplication` |
| Signing identity | `Developer ID Installer: Your Name (XXXXXXXXXX)` |

The identity menu only offers the certificates found in your keychain. Below the field,
XPackager reminds you what each type implies: `Developer ID Installer` for a package that
installs with a double click, `3rd Party Mac Developer Installer` for a Mac App Store
submission.

Then, further down:

- **Re-sign the payload with Hardened Runtime before packaging**: tick it.
- **Identity (Developer ID Application)**: your application certificate.
- **Notarize after building**: tick it. The profile used is the one from Settings.

> **Why the « Re-sign » box is not optional in practice.** Many applications arrive signed
> ad hoc, without the Hardened Runtime, and sometimes with the
> `com.apple.security.get-task-allow` debugging entitlement. That is the default for
> applications produced by Xojo. Apple refuses to notarize any of that. So XPackager
> re-signs a **copy** of the payload with `codesign --options runtime --timestamp` — your
> original files are untouched — and leaves no debugging entitlement in it.

### 5.3 Components page

A component is a choice offered at install time, and a location.

| Field | Example |
|---|---|
| Name (choice title) | `Application` |
| Identifier | `com.example.myapplication` |
| Version | `1.0` |
| Install location | `/Applications` |

The identifier follows the reverse domain name convention and must stay **stable from one
version to the next**: that is what lets macOS recognise an update.

Below it, the payload area: **drag your `.app` in from the Finder**. An application is
copied as is; a folder is imported with its whole tree. Select an item to adjust its
**Permissions** — `755` for an executable, `644` for an ordinary file.

The **Install options** (*User-changeable*, *Selected by default*, *Visible in the custom
list*) only make sense with several components: that is what produces the checkboxes of a
custom installation. With a single component, leave them alone.

### 5.4 Requirements page

- **Minimum macOS version**: `15.0` for instance. Left empty, no version is imposed.
- **Allowed architectures**: Apple Silicon, Intel, or both.

You can also add conditions — presence of a file, available memory — that block the
installation with a message.

### 5.5 Presentation page

These are the four screens the person installing will see: **Welcome**, **Read Me**,
**License**, **Conclusion**. A selector switches between them; the editor below is a rich
text editor — bold, italic, lists — and also accepts importing an existing `.rtf` or
`.txt`.

If your installer has to speak several languages, the globe menu, to the right of the
selector, adds a language and switches the editor to it. The **reference language** is
mandatory: it is the fallback when a translation is missing.

### 5.6 Building

**File ▸ Build…** (⌘B). A log window shows every command it runs: re-signing the payload,
`pkgbuild` per component, `productbuild` for assembly and signing, `notarytool submit
--wait`, then `stapler staple`.

Allow one to two minutes: Apple takes that time, not XPackager. The line `status: Accepted`
followed by `The staple and validate action worked!` concludes a successful build.

### 5.7 Checking before you distribute

Three commands, on the `.pkg` you produced:

```bash
spctl -a -vvv -t install MyApplication.pkg
```

Must answer `accepted` and `source=Notarized Developer ID`. That is Gatekeeper's verdict,
the one that counts.

```bash
xcrun stapler validate MyApplication.pkg
```

Confirms the ticket is stapled — so that installing works even offline.

```bash
pkgutil --check-signature MyApplication.pkg
```

Shows the certificate chain and the line `Notarization: trusted by the Apple notary
service`.

To look inside without installing:

```bash
pkgutil --expand-full MyApplication.pkg /tmp/check
```

Finally, the only test that really counts: copy the package to another machine, through a
route that sets the quarantine flag (email, download), and install it for real.

---

## 6. The same thing without the interface

The command line tool builds the same package from the same project file:

```bash
xpackagerbuild MyProject.xpackager
```

Useful options: `-o` for the destination, `--notarize <profile>` to force a profile,
`--no-notarize` to skip notarization while testing — which brings a build back down to a
few seconds.

That is what to use from a delivery script or a build step.

---

## 7. When it fails

| Message | Cause | Remedy |
|---|---|---|
| `The signature of the binary is invalid` | ad hoc signature, not Developer ID | tick **Re-sign the payload with Hardened Runtime** |
| `The executable does not have the hardened runtime enabled` | same cause | same |
| `The executable requests the com.apple.security.get-task-allow entitlement` | debugging entitlement, left behind by Xojo among others | same — XPackager never keeps it |
| `The signature does not include a secure timestamp` | signed without `--timestamp` | same |
| Profile not found | the profile name matches nothing in the keychain | Settings ▸ Notarization ▸ **Verify** |
| No identity in the menu | certificate missing, or private key on another machine | section 3 |
| `A required agreement is missing or has expired` | an Apple agreement is waiting for your signature | developer.apple.com ▸ Agreements |
| `Team is not yet configured for notarization` | developer agreement not signed | developer.apple.com ▸ Agreements |

To read the detail of a refusal from Apple:

```bash
xcrun notarytool log <submission-id> --keychain-profile <profile>
```

The submission identifier appears in the build log, right after `Submission ID received`.

---

## In short

1. Install Xcode
2. Create the **Developer ID Application** *and* **Developer ID Installer** certificates
3. Create an app-specific password and store the notarization profile
4. Check that the five lines of **Help ▸ Check environment…** are green
5. Build

Steps 1 to 3 are done once. After that, a signed and notarized package takes a ⌘B and two
minutes of waiting.
