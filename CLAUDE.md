# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A fork of [PAMGuard/PAMGuard](https://github.com/PAMGuard/PAMGuard) (GPLv3, Maven, Java 21,
Swing/JavaFX). It carries a small set of patches for **headless batch processing** (e.g. in Docker)
and publishes a pinned fat jar as a GitHub Release asset. Keep patches small and rebasable.

## Remotes & branches

- `origin` = this fork, `upstream` = PAMGuard/PAMGuard (push to upstream is disabled).
- `fugro` — long-lived patch branch; all fork work goes here. Rebased onto upstream release tags (`V<version>`)
  when upstream releases.
- `main` — upstream release tag plus the release workflow (kept there for visibility;
  tag builds use the workflow file from the tagged commit).
- `fix/*` — one branch per upstream PR, based on `upstream/main`, containing exactly one
  cherry-picked commit. **Never open a PR from `fugro`** — it would carry all fork patches.
- `git log upstream/main..fugro` lists the patches currently carried.

## Build

```bash
mvn -B clean install shade:shade -DskipTests   # fat jar -> target/Pamguard-<version>.jar
mvn compile -DskipTests                        # quick compile check
```

Java 21 is required (the code uses Java 21 APIs; the pom compiles with `<release>21</release>`).

## Releasing

Push a `fugro-v<upstream-version>-<iteration>` tag; `.github/workflows/release-jar.yml` builds
the jar and attaches it to a GitHub Release. See `RELEASING.md` for the full flow.

## Headless usage

```bash
java -jar Pamguard-<version>.jar PAMGuard \
  -psf settings.psfx -wavfilefolder <audio-dir> \
  -databasefile out.sqlite3 -binaryfolder <binary-dir> \
  -nogui -nosplash -noDialogs -autostart -autoexit [-debugout]
```

- AWT/Swing is still initialised headless, so containers need Xvfb (or equivalent).
- A `SailDaqJNI` "Unsatisfied Link Error" is the hardware DAQ card and is harmless for
  file-based processing.
- Reference: https://github.com/PAMGuard/PAMGuard/wiki/Command-Line-Options

## Conventions

- Keep changes minimal and upstreamable where possible; GUI behaviour must stay unchanged —
  gate headless-only behaviour on `PamGUIManager.getGUIType() == PamGUIManager.NOGUI` or the
  `-autoexit` global argument.
- No local file paths, company-internal hostnames or infrastructure details in code, commits
  or docs — this repository is public.
