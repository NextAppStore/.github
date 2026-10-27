# App-Entwickler-Anleitung

Diese Anleitung richtet sich an App-Entwickler — in der Regel Dozenten —, die eine eigene App für die Click-n-Deploy-Plattform bauen wollen, z. B. eine vorkonfigurierte Linux- oder Windows-Lernumgebung für ihren Kurs. Sie beschreibt alle Anforderungen und Möglichkeiten — von der minimalen App bis hin zu erweiterten Funktionen wie Packer-Images, User-Management, File-Uploads und Windows-VMs.

### Autoren
1. **Erik Breuer**

### Reviewer (Prüfer)

> Zuletzt bearbeitet: 26.10.2026


---

## 1. Was ist eine App?

Eine App ist ein **Git-Repository** (GitHub oder GitLab) mit einem `terraform/`-Verzeichnis und einem optionalen `packer/`-Verzeichnis.

Die Plattform klont das Repo bei jedem Deploy auf einen versionierten Git-Tag, baut ggf. die in `packer/` definierten Images und wendet den `terraform/`-Plan gegen das OpenStack-Backend an. Die durch Terraform exportierten Outputs werden vom Backend gelesen, um Zugangsdaten an Endnutzer zu mailen und das Infrastruktur-Panel im UI zu befüllen.

> **Hinweis:** Die Plattform führt den `terraform/`-Code intern mit **OpenTofu** aus, einem zu Terraform kompatiblen Fork. Für die App-Entwicklung ändert das nichts — lokal kann ganz normal mit Terraform gearbeitet werden.

---

## 2. Voraussetzungen für lokale Entwicklung

- **Terraform** >= 1.5
- **Packer** >= 1.9 (nur wenn ein eigenes Image gebaut wird)
- **OpenStack-Zugang** (`clouds.yaml`)
- **RDP-Client** (nur für Windows-Apps), z. B. „Windows App" bzw. Microsoft Remote Desktop unter macOS, `mstsc` unter Windows

### Installation (macOS)

```bash
brew install terraform
brew install packer
```

### Installation (Windows)

```bash
winget install Hashicorp.Terraform
winget install Hashicorp.Packer
```

### OpenStack Auth (`clouds.yaml`)

Die `clouds.yaml` kann direkt aus OpenStack heruntergeladen werden:
**Profil (oben rechts) → OpenStack clouds.yaml-Datei herunterladen**

Die heruntergeladene Datei enthält kein Passwort — folgende Zeile muss manuell unter `auth:` ergänzt werden:

```yaml
password: "<DEIN PASSWORT>"
```

Standardpfad:

```
~/.config/openstack/clouds.yaml
```

Beispiel:

```yaml
clouds:
  openstack:
    auth:
      auth_url: <AUTH_URL>
      username: "<USERNAME>"
      password: "<PASSWORD>"
      project_name: "<PROJECT_NAME>"
      user_domain_name: "<USER_DOMAIN_NAME>"
    region_name: "<REGION_NAME>"
    interface: "public"
    identity_api_version: 3
```

> **Wichtig:** Der Profilname in der `clouds.yaml` **muss** `openstack` heißen — die Plattform erwartet genau diesen Namen.

---

## 3. Repository erstellen

### Option A: Template der Organisation nutzen (empfohlen)

Das Template-Repository unter [six7-click-n-deploy/template-app](https://github.com/six7-click-n-deploy/template-app) auf GitHub öffnen und oben rechts auf **"Use this template"** → **"Create a new repository"** klicken. Anschließend das neue Repository lokal klonen:

```bash
git clone <DEINE_REPO_URL>
cd <REPO_NAME>
```

Das Template enthält bereits eine vollständige Struktur:

```
template-app/
├── packer/
│   ├── template.pkr.hcl          # Packer Template (Image Build)
│   ├── variables.pkr.hcl         # Packer-Variablen
│   └── scripts/
│       └── provision.sh          # Provisioning Script
│
├── terraform/
│   ├── main.tf                   # OpenStack Ressourcen
│   ├── variables.tf              # Variablen
│   ├── outputs.tf                # Outputs
│   ├── user-data.yaml.tpl        # cloud-init Template
│   └── terraform.tfvars          # Lokale Konfiguration (nicht committen)
│
├── .github/
│   ├── workflows/
│   │   ├── packer.yml            # CI: packer fmt + validate
│   │   └── terraform.yml         # CI: terraform fmt + validate + tflint + tfsec
│   └── actions/
│       └── action.yml            # Custom Action: Packer installieren
├── .gitignore
└── README.md
```

Das Template setzt bereits **User-Management** und ein **Single-Packer-Image** um — beides ist optional und kann entfernt werden, wenn es nicht benötigt wird.

#### Welches Template?

| Template | Betriebssystem | Zugang für Endnutzer | Wann verwenden? |
|---|---|---|---|
| `template-app` | Linux (Ubuntu 22.04) | Web-URL oder SSH | Webanwendungen, Terminal-Umgebungen, Server-Software |
| `windows-template` | Windows 11 | Remote Desktop (RDP) | Windows-Software, z. B. Statistik- oder Entwicklungsprogramme |


> **Tipp:** Wer wenig Erfahrung mit Terraform hat, kann sich von einem KI-Assistenten durch die Erstellung der App führen lassen — siehe Abschnitt 11. Die Anforderungen dieser Anleitung gelten trotzdem.

#### Bestehende Apps als Vorlage

Neben den Templates eignen sich die bereits vorhandenen Apps der Organisation als Nachschlagewerk für typische Anwendungsfälle:

| App | OS | Packer | Zugang | VM-Aufteilung | Interessant für |
|---|---|---|---|---|---|
| `Ubuntu-App` | Linux | ja | SSH | eine gemeinsame VM | Linux-Account pro Nutzer auf einer VM |
| `Jupyter-Notebook` | Linux | ja | Web | eine gemeinsame VM | Web-Login pro Nutzer (JupyterHub) |
| `pgAdmin` | Linux | ja | Web | eine VM pro Team | Webanwendung mit Datenbank |
| `Web-LaTeX` | Linux | ja | Web | eine VM pro Team | Datei-Upload pro Team (ZIP) |
| `Online-IDE` | Linux | ja | Web | eine VM pro Team | eigene Security Group, Flavor pro Team, Datei-Upload |
| `Kubernetes-App` | Linux | ja | SSH + API | ein k3s-Cluster auf einer VM | komplexe Umgebung mit eigenem CLI |
| `Windows-App` | Windows | nein | RDP | eine VM pro Team | Software-Installation unter Windows |
| `windows-spss` | Windows | nein | RDP | eine VM pro Team | Datei-Upload unter Windows (ZIP + Datensatz) |

### Option B: Eigenes Repository ohne Template

Ein eigenes Git-Repository kann ebenfalls genutzt werden. Die Mindestanforderung ist ein `terraform/`-Verzeichnis mit den drei Pflichtdateien (siehe Abschnitt 4).

---

## 4. Minimale App (ohne Packer, ohne funktionierendes User-Management)

Die kleinste funktionierende App besteht aus drei Dateien im `terraform/`-Verzeichnis:

```
my-app/
└── terraform/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### `terraform/main.tf`

```hcl
terraform {
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.54"
    }
  }
}

