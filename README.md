# ayugram-plengine-git

Arch Linux package for [AyuGramDesktop-PLEngine](https://github.com/okhsunrog/AyuGramDesktop-PLEngine) — AyuGram Desktop with the cross-platform PLEngine plugin system — built from the `linux-plengine` branch in CI and installed with `pacman -U`.

`provides=ayugram-desktop` / `conflicts=ayugram-desktop`, so it takes the place of the AUR / chaotic-aur package.

## Why CI

It is a `-git` package linked against a pile of C++ libraries with unstable ABIs (`abseil-cpp`, `protobuf`, `libtg_owt`, Qt). Every soname bump on the laptop leaves it broken until it is rebuilt:

```
AyuGram: error while loading shared libraries: libabsl_strings.so.2605.0.0: cannot open shared object file
```

Rebuilding locally takes a long time and pins the laptop. The CI job builds in a clean `archlinux:base-devel` container against *current* repo packages, installs the result, and checks `ldd /usr/bin/AyuGram` for unresolved libraries before uploading — so a rebuild is one click and a download.

All dependencies are in `core`/`extra` (including `libtg_owt`); no AUR helper is needed in the container.

## Build / install

Via CI (the usual route): trigger **Run workflow** on the `build ayugram` action (or push to `main`), then:

```sh
gh run download <run-id> --repo okhsunrog/ayugram-plengine-git -D ../ci-artifacts/<run-id>-ayugram-plengine-git-main
sudo pacman -U ../ci-artifacts/<run-id>-ayugram-plengine-git-main/ayugram-plengine-git-pkgs/*.pkg.tar.zst
```

Locally:

```sh
makepkg -si            # _jobs=10 by default; override with _jobs=N makepkg -si
```

The package is only as fresh as the repo snapshot the runner had at build time. If the laptop is *newer* than the runner (a soname bump landed in the repos between the run and the `pacman -U`), rebuild again — the check in CI is against the container, not the laptop.

## Knobs

- `_branch` (default `linux-plengine`) — which branch of AyuGramDesktop-PLEngine to build
- `_jobs` (default `10`) — parallelism for both cmake builds; CI sets it to `nproc`

`pkgver` is derived by `pkgver()` from `git describe`, so the value in the PKGBUILD is just the last one that was built; makepkg updates it in place.

## Stopgap without rebuilding

When only the abseil soname moved and there is no time to rebuild, the old libraries can sit next to the new ones (sonames differ, nothing conflicts). Symlinking the new `.so` under the old name does **not** work: abseil mangles the LTS date into every symbol (`absl::lts_20260526::…`).

```sh
curl -O https://archive.archlinux.org/packages/a/abseil-cpp/abseil-cpp-<old-ver>-x86_64.pkg.tar.zst
tar --zstd -xf abseil-cpp-<old-ver>-x86_64.pkg.tar.zst --wildcards 'usr/lib/*.so.<old-soname>.0.0'
sudo install -m644 usr/lib/*.so.<old-soname>.0.0 /usr/lib/
```

Those files are untracked by pacman — `sudo rm /usr/lib/*.so.<old-soname>.0.0` after installing a proper rebuild.

## License

Package recipe: GPL-3.0-or-later, same as upstream.
