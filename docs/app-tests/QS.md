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
| `Ubuntu-App` | Linux | ja | 1| – | 00:18| Ja | |
| `Jupyter-Notebook` | Linux | ja | 1| – | 00:17|Ja | |
| `pgAdmin` | Linux | ja |1 | – | 00:21| Ja| |
| `Web-LaTeX` | Linux | ja | 1| – | 00:25|Ja | | (6 min wenn Image nicht vorhanden)
| `Online-IDE` | Linux | ja | 1| mb1.large| 00:25| Ja| |
| `Kubernetes` | Linux | ja | 1| gp1.large| 00:32| Ja| |
| `Windows-App` | Windows | nein |1 | win11.medium| |Ja | |
| `windows-spss` | Windows | nein | 1|win11.medium | 00:23| Ja| |

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
| SSH-Login mit Nutzer-Account funktioniert | | |
| Linux-Account pro Nutzer wurde angelegt | | |
| Infrastruktur-Panel im UI zeigt korrekte IP / Outputs | | |
| Terminal reagiert flüssig | | |
| Destroy / Löschen funktioniert (Dauer: ) | |00:10 |

### 2.4 Ressourcenverbrauch (während VM unter Last)



---

## 3. Fehler & Beobachtungen

| Nr. | App / Lauf | Fehlermeldung / Beobachtung | Behoben? Wie? | GitHub-Issue |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |

### Leitfragen

| Frage | Antwort |
|---|---|
| Gab es Timeouts (Terraform, SSH, OpenStack-API)? | Nein|