provider "openstack" {
  cloud = "openstack"
}

resource "openstack_compute_instance_v2" "vm" {
  name        = "my-app-vm"
  image_name  = "Ubuntu 22.04"
  flavor_name = var.flavor_name

  network {
    uuid = var.network_uuid
  }
}
```

### `terraform/variables.tf`

Variablen, die der AppStore oder der Nutzer beim Deployment konfiguriert, werden hier deklariert. Für lokales Testen kann eine `terraform.tfvars`-Datei angelegt werden (nicht committen — steht im `.gitignore`).

```hcl
################################################
# PFLICHT-Variablen
################################################
variable "users" {
  description = "Per-team roster — vom Worker injiziert. @platform:internal"
  type = map(list(object({
    email = string
  })))
  default = {}
}

# image_name ist nur Pflicht wenn packer existiert
variable "image_name" {
  description = "Glance-Image-Name — vom Worker zur Apply-Zeit gesetzt. @platform:internal"
  type        = string
}

################################################
# Beispiele frei konfigurierbarer Variable
################################################
variable "flavor_name" {
  description = "VM-Größe @openstack:flavor:name"
  type        = string
  default     = "gp1.small"
}

variable "network_uuid" {
  description = "Hauptnetzwerk @openstack:network:id"
  type        = string
}
```

### `terraform/outputs.tf`

Outputs werden vom AppStore ausgelesen und dem Nutzer angezeigt. Auch wenn keine Daten zurückgegeben werden sollen, **müssen** alle drei Outputs deklariert sein.

```hcl
output "user_accounts" {
  description = "User-Zugangsdaten"
  sensitive   = true
  value       = {}
}

output "team_vms" {
  description = "Details aller Team-VMs"
  value       = {}
}

output "teams_summary" {
  description = "Übersicht: Teams und User-Anzahl"
  value       = {}
}
```

### `terraform/terraform.tfvars` (lokal, nicht committen)

```hcl
flavor_name  = "gp1.small"
network_uuid = "34a00b87-57ce-42c4-8e1b-9ea8a657ec2e"
```

---

## 5. Pflichtanforderungen für den AppStore

Damit die App im AppStore korrekt funktioniert, **müssen** folgende Variablen in `variables.tf` deklariert sein:

### `users` (immer Pflicht)

```hcl
variable "users" {
  description = "Vom Worker injiziert. @platform:internal"
  type = map(list(object({
    email = string
  })))
  default = {}
}
```

Der AppStore injiziert beim Deployment die Teams und zugehörigen User in diese Variable. Die Struktur ist dabei eine Map, bei der der Key der Team-Name ist und der Wert eine Liste von Nutzern mit ihrer E-Mail-Adresse. Wie der App-Entwickler mit dieser Struktur intern umgeht (ob er Teams als getrennte VMs deployt, eine gemeinsame VM baut, etc.), ist ihm überlassen.

### `image_name` (Pflicht wenn `packer/` existiert)

```hcl
variable "image_name" {
  description = "Glance-Image-Name — vom Worker zur Apply-Zeit gesetzt. @platform:internal"
  type        = string
}
```

Der AppStore setzt diesen Wert automatisch auf den Namen des gebauten Packer-Images. Die Variable darf nicht im Deployment-Wizard erscheinen — dafür sorgt der `@platform:internal`-Marker (siehe Abschnitt 6.3).

**Apps ohne Packer**, die ein bereits in OpenStack vorhandenes Image nutzen (z. B. `Windows-App` oder `windows-spss` mit „Windows 11 25H2 (UEFI)"), bekommen **keinen** `image_name` vom AppStore injiziert. Hier wird der Image-Name als `default` der Variable gesetzt — dieser Default ist dann der tatsächlich verwendete Wert:

```hcl
variable "image_name" {
  description = "Glance-Image-Name des bestehenden Images. @platform:internal"
  type        = string
  default     = "Windows 11 25H2 (UEFI)"
}
```

---

## 6. Optionale Funktionen

### 6.1 Packer — Eigenes VM-Image bauen

Packer ermöglicht es, ein eigenes VM-Image zu bauen, das bereits alle nötigen Abhängigkeiten, Konfigurationen und Applikationen enthält. Statt bei jedem Deployment alles neu zu installieren, wird das Image einmalig gebaut und danach für alle Deployments wiederverwendet. Der AppStore baut das Packer-Image automatisch vor dem Terraform-Deployment.

#### Struktur

```
my-app/
├── packer/
│   ├── template.pkr.hcl
│   ├── variables.pkr.hcl
│   └── scripts/
│       └── provision.sh
└── terraform/
    └── ...
