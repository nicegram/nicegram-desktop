# Nicegram Desktop

Nicegram Desktop is an independent Telegram client based on
[Telegram Desktop](https://github.com/telegramdesktop/tdesktop). This repository
contains the desktop client fork and Nicegram-specific client work; it is not a
guarantee that every source-visible feature is enabled in a public release.

- Product: [nicegram.me](https://nicegram.me)
- Download: [official Nicegram downloads](https://nicegram.me/download)
- User documentation: [Nicegram Help Center](https://wiki.nicegram.me)
- Upstream: [Telegram Desktop](https://github.com/telegramdesktop/tdesktop)

Nicegram is not affiliated with Telegram.

## Nicegram additions visible in this repository

The current source contains desktop work for:

- AI-assisted composition and tone controls;
- rich-message editing and transport;
- communities and related navigation/data surfaces;
- ephemeral bot messages;
- text recognition in the media viewer.

Several of these features rely on a matching extended server contract. Until a
released client and service are verified together, describe them as
**source-visible**, not generally available. Mobile Nicegram features do not imply
Desktop parity.

## Build

The project retains Telegram Desktop’s build system and platform documentation:

- [Windows](docs/building-win.md)
- [macOS](docs/building-mac.md)
- [GNU/Linux with Docker](docs/building-linux.md)

Clone with submodules before building:

```bash
git clone --recursive https://github.com/nicegram/nicegram-desktop.git
cd nicegram-desktop
```

Use your own Telegram API credentials and application identity. Some Nicegram
services or production configuration are not provided by the public client source.

## License

The project is published under GPLv3 with the OpenSSL exception; see
[`LICENSE`](LICENSE). Third-party components retain their respective licenses.
