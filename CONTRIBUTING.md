# Contributing to Compass Heading Display

Thanks for taking the time to help improve Compass Heading Display.

This repository hosts the **documentation and public issue tracker** for the mod. The mod's source code is closed and distributed exclusively through the official GIANTS ModHub. We are not accepting code pull requests.

There are still three productive ways to contribute:

## 1. File a bug report

If something doesn't work the way the documentation says it should, please open an issue using the [Bug Report template](https://github.com/rocklandusa/FS25_CompassHeading/issues/new?template=bug_report.yml).

Useful bugs include all of:

- Your Compass Heading Display version (visible in the in-game mod menu).
- Your Farming Simulator 25 version.
- Whether the bug happens in single-player, on a dedicated server, or both.
- A short list of steps to reproduce the issue.
- The relevant lines from your `log.txt` (located at `Documents\My Games\FarmingSimulator2025\log.txt`).

A vague "the compass doesn't work" report is hard to act on — a 5-line repro plus log excerpt typically gets a fix into the next ModHub submission.

## 2. Request a feature

Open an issue using the [Feature Request template](https://github.com/rocklandusa/FS25_CompassHeading/issues/new?template=feature_request.yml). Describe the use case before the implementation — *what* you want to do, *why* you currently can't, and *who* it would help.

Feature requests are evaluated against the project's design principles:

- The compass should remain a non-intrusive HUD element — features that hide the player view or steal focus are out of scope.
- The public API must remain backwards-compatible.
- New marker styles, categories, animations, and events are welcome; anything that requires a hard dependency from third-party mods is not.

## 3. Ask for integration help (other mod authors)

Building a satellite mod that wants to place markers on the compass? Open an issue using the [Integration Help template](https://github.com/rocklandusa/FS25_CompassHeading/issues/new?template=integration_help.yml) and we'll point you at the right section of the [Developer Guide](DEVGUIDE.md).

If you're hitting a sharp edge in the API that the docs don't cover, that's a doc bug — file it under Integration Help and we'll update the guide.

## Code of conduct

Be respectful. Personal attacks, harassment, and off-topic disputes will result in the issue being locked. We are a small volunteer project; treating maintainers and other contributors well is the only price of admission.

## What we cannot accept

- **Pull requests adding or modifying code in this repository.** The mod source is closed. Documentation typo fixes are welcome via PR.
- **Reverse-engineered code** posted in issues or comments. Doing so is a violation of the mod's [LICENSE](LICENSE) and the issue will be deleted.
- **Support requests for other mods.** If a third-party mod is calling our API incorrectly and producing errors, please raise the issue on that mod's tracker first.