```

#### `packer/variables.pkr.hcl`

Definiert die Eingabevariablen für den Packer-Build. Pflicht-Variablen:

```hcl
variable "image_name" {
  type        = string
  description = "Glance-Image-Name — vom Worker zur Build-Zeit gesetzt. @platform:internal"
}

variable "networks" {
  type        = list(string)
  description = "@openstack:network:id:list Build-Netzwerke"
  default     = ["<NETWORK-ID>"]
}

variable "security_groups" {
  type        = list(string)
  description = "@openstack:security_group:id:list Build-Security-Groups"
  default     = ["<SECURITY-GROUP-ID>"]
}
```

#### `packer/template.pkr.hcl`

Definiert die Build-Quelle und ruft das Provisioning Script auf:

```hcl
packer {
  required_plugins {
    openstack = {
      source  = "github.com/hashicorp/openstack"
      version = "~> 1"
    }
  }
}

source "openstack" "image" {
  image_name        = var.image_name
  source_image_name = "Ubuntu 22.04"
  flavor            = "gp1.small"
  networks          = var.networks
  security_groups   = var.security_groups
  ssh_username      = "ubuntu"
}

build {
  sources = ["source.openstack.image"]

  provisioner "shell" {
    script = "scripts/provision.sh"
  }
}
```

#### `packer/scripts/provision.sh`

Hier wird definiert, was ins Image kommt. Typische Inhalte:

- Pakete und Runtimes installieren
- App-Artefakte deployen (Binary, Container, Webapp)
- Konfigurationsdateien schreiben
- systemd-Services einrichten
- Optional: Reverse Proxy / TLS konfigurieren

```bash
#!/bin/bash
set -euo pipefail

apt-get update -y
apt-get install -y nginx

# App-Installation hier...

systemctl enable nginx
```


---

### 6.2 User-Management

User-Management ermöglicht es, jedem Endnutzer individuelle Zugangsdaten zu geben, sodass Nutzer abgegrenzt voneinander arbeiten können. Wie das User-Management technisch umgesetzt wird (getrennte VMs pro Team, Nutzerkonten auf einer VM, etc.) ist dem App-Entwickler überlassen.

Der AppStore liefert die Team- und Nutzerstruktur über die `users`-Variable (siehe Abschnitt 5). Die App muss die generierten Zugangsdaten über den `user_accounts`-Output zurückgeben, damit der AppStore sie per Mail an die Endnutzer versendet.

#### Struktur mit User-Management

```
my-app/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── user-data.yaml.tpl    # cloud-init Template für Zugangsdaten
└── ...
```

#### `terraform/user-data.yaml.tpl`

Das cloud-init-Template wird beim VM-Start ausgeführt und übergibt Laufzeit-Daten (Zugangsdaten, Dateien) an die VM. Es wird in `main.tf` über `templatefile()` gerendert:

```yaml
#cloud-config

bootcmd:
  # Vor write_files ausgeführt — Verzeichnisse anlegen die write_files benötigt
  - mkdir -p /etc/myapp/users
  - chown root:root /etc/myapp/users
  - chmod 750 /etc/myapp/users

write_files:
%{ for user in team_users ~}
  - path: '/etc/myapp/users/${user.email}.env'
    permissions: '0640'
    owner: 'root:myapp'
    content: |
      EMAIL=${user.email}
      PASSWORD=${user.password}
%{ endfor ~}

runcmd:
  - bash /usr/local/bin/myapp-provision.sh > /var/log/myapp-provision.log 2>&1
```

#### Output `user_accounts` (Pflicht für Mail-Versand)

Damit der AppStore Zugangsdaten per Mail versendet, **muss** `user_accounts` korrekt befüllt sein. Der Key **muss** die Form `<team-name>-<username>` haben:

```hcl
output "user_accounts" {
  sensitive = true
  value = {
    "team-a-alice" = {
      username = "alice"
      type     = "password"
      auth     = "hunter2"
      ip       = "10.0.1.42"
      port     = 8080
    }
  }
}
```

**Verfügbare Auth-Typen:**

| `type`     | Darstellung in der Mail        | Inhalt von `auth`          |
|------------|-------------------------------|---------------------------|
| `password` | "Password: …"                 | Passwort-String            |
| `ssh_key`  | Monospace SSH-Key-Block        | Public-Key oder Hinweistext |
| `oauth`    | "Login at …"-Link              | Login-URL                  |
| `none`     | Kein Credential-Block          | `auth` kann fehlen         |

**Optionale Felder** in `user_accounts`, die das Infrastruktur-Panel auswertet:

| Feld       | Bedeutung |
|------------|-----------|
| `ip_v6`    | IPv6-Adresse der VM des Nutzers. Ist sie gesetzt, kann der Nutzer im Panel zwischen IPv4 und IPv6 umschalten (siehe „Dual-Stack" unten). |
| `authtype` | Art des Zugangs: `ssh`, `rdp` oder `url`. Bei `rdp` zeigt das Panel einen fertigen `mstsc`-Befehl und den Benutzernamen zum Kopieren an (siehe Abschnitt 6.6). |

#### `metadata.team` auf VM-Ressourcen

Damit der AppStore VMs im Infrastruktur-Panel dem richtigen Team zuordnet, **sollte** jede VM ein `metadata`-Tag mit dem Team-Key erhalten:

```hcl
resource "openstack_compute_instance_v2" "team_vm" {
  for_each = toset(local.teams)
  name     = "vm-${each.key}"

  metadata = {
    team = each.key
  }
  # ...
}
```

Ohne diesen Tag wird die VM im Panel unter "Shared" angezeigt.

#### `team_vms`-Output: Pflichtfelder für das Infrastruktur-Panel

Der `team_vms`-Output befüllt das Infrastruktur-Panel im AppStore. Damit der AppStore den Nutzern den korrekten Zugangslink bzw. SSH-Befehl anzeigt, **müssen** folgende Feldnamen exakt eingehalten werden:

| App-Typ | Pflichtfeld | Beispielwert |
|---|---|---|
| Webanwendung | `url` | `"http://141.72.13.1:8080"` |
| SSH-Zugang | `ssh_command` | `"ssh alice@141.72.13.1"` |
| Remote Desktop (Windows) | — (über `user_accounts` mit `authtype = "rdp"`) | — |

Andere Feldnamen werden im Panel zwar angezeigt, aber **nicht** als klickbarer Link bzw. kopierbarer SSH-Befehl hervorgehoben.

```hcl
# Beispiel Webanwendung
output "team_vms" {
  value = {
    for team in local.teams_list : team => {
      url         = "http://${openstack_networking_floatingip_v2.team_fip[team].address}:8080"
      floating_ip = openstack_networking_floatingip_v2.team_fip[team].address
    }
  }
}

