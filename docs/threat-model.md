# Threat Model — Telepathy-Community

**Owner:** Al Baker (al@prose.ltd) · **Last reviewed:** 2026-08-13 · **Review:** On any release, or on a change to packaging or dependencies

This document is public, because this repository is. It describes only what is already
visible here or on PyPI.

## System Overview

Telepathy is a Python command-line OSINT toolkit for collecting and analysing public
Telegram data. This repository is its **public release**: source under `src/telepathy/`,
packaging metadata in `setup.py`, and the artefact published to PyPI as `telepathy`.

Three facts shape every threat below, and each is verified on `main` rather than assumed:

- **It is not actively maintained.** The README says so in its own words.
- **The source on `main` does not parse.** `src/telepathy/telepathy.py` raises a
  `SyntaxError` at line 1743 — a class definition with no body — so install-from-source is
  broken. Verified 2026-08-13 with `ast.parse`.
- **Built distributions are committed to the repository.** `dist/` holds wheels and
  sdists for 2.2.50, 2.2.58 and 2.3.2, and `build/lib/telepathy/` holds a second copy of
  the sources.

It runs entirely on the user's own machine, against credentials the user supplies. This
project operates no server and holds no user data.

## Assets

| Asset | Why it matters |
|---|---|
| The **`telepathy` name on PyPI** | Whoever controls it can ship code to everyone who types `pip install telepathy` |
| **Committed artefacts in `dist/`** | Downloadable directly from a public repository, and not reproducible from a source tree that does not parse |
| **The published `main` branch** | The install-from-source path people are told to use |
| **The reputation of the tool** | Its users are journalists and researchers; a malicious release harms the people it is used to protect |
| **Users' Telegram credentials and session files** | Held by the user, never by this project, but handled by this code on their machine |

## Threats

Each entry names who, by what path, and to what end. Mitigations are paired to the threat
rather than listed apart, because a list of mitigations nobody can map to a threat is not
evidence that the threat is covered.

- **T1 — Someone with write access, or who obtains the PyPI token, publishes a malicious
  release.** Users installing or upgrading run attacker code with access to their Telegram
  session. *Mitigation:* publishing is manual and infrequent; PyPI account controls are the
  only real barrier. **Not mitigated in this repository** — nothing here gates a release.
- **T2 — An attacker registers a similarly-named package and relies on a typo or a
  search-engine result.** Same outcome as T1, without needing access to anything.
  *Mitigation:* none available from this repository. Listed because it is the most likely
  path for an unmaintained tool with a recognisable name.
- **T3 — A user downloads a wheel from `dist/` rather than from PyPI.** They receive an
  artefact that no current process rebuilds or verifies, and which cannot be reproduced
  from `main`, because `main` does not parse. *Mitigation:* the README directs users to
  `pip install telepathy`. The artefacts remain, and removing them is the real fix.
- **T4 — A dependency is compromised upstream.** `setup.py` requires
  `telethon == 1.36.0` while `requirements.txt` pins `Telethon==1.25.2`, so what a user
  gets depends on which install path they followed. *Mitigation:* Dependabot is enabled at
  the organisation level; there is **no `.github/dependabot.yml`** in this repository, and
  no automated dependency scan runs here.
- **T5 — A vulnerability is reported and nobody reads it.** `setup.py` publishes
  `author_email` as a personal address of a former maintainer, and the README states that
  external bug reports are not currently triaged. A reporter follows the published contact
  and reaches nobody. *Mitigation:* the README names **al@prose.ltd** as the escalation
  route; the package metadata still does not.
- **T6 — CI runs no security check on this repository at all.** A public repository
  receives an untrusted pull request and nothing scans it. *Mitigation:* none currently.
  The `security-suite` workflow present here **cannot run**: it calls a reusable workflow
  in a private repository, which GitHub will not resolve for a public caller, so every run
  fails before any job starts.

## Mitigations

The paired mitigations above are the substance. Estate-wide controls that do reach here:

- GitHub secret scanning and push protection, applied organisation-wide.
- Dependabot alerts at the organisation level.
- Branch protection on `main`.

## Decisions

- **The tool is unmaintained, and that is accepted rather than hidden.** The README states
  it plainly at the top. This document does not treat the lack of a fix as a gap to close.
- **The `dist/` artefacts and `build/` copy are left in place for now.** Removing them is
  the correct fix for T3 and is a change to a public repository's history-facing content,
  so it is recorded as a decision rather than made silently here.
- **No security scanning is a known state, not an oversight.** See T6; the four available
  remedies are set out in issue #104 and the choice is the owner's.
