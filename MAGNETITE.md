# Magnetite's fork of cef-rs

This is the fork of [tauri-apps/cef-rs](https://github.com/tauri-apps/cef-rs)
that the [Magnetite browser](https://github.com/magnetite-browser/magnetite)
builds with. It exists for one reason: the bindings must be for the CEF
version Magnetite ships. The `cef` crate on crates.io is bound to the version
upstream published last, and `cef-dll-sys` only accepts a CEF archive that is
not newer than that. With this fork the browser can move to a new CEF (and
its Chromium security fixes) without waiting.

The fork stays as close to upstream as it can. Fixes and features belong
upstream; what is changed here is the CEF version and the bindings generated
for it. The licence is upstream's (Apache-2.0 or MIT).

## Branches

- `magnetite` (the default): what the browser uses. The browser pins a
  commit of it in `crates/engine/Cargo.toml`.
- `dev`: a copy of upstream's `dev`, never changed here.

## A new CEF version

1. Bring `dev` up to upstream's and merge it into a branch off `magnetite`.
   Merge, do not squash, so the next merge knows what it already has.
2. Set the workspace version in `Cargo.toml` to `<crate version>+<CEF version>`.
3. Generate the bindings:
   `cargo run -p update-bindings -- -b -d -t aarch64-apple-darwin`.
   It downloads the official archive of that version into `target/` and
   writes the file of that one target only. For a patch release of CEF only
   three constants change (`CEF_VERSION`, `CEF_VERSION_PATCH`,
   `CHROME_VERSION_PATCH`): set them by hand in the other targets' files in
   `sys/src/bindings`. If more than these changed, generate each target's
   file with `-t <target>` (so far only done for the target above).
4. Open a pull request against `magnetite` and merge it with a merge commit.
5. In the browser: pin the new commit, and the CEF release in `scripts/cef.sh`.

## Checks

`.github/workflows/magnetite.yml` builds the workspace and runs its tests on
Linux and macOS against the official CEF archive of the bound version.
Upstream's own workflows are for `dev`, and those that publish only run in
upstream's organisation. CodeRabbit reviews pull requests (`.coderabbit.yaml`).

Security issues of the fork: security@magnetitebrowser.org. Issues of cef-rs
itself belong to [upstream](https://github.com/tauri-apps/cef-rs/issues).