# Beispiel SSH-Anwendung
output "team_vms" {
  value = {
    for team in local.teams_list : team => {
      ssh_command = "ssh ubuntu@${openstack_networking_floatingip_v2.team_fip[team].address}"
      floating_ip = openstack_networking_floatingip_v2.team_fip[team].address
    }
  }
}
```

#### Dual-Stack (IPv4 + IPv6) über das Netz `DHBWV6`

Das Netz `DHBWV6` ist dual-stack: ein v4-Subnetz (DHCP) und ein v6-Subnetz
(`dhcpv6-stateful`) hängen am selben Neutron-Netz. Jeder Port, der an
`DHBWV6` angeschlossen wird, bekommt automatisch **beide** Fixed-IPs — kein
Vorab-Port-Trick nötig (die explizite Port-Erstellung *vor* dem VM-Start, wie
sie `Windows-App` nutzt, ist nur ein Workaround für Windows-Gäste, deren
DHCPv6-Client auf dem virtIO-Adapter nicht zuverlässig funktioniert; für
Linux-Gäste reicht `network_uuid` auf `DHBWV6` zu setzen).

Damit der AppStore die IPv6-Adresse anzeigen kann, **sollte** `team_vms` (und
optional `user_accounts`) zusätzlich ein `fixed_ip_v6`-Feld (bzw. `ip_v6` in
`user_accounts`) führen — als Sibling-Feld neben dem bestehenden
`fixed_ip`/`ip`, kein Pflichtfeld, aber empfohlene Konvention:

```hcl
output "team_vms" {
  value = {
    for team in local.teams_list : team => {
      fixed_ip    = openstack_compute_instance_v2.team_vm[team].network[0].fixed_ip_v4
      fixed_ip_v6 = openstack_compute_instance_v2.team_vm[team].network[0].fixed_ip_v6
      floating_ip = local.enable_floating_ip ? openstack_networking_floatingip_v2.team_fip[team].address : null
    }
  }
}
```

---

### 6.3 `@openstack`-Marker für Variablen

Mit dem `@openstack`-Marker in der `description` einer Terraform- oder Packer-Variable steuert der App-Entwickler, welches UI-Element der AppStore-Wizard rendert. Ohne Marker erhält der Nutzer nur ein freies Textfeld. Je nach Marker öffnen sich Dropdown-Menüs (z.B. für Flavors oder Netzwerke), Multi-Select-Felder oder Datei-Upload-Felder.

#### Übersicht aller Marker-Möglichkeiten

```
@openstack:<type>[:<mode>][:<multi>][:<var_scope>]
```

| Slot         | Mögliche Werte                                                                                                           | Default                  |
|--------------|--------------------------------------------------------------------------------------------------------------------------|--------------------------|
| `type`       | `network`, `subnet`, `flavor`, `image`, `keypair`, `security_group`, `floating_ip_pool`, `volume`, `router`, `availability_zone`, `file` | —         |
| `mode`       | `id`, `name`                                                                                                             | `name`                   |
| `multi`      | `single`, `multi` / `list`                                                                                               | aus HCL-Typ abgeleitet   |
| `var_scope`  | `all`, `team`, `user`                                                                                                    | `all`                    |

Der `@openstack`-Prefix ist case-insensitive. Mehrere Marker pro Description sind erlaubt — der erste Marker mit bekanntem Type gewinnt.

#### Beispiele

```hcl
# Netzwerk-Picker (Name), Single-Select:
variable "network_name" {
  description = "@openstack:network:name Primäres Netzwerk"
  type        = string
}

# Netzwerk-Picker (UUID), Multi-Select (z.B. für Packer):
variable "networks" {
  description = "@openstack:network:id:list Build-Netzwerke"
  type        = list(string)
}

# Flavor-Picker, einer pro Team:
variable "team_flavor_ids" {
  description = "@openstack:flavor:id:single:team Flavor pro Team"
  type        = map(string)
  default     = {}
}

# Security-Group-Picker, Multi-Select:
variable "secgroups" {
  description = "@openstack:security_group:name:multi"
  type        = list(string)
}

# Freies Textfeld pro User (kein OpenStack-Typ):
variable "github_handles" {
  description = "@openstack:::user GitHub-Username pro Endnutzer"
  type        = map(string)
  default     = {}
}

