# Protokoll A — Der volle Weg

**Name:** Jannis Voß **Team:** Backend **Datum:** 27.10.2026 **Dauer:** ca. 20 Min

## Ablauf

Fülle **Beobachtet** nur aus, wenn etwas anders war als erwartet. Sonst reicht der Haken.

### 1. Anmelden

| | |
|---|---|
| **Was du tust** | Seite öffnen, anmelden |
| **Erwartet** | Du landest im Dashboard und siehst deinen Namen oben rechts |
| **Beobachtet** | |
| **OK?** | Ja |

### 2. Kurs anlegen

| | |
|---|---|
| **Was du tust** | Zu "Kurse", einen neuen Kurs anlegen |
| **Erwartet** | Kurs erscheint in der Liste |
| **Beobachtet** | |
| **OK?** | Ja |

### 3. Teilnehmer hinzufügen

| | |
|---|---|
| **Was du tust** | Kurs öffnen, mindestens zwei Studierende hinzufügen |
| **Erwartet** | Die Mitgliederzahl stimmt danach |
| **Beobachtet** | Kurs öffnen funktioniert sowohl mit dem Button "Details öffnen" als auch auf die Kachel klicken (doppelte Funktionalität, vielleicht Button einfach entfernen) |
| **OK?** | |

### 4. App auswählen

| | |
|---|---|
| **Was du tust** | Zu "Apps", eine App öffnen, Version wählen, auf Deployen |
| **Erwartet** | Der Wizard startet bei Schritt 1 |
| **Beobachtet** | App-Informationen und Versionsdetails sind ganz unten, vielleicht als seperaten Tab neben Übersicht und AppStore; AppStore Name für den Tab ist nicht intuitiv und aussagekräftig |
| **OK?** | |

*Weiterer Bug:* Wenn man eine App auswählt, und dann auf "Jetzt deployen", und dann unten links auf zurück klickt, passiert folgendes: Wenn ich nun oben links auf zurück zur Übersicht klicke, sollte man wieder zur Liste an Apps kommen, kommt aber wieder auf "Jetzt deployen".

### 4b. VM-Topologie der Apps prüfen

Jede App soll als Metadaten hinterlegt haben, wie viele VMs sie erzeugt.
Geh die Apps durch und trage ein, was angezeigt wird.

| App | Soll | Angezeigt | OK? |
|---|---|---|---|
| Ubuntu-App | Eine gemeinsame VM | | |
| Jupyter-Notebook | Eine gemeinsame VM | | |
| pgAdmin | Eine VM pro Team | | |
| Web-LaTeX | Eine VM pro Team | | |
| Online-IDE | Eine VM pro Team | | |
| Kubernetes-App | Eine VM pro Team | | |
| Windows-App | Eine VM pro Team | | |
| windows-spss | Eine VM pro Team | | |

**Falls bei einer App gar nichts steht:** eintragen, welche — das ist der eigentliche Befund.

*Habe hier grundsätzlich die Anzeige überhaupt nicht gefunden*

### 5. Wizard — Schritt 1 bis 4

| | |
|---|---|
| **Was du tust** | Namen vergeben, Kurs und Studierende wählen, Teams bilden, Variablen durchgehen, Übersicht prüfen |
| **Erwartet** | Du kommst ohne Rückfragen durch; Pflichtfelder sind erkennbar; die Übersicht zeigt, was du eingegeben hast |
| **Beobachtet** | |
| **OK?** | Ja |

**Zusatzfrage:** Hast du verstanden, was "Erweiterte Einstellungen" bei den Variablen bedeutet? Ja

### 6. Deployen

| | |
|---|---|
| **Was du tust** | Auf "Deployen" klicken |
| **Erwartet** | Du landest in der Detailansicht, der Fortschritt läuft sichtbar los |
| **Beobachtet** | |
| **OK?** | Ja |

### 7. Während es läuft

| | |
|---|---|
| **Was du tust** | Beobachten: Phasen-Anzeige, Live-Logs |
| **Erwartet** | Die Phase ändert sich, Logzeilen laufen durch, der Fortschritt wirkt plausibel |
| **Beobachtet** | |
| **OK?** | Ja |

### 8. Nach dem Deployment

| | |
|---|---|
| **Was du tust** | Zugangsdaten der Teams ansehen |
| **Erwartet** | Pro Teilnehmer gibt es Zugangsdaten; Kopieren funktioniert |
| **Beobachtet** | Wenn man das Passwort unhided, verschiebt sich die SSH links nach links, vielleicht hier optimieren |
| **OK?** | Ja |

## Zum Schluss

**Wo hast du gezögert oder überlegen müssen?**
/

**Was hat gefehlt, das du erwartet hättest?**
/
