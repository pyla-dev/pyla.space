<p align="center">
  <img src="docs/logo_icon_400.png" width="120" alt="PYLA logo"/>
</p>

# PYLA — Protect Your Local Access

🌐 [pyla.space](https://pyla.space) · πύλαι ("the gates")

An open source firewall GUI for Ubuntu that makes **kernel-level network posture visible**: what's listening, what's exposed, what should never have been — in one screen, in plain language.

Built on the `ufw` / netfilter engine you already trust.
No accounts. No telemetry. No cloud.

> Hot gates weren't about taller walls — they were about knowing exactly where the gates are, and who passes through.

## Why

`ufw` is powerful, but it lives in a terminal most people were never handed the keys to. Security posture tools exist for datacenters and enterprise agents — nothing honest and local for the machine in front of you. PYLA closes that gap: the same kernel truth, made legible.

## How it works

- **Rust + Tauri 2 desktop app** for Ubuntu (tested on 26.04). It does not reinvent the firewall:
  it reads and drives the `ufw` / nftables engine already in the kernel.
- **Strict privilege separation.** The GUI runs as your user. Every root operation goes through a
  small helper (`pyla-helper`) authorised by a dedicated polkit action (`space.pyla.app.helper`,
  JSON in / JSON out, absolute paths, cleared environment, never a shell).
- **Exposure view, evaluated per address family.** Every TCP listener on the machine is assessed
  against the live nftables input chains: verdict per listener (`blocked`, `conditional`, `local`,
  `unknown`) with the exact matching rule, and for dual-stack sockets a separate IPv4 and IPv6
  verdict — because an IPv4 drop can mask an IPv6 opening.
- **Rule sets with preview and automatic rollback.** Apply shows a live diff first; after apply the
  state is re-read and, if it differs, the previous rules are restored (last 30 backups kept under
  `/var/lib/pyla/backups`).
- **CVE-aware hardening.** Ubuntu kernel CVEs are fetched from the Ubuntu Security API and mapped to
  subsystem mitigations (sysctl toggles, module blocklists) that persist across reboots in files
  clearly marked as tool-managed.

## Install

**Prebuilt package (Ubuntu 26.04, amd64):** grab the `.deb` from
[the latest release](https://github.com/pyla-dev/pyla/releases/latest) — it ships the app, the privileged
helper, the polkit policy, the desktop entry and the icons.

```sh
curl -LO https://github.com/pyla-dev/pyla/releases/latest/download/pyla_0.2.0_amd64.deb
curl -LO https://github.com/pyla-dev/pyla/releases/latest/download/pyla_0.2.0_amd64.deb.sha256
sha256sum -c pyla_0.2.0_amd64.deb.sha256          # must print "OK" before installing
sudo apt install ./pyla_0.2.0_amd64.deb
```

The code lives in [`pyla-dev/pyla`](https://github.com/pyla-dev/pyla). Requirements: Ubuntu with
UFW + nftables, `polkit`, and for building from source a stable Rust toolchain plus the
[Tauri 2 prerequisites](https://tauri.app/start/prerequisites/).

```sh
git clone https://github.com/pyla-dev/pyla.git
cd pyla
cargo test                                  # unit tests: no root, no network
cargo build --release                       # app + helper
sudo packaging/install-helper.sh            # privileged helper + polkit policy
packaging/install-desktop.sh                # launcher nel menu (no sudo)
packaging/make-deb.sh                       # in alternativa: pacchetto .deb
```

No telemetry, no accounts, no phone-home: the only outbound request is the Ubuntu Security API for
the CVE view.

## Contributing

Forks and pull requests welcome. Open an issue first for anything bigger than a typo — especially around security-relevant behavior.

## License

MIT — see [LICENSE](LICENSE).

Copyright (c) 2026 The PYLA Project Authors.