# Multi-Select Security Groups pro Team:
variable "team_secgroups" {
  description = "@openstack:security_group:name:multi:team"
  type        = map(list(string))
  default     = {}
}
```

#### `var_scope`: Werte pro Team oder pro User

Mit `:team` oder `:user` als letztem Slot rendert der Wizard einen Picker **pro Team bzw. pro User** und übergibt das Ergebnis als Map an Terraform.

`:all` ist der Default-Scope und muss nicht explizit angegeben werden. Der Wizard rendert dann genau **ein** Eingabefeld für alle Teams gemeinsam — der Wert wird direkt (kein Map-Wrapper) an Terraform übergeben. Beispiel: ein Netzwerk-Picker der für alle Teams gilt.

**Pflicht:** Wenn `var_scope` `:team` oder `:user` ist, **muss** der HCL-Typ ein `map(...)` sein — sonst schlägt die Approval mit `MARKER_SCOPED_REQUIRES_MAP` fehl. Bei `:all` gilt diese Einschränkung nicht.

Kurzform für reinen `var_scope` ohne OpenStack-Typ: `@openstack::team` (Scope im Mode-Slot) ist äquivalent zu `@openstack:::team`.

In Packer-Variablen ist `var_scope=:team` oder `:user` **verboten** — ein Packer-Build erzeugt ein einzelnes Image, das von allen Teams gemeinsam genutzt wird.

#### `@platform:internal` — Variablen aus dem Wizard ausblenden

Variablen, die vom AppStore automatisch zur Laufzeit gesetzt werden, **müssen** mit `@platform:internal` markiert werden, damit sie nicht im Deployment-Wizard für den Nutzer erscheinen und fälschlich überschrieben werden können.

```hcl
variable "image_name" {
  description = "Glance-Image-Name — vom Worker zur Apply-Zeit gesetzt. @platform:internal"
  type        = string
}

variable "users" {
  description = "Per-team roster — vom Worker injiziert. @platform:internal"
  type        = map(list(object({ email = string })))
  default     = {}
}
```

Folgende Variablen werden vom AppStore automatisch injiziert und **müssen** deklariert sein:

| Variable              | Wann injiziert                        |
|-----------------------|---------------------------------------|
| `users`               | Immer, sobald Teams im Deploy existieren |
| `image_name`          | Bei Single-Packer-Image-Apps          |
| `image_name_<key>`    | Bei Multi-Packer-Image-Apps (je Subdirectory-Key) |

---

### 6.4 File-Upload-Variablen

Mit dem `@openstack:file:`-Marker kann der App-Entwickler Datei-Uploads im AppStore-Wizard ermöglichen. Der Dateiinhalt wird base64-encodiert in eine Terraform-Map injiziert und kann per `cloud-init` auf der VM materialisiert werden.

**Pflicht** bei File-Uploads: Erlaubte Dateiendungen **müssen** angegeben werden. Als Trenner **muss** `|` verwendet werden (kein Komma).

#### Marker-Form

```
@openstack:file:<scope>:<ext1>|<ext2>|...
```

- `<scope>` ∈ `all` | `team` | `user`
- `<ext1>|<ext2>` — Pipe-separierte Liste erlaubter Extensions (lowercase, ohne Punkt)

File-Marker sind in Packer-Variablen **verboten**.

#### Größenbeschränkung

Der AppStore akzeptiert maximal **2 MB pro Datei** und **10 MB pro Deployment**. Entscheidend ist in der Praxis aber eine andere Grenze: Wird der Dateiinhalt (wie in den Beispielen unten) über das cloud-init- bzw. cloudbase-init-Template auf die VM gebracht, landet er in den `user_data` der VM — und OpenStack begrenzt `user_data` auf **ca. 64 KB**. Ist das Template größer, startet die VM nicht. Über diesen Weg eignen sich daher nur **kleine Dateien** (z. B. eine Aufgabenstellung als PDF oder ein kleines ZIP). Für größere Dateien vorher mit dem Plattform-Team einen anderen Weg klären.

#### HCL-Typen je Scope

| Scope  | HCL-Typ |
|--------|---------|
| `all`  | `map(object({ name=string, content_b64=string, content_type=string, size=number }))` |
| `team` | `map(map(object({ name=string, content_b64=string, content_type=string, size=number })))` |
| `user` | `map(map(object({ name=string, content_b64=string, content_type=string, size=number })))` |

#### Beispiel: Ein Upload-Slot für alle Teams

```hcl
variable "assignment_files" {
  description = <<-EOT
    @openstack:file:all:pdf
    Aufgabenstellung — eine PDF für alle Teams.
  EOT
  type = map(object({
    name         = string
    content_b64  = string
    content_type = string
    size         = number
  }))
  default = {}
}
```

Konsum in `user-data.yaml.tpl`:

```yaml
write_files:
%{ for slot_key, file in assignment_files ~}
  - path: /opt/app/${file.name}
    permissions: "0644"
    encoding: b64
    content: ${file.content_b64}
%{ endfor ~}
```

Bei Windows-Apps werden die Dateien stattdessen im `cloudbase-init.txt.tpl` per PowerShell geschrieben — ein fertiges Beispiel (Ablage auf dem öffentlichen Desktop, ZIPs werden entpackt) enthält das `windows-template`.

#### Beispiel: Ein Slot pro Team

```hcl
variable "team_briefings" {
  description = "@openstack:file:team:pdf|docx Briefing-Doku pro Team"
  type = map(map(object({
    name         = string
    content_b64  = string
    content_type = string
    size         = number
  })))
  default = {}
}
```

Der äußere Map-Key ist die Team-ID, der innere der Slot-Key.

#### Wichtig: Verhalten beim Destroy

Beim Destroy eines Deployments entfernt der AppStore alle file-shaped Variablen aus dem `-var`-Set. File-Variablen dürfen deshalb **nicht** in `count = …` oder `for_each = …` referenziert werden — beim Destroy wäre der Wert leer und Terraform würde Ressourcen fälschlich löschen wollen.

---

### 6.5 Multi-Packer-Image-Apps

Statt eines einzelnen Images können mehrere Images parallel gebaut werden — z.B. ein separates Image für Webserver und Datenbank. Der Wechsel von Single- zu Multi-Image ist ausschließlich eine Frage des Verzeichnis-Layouts im `packer/`-Ordner.

#### Struktur

```
my-app/
├── packer/
│   ├── webserver/
│   │   ├── template.pkr.hcl
│   │   ├── variables.pkr.hcl
│   │   └── scripts/
│   │       └── provision.sh
│   └── database/
│       ├── template.pkr.hcl
│       ├── variables.pkr.hcl
│       └── scripts/
│           └── provision.sh
└── terraform/
    └── ...
