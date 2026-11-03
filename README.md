# Brume 3 (GL-MT5000) – OpenWrt per GitHub Actions

Baut OpenWrt für den GL.iNet GL-MT5000 so nah wie möglich am Standard-Image
eines offiziell unterstützten Geräts: keine eigenen Dateien, keine eigenen
Patches, Paketauswahl = Gerätestandard + LuCI.

## Quelle

Der Brume 3 wird von keinem OpenWrt-Release unterstützt. Der Geräte-Support
existiert nur als PR [#24237](https://github.com/openwrt/openwrt/pull/24237)
gegen OpenWrt `main` (Kernel 6.18); das aktuelle Stable-Release 25.12.x nutzt
Kernel 6.12 und enthält ihn nicht.

Gebaut wird OpenWrt `main` auf einem festgenagelten Stand, plus die Patches
aus `patches/` (per `git am`):

| Patch | Inhalt |
|---|---|
| `0001` | GL.iNet-Commit `89a10ec` aus PR #24237, neu aufgesetzt auf `main` `46e6335` (Linux 6.18.60). Konflikte wie im Review beschrieben gelöst: dummy_dev-Hunks entfernt (jetzt upstream), `nf_flow_table_path.c` an 706-03/708 angepasst, 797-01 auf 795-11 aufgesetzt, qualcommax 0950 entfällt/0955 aufgefrischt |
| `0002` | Special-Tag-Gating (`TX_DMA_SPTAG_V3`, `MTK_GDMA_SPECIAL_TAG`) als eigener Patch, über `mtk_uses_dsa()` |
| `0003` | Tagger: MAC-Header nach dem Entfernen des Trap-Headers zurücksetzen |
| `0004` | DTS: Factory-Zellen ddns, sn-bak, sn, country-code |

Geprüft: Die komplette Kernel-Patchserie lässt sich ohne Fuzz und ohne
Offsets auf Linux 6.18.60 anwenden, für mediatek, realtek, ipq40xx,
qualcommax und ramips. Die betroffenen Treiber und das DTS kompilieren
fehlerfrei für arm64 (filogic-Konfiguration). **Nicht auf Hardware getestet**
– Hardware-Flow-Offloading zunächst ausgeschaltet lassen.

| Teil | Stand |
|---|---|
| Quellbaum | `openwrt/openwrt` @ `46e6335` (03.11.2026) + `patches/` |
| Feed packages | `1f7abcb` |
| Feed luci | `ae19cdf` |
| Feed routing | `ea8633d` |

Das ist ein Snapshot von `main`, kein Stable-Release. Sobald der Brume 3 in
einem OpenWrt-Release enthalten ist, besser das offizielle Image nehmen.

Neuen Stand bauen: `SOURCE_REF` und die Feed-Commits in
`.github/workflows/build.yml` anpassen und prüfen, dass `patches/` weiterhin
ohne Fuzz anwendbar ist.

## Bauen

**Actions → „Brume 3 OpenWrt bauen“ → Run workflow** (oder Push auf `main`).
Unter **Settings → Actions → General → Workflow permissions** muss
**Read and write** gesetzt sein (für Release und Branch `build-logs`).

- `source_ref`: OpenWrt-`main`-Commit, Standard ist der geprüfte Stand.
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
