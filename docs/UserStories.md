# User Stories



## Übersicht

| ID | Rolle | Story |
|---|---|---|
| [IT-1](#it-1) | IT Dozent | VM schnell bereitstellen |
| [IT-2](#it-2) | IT Dozent | Standardeinstellungen oder eigene Konfiguration |
| [IT-3](#it-3) | IT Dozent | Gemeinsame VM oder eigene VM pro Studierendem |
| [IT-4](#it-4) | IT Dozent | Studierende Gruppen zuordnen |
| [IT-5](#it-5) | IT Dozent | Einfache Bedienoberfläche |
| [IT-6](#it-6) | IT Dozent | VMs untereinander vernetzen |
| [BWL-1](#bwl-1) | BWL Dozent | Informationen vor der Installation |
| [BWL-2](#bwl-2) | BWL Dozent | Mit wenigen Klicks starten |
| [BWL-3](#bwl-3) | BWL Dozent | Empfehlungen und Hilfe bei der Konfiguration |
| [BWL-4](#bwl-4) | BWL Dozent | Übungsdatensätze zentral ablegen |
| [BWL-5](#bwl-5) | BWL Dozent | Bedienung ohne technische Vorkenntnisse |
| [BWL-6](#bwl-6) | BWL Dozent | Integration in Moodle |
| [ADMIN-1](#admin-1) | Administrator | Integration in den Self-Service |
| [ADMIN-2](#admin-2) | Administrator | Automatische Erstellung von Security-Groups |

---

## INVEST

| | Begriff | Prüffrage |
|---|---|---|
| **I** | Independent | Lässt sich die Story umsetzen, ohne dass eine andere vorher fertig sein muss? |
| **N** | Negotiable | Beschreibt sie *was* gebraucht wird, nicht schon *wie* es technisch gelöst wird? |
| **V** | Valuable | Hat sie einen erkennbaren Nutzen für die Rolle? |
| **E** | Estimable | Ist sie klar genug, dass das Team den Aufwand abschätzen kann? |
| **S** | Small | Ist sie klein genug für einen Sprint, oder stecken mehrere Funktionen darin? |
| **T** | Testable | Lässt sich eindeutig prüfen, ob sie erfüllt ist? |


---

## IT Dozent


<a id="it-1"></a>

### IT-1 - VM schnell bereitstellen

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | für meine Lehrveranstaltung schnell eine VM bereitstellen, |
| **damit** | alle Studierenden mit einer einheitlichen Umgebung arbeiten können. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Der Dozent kann eine App aus dem Katalog auswählen und für die Studierenden eines Kurses deployen. |
| 2 | Alle Studierenden erhalten dieselbe App-Version. |
| 3 | Benötigt die App keine Pflichtangaben, lässt sich das Deployment per Schnell-Deploy direkt von der App-Kachel starten, ohne den Wizard durchlaufen zu müssen. |
| 4 | Der Fortschritt des Deployments ist live sichtbar. Erfolg oder Fehler wird angezeigt. |
| 5 | Nach erfolgreichem Deployment sehen die Studierenden ihre Zugangsdaten in der Oberfläche und erhalten sie per E-Mail. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

Anmerkung: „schnell“ ist für sich nicht messbar — Kriterium 3 (Schnell-Deploy) macht es prüfbar.

<a id="it-2"></a>

### IT-2 - Standardeinstellungen oder eigene Konfiguration

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | die Möglichkeit haben, eine VM mit funktionierenden Standardeinstellungen auszuwählen, aber auch wenn nötig, bestimmte Betriebssystem-/Software-Konfiguration bereitzustellen. |

<a id="it-2a"></a>

#### IT-2a - Funktionierende Standardeinstellungen

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | eine VM mit funktionierenden Standardeinstellungen auswählen, |
| **damit** | ich ohne Anpassungen eine lauffähige Umgebung bekomme. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Die Einstellungen einer App sind mit Standardwerten vorbelegt, sofern die App keine Pflichtangabe verlangt. |
| 2 | Ein Deployment, bei dem keine Einstellung geändert wird, läuft erfolgreich durch. |
| 3 | Die verwendeten Standardwerte sind vor dem Start sichtbar. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

<a id="it-2b"></a>

#### IT-2b - Eigene Betriebssystem-/Software-Konfiguration

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | bei Bedarf eine bestimmte Betriebssystem-/Software-Konfiguration bereitstellen, |
| **damit** | die Umgebung zu den Inhalten meiner Lehrveranstaltung passt. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Die App-Version ist frei wählbar. |
| 2 | Betriebssystem-Image und VM-Größe und weitere Einstellungen wie Resourcen lassen sich anpassen, sofern die App das vorsieht. |
| 3 | Ungültige Eingaben oder fehlende Pflichtangaben werden vor dem Start mit einer verständlichen Meldung abgelehnt. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

<a id="it-3"></a>

### IT-3 - Gemeinsame VM oder eigene VM pro Studierendem

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | mich zwischen einer gemeinsamen VM für den ganzen Kurs und einer eigenen VM pro Studierendem entscheiden können, |
| **damit** | ich die Umgebung an das jeweilige Lehrformat und die benötigten Ressourcen anpassen kann. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Beim Deployment lässt sich wählen, ob alle Studierenden eine gemeinsame VM nutzen oder jeder eine eigene VM erhält. |
| 2 | Vor dem Start wird angezeigt, wie viele VMs entstehen. |
| 3 | Studierende erhalten nur Zugang zu der Umgebung, der sie zugeordnet sind. |
| 4 | Die freien Ressourcen (Quota) sind beim Deployment einsehbar. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| teilweise | ja | ja | ja | ja | ja |

Anmerkung: Story baut auf der Gruppenzuordnung aus IT-4 auf.

<a id="it-4"></a>

### IT-4 - Studierende Gruppen zuordnen

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | mehrere VMs für unterschiedliche Gruppen bereitstellen und möchte deshalb auswählen, welche Studierenden in welcher Gruppe sind, |
| **damit** | jede Gruppe gemeinsam in ihrer eigenen Umgebung arbeiten kann. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Beim Deployment lassen sich mehrere Gruppen (Teams) anlegen und benennen. |
| 2 | Jeder Studierende des Kurses lässt sich genau einer Gruppe zuordnen. |
| 3 | Studierende ohne Gruppe werden vor dem Start angezeigt. |
| 4 | Die Mitglieder einer Gruppe arbeiten gemeinsam in derselben Umgebung. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

<a id="it-5"></a>

### IT-5 - Einfache Bedienoberfläche

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | eine einfache Bedienoberfläche, |
| **damit** | kaum Zeit für die Erstellung der VMs verloren geht. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Ein Deployment führt durch wenige, klar erkennbare Schritte. |
| 2 | Man kann zwischen den Schritten zurückgehen, ohne Eingaben zu verlieren. |
| 3 | Fehlende Angaben werden direkt im jeweiligen Schritt angezeigt. |
| 4 | Fachbegriffe aus OpenStack oder Terraform erscheinen nicht ohne Erklärung. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | teilweise | ja | teilweise |

Anmerkung: „einfach“ ist nicht messbar — die Story ist eher eine Qualitätsanforderung an alle anderen Stories, erst die Kriterien machen sie prüfbar. Überschneidet sich mit BWL-6.

<a id="it-6"></a>

### IT-6 - VMs untereinander vernetzen

| | |
|---|---|
| **Als** | IT Dozent |
| **möchte ich** | dass die einzelnen VMs bei Bedarf untereinander vernetzt sind, |
| **damit** | ich Netzwerk- und Client-Server-Szenarien im Kurs durchspielen kann. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Die VMs eines Deployments liegen bei Bedarf in einem gemeinsamen Netzwerk und erreichen sich gegenseitig über ihre privaten IP-Adressen. |
| 2 | Die IP-Adressen aller VMs eines Deployments sind für die Lehrperson in der Infrastruktur-Ansicht sichtbar. |
| 3 | Sollte das Feature nicht aktiviert sein, sollen die VMs sich nicht gegenseitig erreichen. |

**INVEST**

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| teilweise | ja | ja | ja | ja | ja |

Anmerkung: IT-6 und BWL-4 beschreiben dieselbe Einstellung von zwei Seiten (vernetzt / getrennt) -> sollten zusammen umgesetzt werden.

---

## BWL Dozent

<a id="bwl-1"></a>

### BWL-1 - Informationen vor der Installation

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | vor der Installation Informationen zu einer App oder VM sehen können, |
| **damit** | ich beurteilen kann, ob sie für meine Lehrveranstaltung geeignet ist. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | |
| 2 | |
| 3 | |

<a id="bwl-2"></a>

### BWL-2 - Mit wenigen Klicks starten

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | mit wenigen Klicks eine vorkonfigurierte Lernumgebung starten können, |
| **damit** | ich ohne großen Konfigurationsaufwand direkt mit der Lehrveranstaltung beginnen kann. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | |
| 2 | |
| 3 | |

<a id="bwl-3"></a>

### BWL-3 - Empfehlungen und Hilfe bei der Konfiguration

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | Empfehlungen bzw. Hilfe für Konfigurationen haben, |
| **damit** | ich die richtigen Einstellungen für meinen Use-Case treffen kann (z. B. Tooltips, vorausgefüllte Standardwerte, „Empfohlen“-Labels bei Optionen). |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | |
| 2 | |
| 3 | |

<a id="bwl-4"></a>

### BWL-4 - Übungsdatensätze zentral ablegen

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | Übungsdatensätze in Form von Dateien auf der/den VM/VMs ablegen können, |
| **damit** | alle Studenten mit diesen Datensätzen starten können. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Während des Deployment-Prozess sollen Dateien hochgeladen werden können oder auf einen alternativen Weg verwiesen werden. |
| 2 | Jede Gruppe hat eine eigene Kopie der Dateien, um nicht die der anderen Gruppe verändern zu können. |
| 3 | |

<a id="bwl-5"></a>

### BWL-5 - Bedienung ohne technische Vorkenntnisse

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | eine Oberfläche ohne technische Vorkenntnisse bedienen können, |
| **damit** | der Fokus auf den fachlichen Inhalten bleibt, und nicht auf dem Einrichten der Umgebung. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Das Bereitstellen von virtuellen Maschinen soll auf dem "Golden Path" keine technischen Auswahlmöglichkeiten beinhalten, bspw. Security-Groups oder Netzwerke.  |
| 2 | Technische Begriffe auf der Seite eines Deployments standardmäßig verstecken oder umbennen, wie Git-Repository oder Image-ID. |

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

<a id="bwl-6"></a>

### BWL-6 - Integration in Moodle

| | |
|---|---|
| **Als** | BWL Dozent |
| **möchte ich** | ScholarStack direkt aus Moodle aufrufen können, |
| **damit** | ich alle meine Inhalte zur Vorlesung an einem Ort finde. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | ScholarStack erhält Accountdaten über Moodle.  |
| 2 | In Moodle wird eine Nutzeroberfläche von ScholarStack angezeigt. |

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

---

## Admin
*Der Administrator verwaltet den AppStore, betreibt diesen und ist Ansprechperson bei technischen Problemen.*

<a id="admin-1"></a>

### ADMIN-1 - Integration in den Self-Service
| | |
|---|---|
| **Als** | Administrator |
| **möchte ich** | eine nahtlose Integration von ScholarStack in den Self-Service |
| **damit** | der AppStore automatisch die OpenStack-Kontingente des Nutzers erkennt und mit diesen arbeiten kann. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | ScholarStack unterstützt mehrere OpenStack-Projekte pro Nutzeraccount.  |
| 2 | ScholarStack kann die Projekte des Nutzers und die Ressourcen dieser aus dem Self-Service auslesen, und diese dem Nutzer anzeigen. |

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

<a id="admin-2"></a>

### ADMIN-2 - Automatische Erstellung von Security-Groups
| | |
|---|---|
| **Als** | Administrator |
| **möchte ich** | automatisch passende Security-Groups zu den Apps in ScholarStack |
| **damit** | nur die unbedingt benötigten Ports der App aus dem passenden Netz erreichbar sind. |

**Akzeptanzkriterien**

| # | Kriterium |
|:-:|---|
| 1 | Jedes App-Template gibt an, welche Ports freigegeben werden müssen. |
| 2 | Es werden automatisch zu den Apps beim Deployment passende Security-Groups angelegt/zugeordnet. |
| 3 | Security-Groups eines Deployment können in der Nutzeroberfläche im Nachhinein angepasst werden. |

| I | N | V | E | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ja | ja | ja | ja | ja | ja |