```

Die Funktion der einzelnen Dateien (`template.pkr.hcl`, `variables.pkr.hcl`, `provision.sh`) ist dieselbe wie bei der Single-Image-App (siehe Abschnitt 6.1).

**Regeln für Subdirectory-Keys** (Verzeichnisnamen):
- Format: `[a-z][a-z0-9_-]{0,30}`
- Ungültige Keys führen zu einem `PackerTemplateDiscoveryError`
- Verzeichnisse ohne `template.pkr.hcl` (z.B. `_common/`, `scripts/`) werden ignoriert
- Build-Reihenfolge: alphabetisch nach Verzeichnisname (`database` baut vor `webserver`)
- Single- und Multi-Image gleichzeitig ist **verboten**: `packer/template.pkr.hcl` und `packer/<key>/template.pkr.hcl` dürfen nicht gleichzeitig existieren

#### Packer-Variablen je Subdirectory

In jedem Subdirectory referenziert das Template `var.image_name` — **nicht** `var.image_name_<key>`. Der AppStore injiziert pro Build den korrekten Namen:

```hcl
# packer/webserver/variables.pkr.hcl
variable "image_name" {
  type        = string
  description = "Glance-Image-Name — vom Worker zur Build-Zeit gesetzt. @platform:internal"
}
```

#### Terraform-Variablen für Multi-Image (Pflicht)

Für jeden Packer-Subdirectory-Key **muss** eine separate `image_name_<key>`-Variable in `variables.tf` deklariert werden:

```hcl
variable "image_name_webserver" {
  description = "Glance-Image-Name des Webserver-Images — vom Worker gesetzt. @platform:internal"
  type        = string
}

variable "image_name_database" {
  description = "Glance-Image-Name des Database-Images — vom Worker gesetzt. @platform:internal"
  type        = string
}
```

---

### 6.6 Windows-Apps (RDP)

Für Software, die nur unter Windows läuft, können Apps auch Windows-11-VMs bereitstellen. Endnutzer greifen dann per **Remote Desktop (RDP, Port 3389)** auf einen vollständigen Windows-Desktop zu. Als Startpunkt dient das `windows-template` (siehe Abschnitt 3); es ist aus der getesteten `Windows-App` abgeleitet.

#### Unterschiede zu Linux-Apps

| | Linux-App (`template-app`) | Windows-App (`windows-template`) |
|---|---|---|
| Image | per Packer gebautes Image | per Packer gebautes Image auf Basis von „Windows 11 25H2 (UEFI)" |
| Packer-Verbindung | SSH | **WinRM über HTTPS (Port 5986)**, temporärer Admin `packer` |
| `image_name` | vom AppStore injiziert | vom AppStore injiziert |
| Boot-Konfiguration | `user-data.yaml.tpl` (cloud-init, YAML) | `cloudbase-init.txt.tpl` (cloudbase-init, PowerShell) |
| Software | im Packer-Image vorinstalliert (`provision.sh`) | im Packer-Image vorinstalliert (`provision.ps1`) |
| Flavor | z. B. `gp1.small` | mind. `win11.medium` (8 GB RAM, 2 vCPU, 80 GB) |
| Zugang | `url` / `ssh_command` | `user_accounts` mit `authtype = "rdp"`, `port = 3389` |
| Bereitstellungsdauer | wenige Minuten | Image-Build ca. 20–40 Min (einmal pro Tag), danach ca. 4–6 Min bis RDP erreichbar |

#### Struktur

```
my-windows-app/
├── packer/
│   ├── template.pkr.hcl          # Image-Build über WinRM
│   ├── variables.pkr.hcl         # image_name, networks, security_groups
│   └── scripts/
│       ├── bootstrap-winrm.ps1.pkrtpl  # user_data der Build-VM: WinRM an
│       ├── provision.ps1         # Software ins Image installieren
│       └── cleanup.ps1           # Aufräumen vor dem Snapshot
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── cloudbase-init.txt.tpl    # PowerShell-Script für den ersten Boot
├── .github/workflows/            # packer.yml + terraform.yml
├── .gitattributes                # LF-Zeilenenden für .tf/.tpl/.ps1
└── README.md
```

Faustregel: **Was für alle VMs gleich ist (Software), gehört ins Packer-Image. Was pro Deployment/Team anders ist (Nutzer, Passwörter, IPv6, Uploads), gehört in `cloudbase-init.txt.tpl`.**

#### Packer-Build (`packer/`)

1. Packer startet eine Build-VM aus „Windows 11 25H2 (UEFI)". Über `user_data` legt cloudbase-init dort den temporären Admin `packer` an (Passwort pro Build zufällig) und öffnet **WinRM über HTTPS (Port 5986)** — das Windows-Gegenstück zu SSH.
2. Packer verbindet sich per WinRM, führt `provision.ps1` aus, startet neu und räumt mit `cleanup.ps1` auf.
3. Der Snapshot wird als Glance-Image gespeichert; die UEFI-Properties (`hw_firmware_type`, `hw_machine_type`) setzt das Template explizit.
4. Beim Deploy entfernt `cloudbase-init.txt.tpl` den Account `packer`, den WinRM-Listener und die Firewallregel wieder.

Als App-Entwickler wird in der Regel nur der Block zur **Software-Installation** in `packer/scripts/provision.ps1` angepasst — ein Block pro Programm, idempotent und mit stiller Installation:

```powershell
$exe = "$env:ProgramFiles\Microsoft VS Code\Code.exe"
if (-not (Test-Path $exe)) {
    $installer = "$env:TEMP\vscode-setup.exe"
    Invoke-WebRequest -Uri "https://update.code.visualstudio.com/latest/win32-x64/stable" -OutFile $installer -UseBasicParsing
    Start-Process -FilePath $installer -ArgumentList "/VERYSILENT", "/NORESTART" -Wait -NoNewWindow
    if (-not (Test-Path $exe)) { throw "VS Code installation failed" }
}
```

- **System-Installer** verwenden (nicht „User Installer"), damit die Software allen Nutzern der VM zur Verfügung steht — die Nutzer-Accounts entstehen erst beim Deploy.
- Fehler mit `throw` abbrechen lassen: Dann schlägt der Build sichtbar fehl, statt ein Image ohne Software zu erzeugen.
- Die Build-VM braucht eine Security Group mit **eingehend TCP 5986** (Default im Template: `packer-winrm`). Die deployten VMs brauchen weiterhin nur RDP.
- **Kein Sysprep:** cloudbase-init führt `user_data` pro neuer OpenStack-Instanz erneut aus; alle VMs teilen sich dadurch die Maschinen-SID (ohne Domäne unkritisch).
- Der gesamte Build muss in das **60-Minuten-Timeout** des Workers passen — bei sehr großer Software (Visual Studio, Office, SQL Server) vorher mit dem Plattform-Team sprechen.

#### `terraform/cloudbase-init.txt.tpl`

Das Script beginnt mit `#ps1_sysnative` und wird wie bei Linux über `templatefile()` in `main.tf` gerendert. Das Template legt bereits die lokalen Windows-Accounts an, aktiviert RDP, entfernt die Packer-Build-Artefakte, legt hochgeladene Dateien ab und setzt die IPv6-Adresse. Hier wird in der Regel nichts angepasst — Software gehört ins Packer-Image.

