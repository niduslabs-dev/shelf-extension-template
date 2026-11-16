# Shelf extension template

Start a Chromium extension for [Shelf](https://shelf.niduslabs.dev), the registry for Helium and ungoogled-chromium. Your code goes in `extension/`. A workflow packs it, signs it with your key and releases it.

Choose **Use this template** to make your own repo. It must be public.

## 1. Make a signing key

```sh
openssl genrsa -out key.pem 2048
```

Your extension's ID comes from this key, so keep it safe and never commit it. Store it outside the repo, in a password manager. If you lose it, you can't release updates.

## 2. Get your extension ID

```sh
openssl rsa -in key.pem -pubout -outform DER 2>/dev/null | shasum -a 256 | head -c 32 | tr 0-9a-f a-p; echo
```

That's the first 32 hex characters of the SHA-256 of your public key, with `0-9a-f` written as `a-p`. Chromium makes the ID the same way.

## 3. Fill in your ID

- In `extension/manifest.json`, set `update_url` to `https://shelf.niduslabs.dev/updates/<your ID>.xml`. Shelf serves your updates from there, so it never changes.
- In `shelf.json`, replace `YOUR_EXTENSION_ID` with your ID and `YOUR_GITHUB_LOGIN` with the GitHub accounts allowed to publish it. It proves you control this repo.

## 4. Add the key as a secret

In your repo, open Settings, then Secrets and variables, then Actions. Add a secret named `SHELF_SIGNING_KEY` holding the whole `key.pem` file. Or with the GitHub CLI:

```sh
gh secret set SHELF_SIGNING_KEY < key.pem
```

## 5. Release

1. Set `version` in `extension/manifest.json`. Each release needs a higher one.
2. In the Actions tab, run the **Release** workflow.
3. It creates Release `v<version>` with the signed `.crx`. Its summary has the Shelf pull request: `release.json` and, the first time, `listing.json`. Open it with the links there.

The workflow stops if `update_url` is wrong, the manifest isn't Manifest V3, or a `.pem` or `.key` file is in `extension/`. Files starting with a dot are left out.

## 6. Review

A Shelf maintainer reviews every pull request. Its automatic check confirms:

- the `.crx` is signed by the key behind your ID, and matches the `sha256` in `release.json`;
- the manifest is Manifest V3, its version matches and goes up, and `update_url` points at Shelf;
- `shelf.json` here lists you as a publisher;
- the `.crx` is an asset of a Release in this repo.

It also lists every permission and site your extension can reach. New listings and new permissions get a closer look.

## For your users

Helium and ungoogled-chromium only install extensions from outside the Chrome Web Store with a flag on. Open `chrome://flags/#extension-mime-request-handling` and set it to "Always prompt for install". Then the Install button on Shelf works, and updates arrive on their own.
