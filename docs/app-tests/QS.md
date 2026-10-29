# QS-Protokoll — Performance & Reliability: Deployment-Zeiten in Produktion

> **Projekt:** OpenStack AppStore
> **Qualitätsziel:** Reliability & Performance — VMs starten zuverlässig, keine Downtimes während Vorlesungen
> **Gemessen wird:** Zeit vom Klick auf „Deploy" bis das Deployment im Frontend als **Active** angezeigt wird.

---

## 0. Allgemeine Angaben

| Feld | Eintrag |
|---|---|
| Datum | 28.10.2016 |
| Durchführende Person(en) | Sebastian & Erik |
| Umgebung | Staging |

---

## 1. Übersicht aller Apps (Prod)

Je App **ein** Deployment mit minimaler Konfiguration (ein Team mit neun Studenten (eine VM pro App), Standard-Flavor).

| App | OS | Packer-Build vorhanden | Anzahl VMs | Flavor (nur wenn wählbar) | Dauer „Deploy" → „Active" (mm:ss) | Erfolg? (✔/✘) | Auffälligkeiten |
|---|---|---|---|---|---|---|---|
| `Ubuntu-App` | Linux | ja | 1| – | 00:18| Ja | – |
| `Jupyter-Notebook` | Linux | ja | 1| – | 00:17|Ja | – |
| `pgAdmin` | Linux | ja |1 | – | 00:21| Ja| – |
| `Web-LaTeX` | Linux | ja | 1| – | 00:25|Ja | 6 min wenn Image nicht vorhanden |
| `Online-IDE` | Linux | ja | 1| mb1.large| 00:25| Ja| – |
| `Kubernetes` | Linux | ja | 1| gp1.large| 00:32| Ja| – |
| `Windows-App` | Windows | nein |1 | win11.medium| |Ja | |
| `windows-spss` | Windows | nein | 1|win11.medium | 00:23| Ja| – |

> Flavor ist nur bei `Online-IDE`, `Windows-App` und `windows-spss` im Wizard wählbar (Defaults: `gp1.small` = 1 vCPU / 2 GB RAM / 10 GB Disk bzw. `win11.medium` = 2 vCPU / 8 GB RAM / 80 GB Disk). Die anderen Apps haben einen festen Flavor (`gp1.small`).

---

## 2. Detailtest — Ubuntu-App

### 2.1 Testaufbau

| Feld | Eintrag |
|---|---|
| App-Version (Git-Tag) | v1.2.4|
| Teams / Nutzer pro Team | 1/5|
| Anzahl VMs | 1|
| Flavor | `gp1.small` (default) |
| vCPU | 1 |
| RAM | 2 GB (2048 MB) |
| Disk | 10 GB |
| Image (Name, Größe in GB) | ee6dfd3a-6605-4d2b-8f94-71dd8ecc02ef-d6a400ad, 10GB |

### 2.2 Messreihe (3 Durchläufe)

| Lauf | Image neu gebaut (Packer)? | **Dauer (mm:ss)** |
|---|---|---|
| 1 (Kaltstart, Image existiert noch nicht) | ja | 02:22 |
| 2 (Image vorhanden) | nein | 00:18 |
| 3 (Image vorhanden) | nein | 00:17 |

### 2.3 Funktionsprüfung nach Deployment

| Prüfung | Ergebnis (✔/✘) | Anmerkung |
|---|---|---|
| SSH-Login mit Nutzer-Account funktioniert | ✔ | – |
| Linux-Account pro Nutzer wurde angelegt | ✔ | – |
| Infrastruktur-Panel im UI zeigt korrekte IP / Outputs | ✔ | – |
| Terminal reagiert flüssig | ✔ | – |
| Destroy / Löschen funktioniert (Dauer: 00:10) | ✔ | – |

### 2.4 Ressourcenverbrauch (während VM unter Last)


| Messwert | Ergebnis |
|---|---|
| CPU | 1 vCPU, AMD EPYC-Genoa (KVM) |
| CPU sysbench (1 Thread) | 1742 events/s |
| RAM gesamt / belegt vor Test | 1,9 GiB / 360 MiB (1,6 GiB verfügbar) |
| RAM-Durchsatz (sequentiell) | 28.454 MiB/s |
| Disk gesamt / belegt | 8,7 GB / 2,4 GB (28 %) |
| Disk zufällig lesen (fio, 4k) | 52,3k IOPS, 204 MiB/s |
| Disk zufällig schreiben (fio, 4k) | 22,4k IOPS, 87,6 MiB/s |
| Disk sequentiell schreiben (dd) | 2,4 GB/s |



---

## 3. Fehler & Beobachtungen

| Nr. | App / Lauf | Fehlermeldung / Beobachtung | Behoben? Wie? | GitHub-Issue |
|---|---|---|---|---|
| 1 | Windows-App | Windows-VM schlägt fehl | Nein, noch nicht behoben | [#6](https://github.com/NextAppStore/Windows-App/issues/6) |

### Leitfragen

| Frage | Antwort |
|---|---|
| Gab es Timeouts (Terraform, SSH, OpenStack-API)? | Nein|


---

## Anhang: Benchmark-Skript (Abschnitt 2.4)

Quelle: [Issue #5](https://github.com/NextAppStore/.github/issues/5). Ausgeführt auf der Ubuntu-VM (`gp1.small`).

```bash
sudo apt update && sudo apt install -y \
  stress-ng sysbench fio iperf3 sysstat \
  dmidecode util-linux procps hdparm lshw


cat > ~/vm_bench.sh <<'EOF'
#!/usr/bin/env bash
set -u
OUT=~/vm_report_$(date +%Y%m%d_%H%M%S).txt
exec > >(tee "$OUT") 2>&1
log(){ echo -e "\n===== $* =====  [$(date +%T)]"; }

NPROC=$(nproc)
log "SYSTEM / BASELINE"
lscpu | grep -E 'Model name|^CPU\(s\)|Thread|Socket|MHz'
free -h; echo; df -h /; systemd-detect-virt

log "CPU – sysbench (Events/s, höher=besser), $NPROC Threads"
sysbench cpu --cpu-max-prime=20000 --threads=$NPROC --time=30 run \
  | grep -E 'events per second|total time'

log "CPU – Single-Core-Referenz (1 Thread)"
sysbench cpu --cpu-max-prime=20000 --threads=1 --time=15 run \
  | grep 'events per second'

log "RAM – Durchsatz (MiB/s), sequentiell schreiben"
sysbench memory --memory-block-size=1M --memory-total-size=10G \
  --threads=$NPROC run | grep -E 'transferred|per second'

log "DISK – fio zufälliges Lesen/Schreiben (IOPS + MiB/s)"
fio --name=randrw --rw=randrw --rwmixread=70 --bs=4k \
  --size=512M --numjobs=1 --iodepth=16 --runtime=30 --time_based \
  --direct=1 --ioengine=libaio --group_reporting \
  --directory=$HOME 2>/dev/null \
  | grep -E 'read:|write:|iops|BW='

log "DISK – sequentieller Durchsatz (dd, MB/s)"
dd if=/dev/zero of=$HOME/ddtest.bin bs=1M count=1024 oflag=direct 2>&1 | tail -1
rm -f $HOME/ddtest.bin $HOME/randrw*

log "FERTIG – Report: $OUT"
EOF
chmod +x ~/vm_bench.sh
~/vm_bench.sh
```
