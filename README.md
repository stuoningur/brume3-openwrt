# Brume 3 (GL-MT5000) – OpenWrt per GitHub Actions

Baut OpenWrt für den GL.iNet GL-MT5000 so nah wie möglich am Standard-Image
eines offiziell unterstützten Geräts: keine eigenen Dateien, keine eigenen
Patches, Paketauswahl = Gerätestandard + LuCI.

## Quelle

Der Brume 3 wird von keinem OpenWrt-Release unterstützt. Der Geräte-Support
existiert nur als PR [#24237](https://github.com/openwrt/openwrt/pull/24237)
gegen OpenWrt `main` (Kernel 6.18); das aktuelle Stable-Release 25.12.x nutzt
Kernel 6.12 und enthält ihn nicht.

Gebaut wird deshalb ein festgenagelter, geprüfter Stand:

| Teil | Stand |
|---|---|
| Quellbaum | `GLiNet-Tech/openwrt` @ `89a10ec` = OpenWrt `main` `19a15f9` (22.10.2026) + genau der PR-Commit |
| Feed packages | `c79462b` |
| Feed luci | `580b6b1` |
| Feed routing | `4b9891b` |

Das ist ein Snapshot von `main`, kein Stable-Release. Sobald der Brume 3 in
einem OpenWrt-Release enthalten ist, besser das offizielle Image nehmen.

Neuen Stand bauen: `SOURCE_REF` bzw. die Feed-Commits in
`.github/workflows/build.yml` anpassen – vorher den Diff gegen OpenWrt `main`
prüfen.

## Bauen

**Actions → „Brume 3 OpenWrt bauen“ → Run workflow** (oder Push auf `main`).
Unter **Settings → Actions → General → Workflow permissions** muss
**Read and write** gesetzt sein (für Release und Branch `build-logs`).

- `source_ref`: Commit im GL.iNet-Repo, Standard ist der geprüfte Stand.
- `all_kmods`: alle Kernelmodule mitbauen; sie liegen im Release als
  `kmods-und-target-pakete.tar.gz`. Dauert länger.

Ergebnis: ein Pre-Release unter **Releases** mit
`…gl-mt5000-squashfs-sysupgrade.bin`, `…factory.bin`, Manifest und
Prüfsummen. Die Prüfsummen stammen aus demselben Build und erkennen nur
Übertragungsfehler, keine Manipulation.

## Flashen

- Von der GL.iNet-Stock-Firmware: `sysupgrade.bin` über die Upgrade-Seite der
  Stock-Oberfläche einspielen, **ohne** Einstellungen zu behalten
  (bzw. `sysupgrade -n`).
- Von einem OpenWrt-Image: normales Sysupgrade.
- **Notfall:** Reset-Taste beim Einschalten ~5 s halten → U-Boot-Failsafe unter
  http://192.168.1.1 (PC direkt an einen LAN-Port, feste IP 192.168.1.2) →
  Image hochladen.

## Nachinstallation einzelner kmods

Pakete aus `kmods-und-target-pakete.tar.gz` **desselben Builds** auf den
Router kopieren und mit `apk add /tmp/<paket>.apk` installieren. Das Image
enthält den öffentlichen Schlüssel seines Builds, die Signatur wird also
normal geprüft. Paketfeeds aus dem Internet passen nicht zum eigenen Kernel –
kmods immer aus dem eigenen Build nehmen.