- PowerShell-Variablen **ohne** geschweifte Klammern schreiben (`$env:TEMP`) — Terraform interpretiert `${...}` im Template selbst.
- Fehler landen auf der VM in `C:\cloudbase-init-app.log` (einsehbar über die OpenStack-Konsole).
- **Lizenzpflichtige Software** (Installer, Lizenzschlüssel) nie im Repository hinterlegen, sondern mit dem Plattform-Team klären.

#### Output `user_accounts` für RDP

```hcl
output "user_accounts" {
  sensitive = true
  value = {
    for uid, user in local.users_map : uid => {
      type     = "password"
      authtype = "rdp"
      ip       = local.access_ip[user.team]
      ip_v6    = local.fixed_ip_v6[user.team]
      port     = 3389
      username = user.username
      auth     = random_password.user_passwords[uid].result
    }
  }
}
```

Das Infrastruktur-Panel zeigt damit pro Nutzer einen kopierbaren `mstsc`-Befehl (IPv4 oder IPv6) und den Benutzernamen an; das Passwort kommt per Mail.

#### Windows-spezifische Fallstricke

Die folgenden Punkte sind im `windows-template` bereits gelöst und sollten beim Anpassen **nicht** verändert werden. Eine ausführliche Erklärung steht im README des Templates.

- Benutzernamen ohne Punkte und mit max. 20 Zeichen (Windows-Limit)
- Passwörter ohne `$`, `` ` ``, `"`, `'` (brechen die PowerShell-Interpolation)
- Gruppen und Firewall-Regeln sprachunabhängig über SIDs bzw. Gruppen-Token (deutsches und englisches Windows)
- IPv6 wird statisch gesetzt, da DHCPv6 für Windows-Gäste im Netz `DHBWV6` nicht zuverlässig funktioniert — dafür wird der Port **vor** der VM angelegt
- Die im Wizard gewählte Security Group (`shared_secgroup_id`) **muss** TCP 3389 für IPv4 und IPv6 erlauben

---

## 7. Lokales Deployment

### Nur Terraform (kein Packer)

```bash
cd terraform
terraform init
terraform validate
terraform plan        # Vorschau: zeigt was erstellt/geändert/gelöscht wird
terraform apply
```

### Mit Packer (erst Image bauen, dann deployen)

```bash
# 1. Packer-Image bauen
cd packer
packer init .
packer validate .
packer build .

# 2. Terraform deployen
cd ../terraform
terraform init
terraform validate
terraform plan
terraform apply
```

**Nach `terraform apply` stehen folgende Outputs zur Verfügung:**
- `user_accounts` — Zugangsdaten (sensitiv, abrufbar mit `terraform output -json user_accounts`)
- `team_vms` — Details aller Team-VMs mit IP-Adressen und App-URL
- `teams_summary` — Übersicht der Teams mit jeweiliger User-Anzahl

---

## 8. Cleanup

### Infrastruktur entfernen

```bash
cd terraform
terraform destroy
```

### Packer-Image entfernen (optional)

```bash
openstack image list
openstack image delete <IMAGE_ID>
```

---

## 9. Checks und CI/CD

Vor dem Pushen können folgende Checks lokal ausgeführt werden:

**Einmalige Installation (macOS):**

```bash
brew install tflint    
brew install tfsec     
```

**Terraform:**

```bash
cd terraform
terraform fmt          # Formatierung korrigieren
terraform validate     # Syntax prüfen
terraform plan         # Deployment-Vorschau
tflint --recursive     # Linter
tfsec .                # Security-Check
```

**Packer:**

```bash
cd packer
packer fmt .           # Formatierung korrigieren
packer validate .      # Syntax prüfen
```

### GitHub Actions Workflow (bei Nutzung des Templates)

Wer das Template der Organisation nutzt, erhält automatisch zwei vorkonfigurierte GitHub Actions Workflows:

| Workflow         | Datei                          | Was wird geprüft                                  |
|------------------|--------------------------------|---------------------------------------------------|
| Terraform CI     | `.github/workflows/terraform.yml` | `terraform fmt`, `terraform validate`, `tflint`, `tfsec` |
| Packer CI        | `.github/workflows/packer.yml`    | `packer fmt`, `packer validate`                   |

Diese Workflows laufen automatisch bei jedem Push. Schlägt einer der Checks fehl, wird der Workflow als fehlgeschlagen markiert.

Das `windows-template` enthält beide Workflows.

---

## 10. App dem AppStore hinzufügen

### Push auf GitHub

