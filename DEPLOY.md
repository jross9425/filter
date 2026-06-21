# Quiet Filter — Non-Removable Lock (deployment kit)

This makes the extension **unremovable from the Chrome UI**: no Remove button, no
disable toggle, no "drag to trash." It's the OS-policy / force-install path.

Your permanent extension ID is already filled into every file here:

    fpobbapdpakdohbkhenlommhahanepmn

Files:
- `quiet-filter.crx` ............ the packed, signed extension (force-installed copy)
- `quiet-filter-SIGNING-KEY.pem`  the key that defines the ID. KEEP IT SAFE & PRIVATE.
                                  You need it to sign future updates and keep the same ID.
- `update.xml` .................. the update manifest Chrome's policy points at
- `windows-lock.reg` ............ apply on Windows
- `macos-lock.mobileconfig` ..... apply on macOS
- `linux-lock.json` ............. apply on Linux

You only have to fill in ONE thing yourself: where you host the files (`YOUR-HOST`).

---

## Step 1 — Host two files over https

Chrome's policy points at `update.xml`, which points at `quiet-filter.crx`. Both
must sit at an https URL the browser can reach. Easiest options, no server needed:

- **GitHub** (free): put both files in a repo. Use the raw URLs, e.g.
  `https://raw.githubusercontent.com/<you>/<repo>/main/update.xml` and
  `.../main/quiet-filter.crx`. (Or turn on GitHub Pages.)
- **Object storage**: a public Cloudflare R2 / Backblaze B2 / S3 bucket.
- **A box on your LAN**: any static https file server.

Then edit `update.xml` — set `codebase` to the full https URL of
`quiet-filter.crx`. (The ID and version are already correct.)

## Step 2 — Point the policy at your `update.xml`

In whichever policy file matches your OS, replace `https://YOUR-HOST/update.xml`
with the real https URL of your hosted `update.xml`. Then:

- **Windows** — right-click `windows-lock.reg` → run as Administrator → restart Chrome.
- **macOS** — replace the two `REPLACE-WITH-UUID-x` values (`uuidgen` in Terminal,
  run twice), double-click the `.mobileconfig`, install the profile (enter your Mac
  password), restart Chrome.
- **Linux** — `sudo cp linux-lock.json /etc/opt/chrome/policies/managed/` → restart Chrome.

## Step 3 — Verify

Go to `chrome://policy` → **Reload policies** → `ExtensionSettings` shows **OK**.
Then `chrome://extensions`: Quiet Filter should be installed and show
"Installed by enterprise policy" with **no Remove or disable control**.

Pair this with the in-extension PIN lock (`lock.js`, already wired into your build)
so the popup can't be neutered either. Together: can't remove it, can't turn it off.

---

## Updating later (without losing the ID)

1. Edit your unpacked dev copy, test.
2. Bump `version` in `manifest.json` AND in `update.xml`.
3. Re-pack signing with **the same** `quiet-filter-SIGNING-KEY.pem`
   (chrome://extensions → Pack extension → supply this key).
4. Replace the hosted `.crx`. Chrome pulls the update automatically.

---

## The honest ceiling (please read)

This stops casual and impulsive removal cold — there is no button to click. But:

- **You set the policy, so you can unset it.** On a machine where you hold the
  admin/root password, you can always delete the registry key / profile / json and
  the lock is gone. Force-install is also documented by Google as "best efforts" —
  some OSes can't fully defend against external removal.
- The genuine wall is the same one every accountability tool relies on: **stop
  being the one who can undo it.** Browse from a standard (non-admin) account with
  the admin password out of easy reach, or have someone you trust hold it / install
  the policy via an MDM you don't control. The long PIN cooldown plus a person who'd
  notice the change is what actually holds.

So: locked against your future impulsive self — yes, strongly. Locked against
present-you-with-the-password — only if you hand that password away.
