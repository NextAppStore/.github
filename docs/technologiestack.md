# Technologiestack

## 1. Frontend: Vue.js

**Beschreibung:** Das Frontend wird als Single Page Application (SPA) mit Vue.js 3 realisiert. Für das State Management (die Verwaltung des globalen Anwendungszustands) kommt Pinia zum Einsatz, während Axios die Kommunikation mit dem Backend übernimmt. Das Styling erfolgt utility-first mittels Tailwind CSS.

**Begründung:** Die Architektur setzt auf eine strikte Trennung von Benutzeroberfläche (UI), Geschäftslogik und Routing. Der Einsatz von TypeScript sorgt hierbei für Typsicherheit, wodurch viele Fehler bereits zur Entwicklungszeit und nicht erst zur Laufzeit abgefangen werden. Tailwind CSS ermöglicht eine schnelle UI-Entwicklung ohne die Wartungslast komplexer, selbstgeschriebener CSS-Dateien. Vue.js wurde gewählt, da es als leichtgewichtig und performant gilt und sich perfekt für Frontends dieser Skalierung eignet. Im Vergleich zu oft schwergewichtigeren Enterprise-Lösungen (wie Angular) bietet es eine geringere Einstiegshürde bei gleicher Leistungsfähigkeit für diesen Anwendungsfall.

---

## 2. Backend: FastAPI (Python)

**Beschreibung:** Das Backend fungiert als asynchrone REST-API, basierend auf FastAPI und Python 3.11+. Als Webserver dient Uvicorn, ein High-Performance ASGI-Server. Die Datenhaltung erfolgt in einer PostgreSQL-Datenbank, die über SQLAlchemy 2.0 (ORM) angesprochen wird.

**Begründung:** Der Fokus liegt auf Performance und modernen Standards. Pydantic validiert alle Dateneingänge rigoros und generiert automatisch die OpenAPI-Dokumentation (Swagger UI), was die Integration für Frontend-Entwickler erleichtert. PostgreSQL wurde aufgrund seiner Stabilität (ACID-Compliance) und der Unterstützung für JSONB-Datentypen gewählt, was Flexibilität bei semi-strukturierten Daten erlaubt. Die Entscheidung fiel bewusst gegen klassische Frameworks wie Django oder Flask. FastAPI ist durch die native Unterstützung von asynchroner Programmierung (Async/Await) und die Nutzung von Starlette deutlich performanter. Zudem bietet SQLAlchemy 2.0 im Gegensatz zu simpleren ORMs die Flexibilität, nahtlos zwischen objektorientiertem Zugriff und Raw SQL zu wechseln.

---

## 3. Worker: Asynchrone Verarbeitung via PostgreSQL

**Beschreibung:** Um langlaufende Prozesse zu handhaben, wird eine Task-Queue-Architektur eingesetzt. Diese wird vollständig in PostgreSQL über eine Tabelle und die Befehle LISTEN und NOTIFY realisiert.

**Begründung:** Durch eine Queue-Architektur bleibt die UI flüssig und das Backend wird entlastet. Die Aufgaben werden durch den Worker abgearbeitet, welcher mehrere Threads zur Verfügung stellt, um die Aufgaben zu parallelisieren. Besonders Infrastruktur-Änderungen via OpenTofu werden so abgearbeitet.

---

## 4. Deployment & Containerisierung: Docker

**Beschreibung:** Die gesamte Anwendung wird mittels Docker containerisiert. Für die lokale Entwicklung kommt `docker-compose.dev.yml` zum Einsatz, das die Services mit hot-reload und gemounteten Source-Dateien startet. Für das Deployment auf der OpenStack-VM existiert `docker-compose.staging.yml` (CD-Pipeline, Image-Versionen festlegbar) bzw. `docker-compose.prod.yml` (always-latest aus der Registry); beide ziehen fertige Images aus der GitHub Container Registry (GHCR) und führen keine Build-Schritte auf der Zielmaschine aus.

Der Stack besteht aus folgenden Containern:

| Container | Aufgaben |
|---|---|
| **PostgreSQL** | Zentrale Datenhaltung der Anwendung (Benutzer, Apps, Deployments) |
| **Keycloak** | Identity Provider; Verwaltung von Benutzern, Rollen und Rechten; OIDC-basierte Authentifizierung für Frontend und Backend |
| **Backend** | REST-API; nimmt Anfragen entgegen, validiert Tokens gegen Keycloak, persistiert Daten und dispatcht Deployment-Tasks an RabbitMQ |
| **Worker** | Konsumiert Tasks aus RabbitMQ; klont App-Repositories; führt Packer und Terraform aus; provisioniert Infrastruktur auf OpenStack |
| **Frontend** | Vue 3 SPA, ausgeliefert von nginx im Container; kommuniziert ausschließlich über die Backend API |
| **nginx** | Reverse Proxy und TLS-Terminierung. Leitet eingehende HTTPS-Requests (Port 443) an Frontend, Backend (`/api/`) und Keycloak (`/realms/`) weiter. Das Self-Signed-Zertifikat wird von Ansible beim ersten Deploy erzeugt und bei Folgedeploys wiederverwendet |

**Deploymentprozess:** Ein Push auf `main` in einem der Service-Repositories (Backend, Frontend, Worker) löst zunächst die jeweilige CI-Pipeline aus, die das Image baut und in die GHCR pusht. Sobald das erfolgreich abgeschlossen ist, triggert die Pipeline automatisch den Deployment-Workflow im deployment-Repository. OpenTofu provisioniert die OpenStack-VM (oder stellt sicher, dass sie läuft) und gibt die IP-Adresse aus. Anschließend führt Ansible das Deployment-Playbook auf der VM aus: es synchronisiert das Repository, schreibt die `.env`-Datei aus dem GitHub-Secret, rendert `keycloak/realm-export.json.j2` mit der öffentlichen URL der VM, zieht die GHCR-Images und startet den Stack via `docker compose up`. Datenbankmigrationen werden als expliziter Ansible-Task ausgeführt, damit Fehler sichtbar im Pipeline-Log erscheinen und nicht im Compose-Output verschwinden.

Der Keycloak-Realm wird beim ersten Hochfahren automatisch aus `realm-export.json` importiert, das Ansible zuvor mit der korrekten APP_BASE_URL befüllt hat. Dadurch entfällt manuelles Klicken im Admin-UI bei jedem frischen Deploy.

**Begründung:** Docker garantiert Portabilität: Entwicklung und Deployment laufen in identischen Umgebungen, was die Gefahr umgebungsbedingter Fehler deutlich reduziert. Multi-Stage Builds halten die Images klein und frei von Build-Werkzeugen.

---

## 5. Infrastructure as Code (IaC): OpenTofu & Packer

**Beschreibung:** Die Infrastruktur wird nicht manuell, sondern durch Code definiert. OpenTofu provisioniert die Ressourcen (VMs, Netzwerke, Security Groups) auf OpenStack. Packer wird genutzt, um Abbilder für die VMs zu bauen.

**Begründung:** Dieser Ansatz ermöglicht Disaster Recovery "per Knopfdruck" und vollständige Versionierung der Infrastruktur in Git. Ein entscheidender Vorteil ist der "Golden Image"-Workflow mit Packer. Anstatt eine leere VM zu starten und dann minutenlang Software zu installieren (wie bei reinem Ansible/Shell-Scripting oft üblich), erstellt Packer ein fertiges Image (Snapshot).