Bevor die App auf GitHub gepusht wird sollte eine Markdown-Datei erstellt oder die Markdown-Datei des Templates bearbeitet werden. Diese Datei sollte die App beschreiben und alle wichtigen Details für App-Entwickler umfassen.

Danach kann der Code auf GitHub gepusht werden.

### Release / Git-Tag erstellen

Die Plattform deployt ausschließlich versionierte Git-Tags — keinen Branch. Vor dem Hinzufügen der App im AppStore **muss** ein Release mit einem Git-Tag im folgenden Format erstellt werden:

```
v<Major>.<Minor>.<Patch>
```

Beispiel: `v1.0.0`, `v0.2.1`

Tags in anderen Formaten funktionieren zwar technisch, sortieren aber nur lexikographisch und können im Wizard zur falschen "neueste Version"-Anzeige führen.

Zu jedem Release **muss** eine umfassende Beschreibung erstellt werden (siehe unten).

### App registrieren

Bei **privaten Repositories** muss zunächst der Collaborator `six7clickndeploy` hinzugefügt werden, damit der AppStore Zugriff hat:
**Settings → Collaborators → Add people → `six7clickndeploy`**

Bei öffentlichen Repositories ist dieser Schritt nicht notwendig.

Danach die **GitHub-URL des Repositories** im AppStore unter **"App hinzufügen"** eintragen:

```
https://github.com/<dein-username>/<repo-name>
```

Der AppStore liest daraufhin die Konfiguration (Variablen, Packer-Template, Terraform) automatisch aus dem Repository ein.

### Pflichtangaben in der App-Beschreibung

Bei der Registrierung und bei jedem neuen Release **muss** eine umfassende Beschreibung der App hinterlegt werden. Diese muss mindestens folgende Punkte abdecken:

- **User-Management:** Ist ein User-Management vorhanden? Wie ist es geregelt? (z.B. ein Account pro User, gemeinsame Zugangsdaten pro Team)
- **VM-Deployment:** Wie viele VMs werden deployed? (z.B. eine VM pro Team, eine VM pro User, eine gemeinsame VM)
- **Konfigurierbare Variablen:** Welche Variablen kann der Deployer konfigurieren und wie werden diese befüllt? (z.B. Flavor-Auswahl, Netzwerk, Datei-Upload)
- **Änderungen im Release:** Was wurde in dieser Version geändert oder hinzugefügt?

---

## 11. KI-Assistent: App-Wizard-Skill

Für App-Entwickler ohne Terraform-Kenntnisse gibt es den **App-Wizard** — einen Skill für den KI-Coding-Assistenten [Claude Code](https://docs.claude.com/en/docs/claude-code/overview). Er stellt in Alltagssprache einige Fragen (Wofür ist die App? Welche Software? Datei-Upload?), leitet daraus ab, ob eine Linux-App (SSH oder Web) oder eine Windows-App (RDP) passt, erzeugt das App-Repository auf Basis von `template-app` bzw. `windows-template` und führt anschließend durch Commit, Push, Versions-Tag und Registrierung im AppStore.

### Nutzung

```bash
git clone https://github.com/NextAppStore/template-app
cd template-app
claude
```

Danach im Chat z. B. *„Ich möchte eine neue App für meine Statistik-Vorlesung erstellen"* eingeben oder den Skill direkt mit `/app-wizard` aufrufen. Der Skill findet die übrigen Templates und Beispiel-Apps im selben bzw. übergeordneten Verzeichnis und fragt nach, bevor er fehlende Repositories klont oder Befehle (z. B. `git push`) ausführt.

### Grenzen — keine Garantie

Der App-Wizard ist eine **Hilfe beim Einstieg, kein geprüftes Werkzeug**. Die erzeugte App kann Fehler enthalten; die Verantwortung für die App bleibt beim App-Entwickler. Insbesondere gilt:

- **Kein eigener Test:** Der Skill selbst prüft höchstens die Syntax (`terraform validate`). Ob die VM startet, die Software installiert wird und der Zugang funktioniert, zeigt erst ein Deployment. Vor dem ersten Release **muss** die App deshalb mit einem Test-Team (ein Nutzer) deployt und geprüft werden. Auf Wunsch begleitet der Skill ein solches Test-Deployment vom Terminal: Er prüft, ob Terraform (und ggf. Packer) installiert sind, und gibt die nötigen Befehle zum Kopieren aus — **ausgeführt werden sie vom App-Entwickler selbst**, der Skill legt keine Ressourcen in OpenStack an.
- **Unbekannte Software:** Für Software, die in keiner bestehenden App vorkommt, beruht die Installation auf dem allgemeinen Wissen des KI-Modells und ist nicht erprobt. Der Skill weist darauf hin — die README der App sollte das unter „Bekannte Einschränkungen" festhalten.
- **Lizenzpflichtige Software** (z. B. SPSS, Office, MATLAB): Installer-Quelle und Lizenz kann der Skill nicht bereitstellen; diese müssen vom App-Entwickler bzw. Plattform-Team kommen und dürfen nicht ins Repository.
- **Nur Standardfälle:** Der Skill deckt die Muster der Templates ab (eine VM pro Team, Datei-Upload, Software-Installation). Sonderfälle wie mehrere Images, Kubernetes oder eigene Netzwerke erzeugt er nicht oder nur eingeschränkt — hier dienen die bestehenden Apps (Abschnitt 3) und diese Anleitung als Grundlage.
- **Windows:** Windows-Apps übernehmen den Packer-Build aus dem `windows-template` (WinRM). Der Build dauert deutlich länger als bei Linux (ca. 20–40 Min) und muss in das 60-Minuten-Timeout des Workers passen; sehr große Software vorher mit dem Plattform-Team klären.
- **Abhängigkeit vom Modell:** Ergebnisse können sich je nach Modellversion und Gesprächsverlauf unterscheiden. Für die Nutzung ist ein Zugang zu Claude Code nötig.

Maßgeblich bleiben immer diese Anleitung und die `AGENTS.md` des jeweiligen Templates.
