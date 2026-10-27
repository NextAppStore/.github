# Protokoll F — Admin

**Name:** Michelle **Team:** Backend **Datum:** 27. Oktober **Dauer:** ca. 40 Min

## Ablauf

### 1. Version freigeben

Vorher: **Person B** hat als Dozentin eine neue App eingereicht und Versionen dieser App requested.

| | |
|---|---|
| **Was du tust** | Die eingeritete App und die requested Versionen prüfen und **freigeben** |
| **Erwartet** | Du findest sie ohne Hilfe; der Status steht auf "Wartet auf Freigabe" |
| **Beobachtet** | War sehr intuitiv zu finden und Person B hat das auch direkt gesehen |
| **OK?** | Ja |

**Rückmeldung an Person B:** Sieht sie den neuen Status? Ja, aber musste neu laden

### 2. Standard-Netzwerk auf DHBWv6

| | |
|---|---|
| **Was du tust** | Zu den Plattform-Einstellungen, Standard-Netzwerk auf **DHBWv6** setzen |
| **Erwartet** | Die Auswahl zeigt echte Netzwerke aus OpenStack; nach dem Speichern steht DHBWv6 als Standard |
| **Beobachtet** | Es gibt DHBWv6, welches schon eingestellt war und NAT in der Auswahl. Wie zu erwarten hat das erneute hinzufügen des Standard Netzwerks nichts verändert. |
| **OK?** | Ja |

### 5. VM-Topologie der Apps prüfen

Jede App soll Topologien als Metadaten hinterlegt haben.
Geh die Apps durch und trage ein, was tatsächlich angezeigt wird.

| App | Soll | Angezeigt | OK? |
|---|---|---|---|
| Ubuntu-App | Eine gemeinsame VM | Nicht gefunden | Nein |
| Jupyter-Notebook | Eine gemeinsame VM | Nicht gefunden | Nein |
| pgAdmin | Eine VM pro Team | Nicht gefunden | Nein |
| Web-LaTeX | Eine VM pro Team | Nicht gefunden | Nein |
| Online-IDE | Eine VM pro Team | Nicht gefunden | Nein |
| Kubernetes-App | Eine VM pro Team | Nicht gefunden | Nein |
| Windows-App | Eine VM pro Team | Nicht gefunden | Nein |
| windows-spss | Eine VM pro Team | Nicht gefunden | Nein |

**Apps, bei denen gar keine Topologie steht:**

Diese Methadaten waren nicht zu finden, bei egal welcher App.

**War ohne Erklärung verständlich, was die Angabe bedeutet?** Nein.
