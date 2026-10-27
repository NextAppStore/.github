# Protokoll D — Sprachwechsel

**Name:** Till **Team:** Backend **Datum:** 27.10.2026 **Dauer:** ca. 10 Min

## Ablauf

### 1. Umschalter finden

| | |
|---|---|
| **Was du tust** | Die Sprachumschaltung suchen |
| **Erwartet** | Oben rechts, **DE / EN** |
| **Beobachtet** | - |
| **OK?** | Ja |

### 2. Auf EN stellen

| | |
|---|---|
| **Was du tust** | Auf EN klicken |
| **Erwartet** | Die Seite wechselt sofort, ohne Neuladen |
| **Beobachtet** | - |
| **OK?** | Ja |

## Durchgang auf Englisch

Geh die Bildschirme durch und notiere **jedes deutsche Wort**, das noch steht.

| Bildschirm | Deutsche Reste gefunden? | Welche |
|---|---|---|
| Dashboard | ja | Instanzen |
| Deployments (Liste) | ja | Beschreibungen nur auf Deutsch<br>Release Tag statt release day |
| Deployment-Detail | nein | |
| Apps | nein | |
| App-Detail | nein | |
| Kurse | nein | |
| Wizard Schritt 1–4 | nein | |
| Profil / Einstellungen | nein | |

**Tipp:** Auch auf Knöpfe zeigen, ohne zu klicken — die Tooltips sind leicht zu übersehen.

## Besondere Stellen

Diese wurden zuletzt angefasst und sind die wahrscheinlichsten Fundstellen.

### Datumsangaben

| | |
|---|---|
| **Was du tust** | Datum im Deployment-Detail und auf der Profilseite ansehen, dann Sprache wechseln |
| **Erwartet** | Auf DE `27.10.2026`, auf EN `10/27/2026` — das Format **ändert sich** |
| **Beobachtet** | In der Deployment Detail Page -> nein;<br>Übersichtspage -> ja |
| **OK?** | ☐ |

### Deployment-Status

| | |
|---|---|
| **Was du tust** | Status in der Deployment-Liste lesen |
| **Erwartet** | Ein echtes Wort wie "erfolgreich" bzw. "successful" — **nicht** `success` oder `pause_failed` |
| **Beobachtet** | - |
| **OK?** | ☐ |

### Variablen-Schritt

| | |
|---|---|
| **Was du tust** | Im Wizard auf Schritt 3, auf das ⓘ neben einer Variable klicken |
| **Erwartet** | Das Popover öffnet sich und nennt Packer oder Terraform |
| **Beobachtet** | - |
| **OK?** | ☐ |

### Anmeldebildschirm

| | |
|---|---|
| **Was du tust** | Abmelden und den Login-Bildschirm auf EN ansehen |
| **Erwartet** | Englisch |
| **Beobachtet** | - |
| **OK?** | ☐ |

## Hält die Sprache?

| | |
|---|---|
| **Was du tust** | Auf EN stellen, Seite neu laden (F5) |
| **Erwartet** | Bleibt auf EN |
| **Beobachtet** | - |
| **OK?** | ☐ |

| | |
|---|---|
| **Was du tust** | Auf EN abmelden, wieder anmelden |
| **Erwartet** | Bleibt auf EN |
| **Beobachtet** | - |
| **OK?** | ☐ |

## Zurück auf Deutsch

| | |
|---|---|
| **Was du tust** | Auf DE zurückstellen, dieselben Bildschirme kurz überfliegen |
| **Erwartet** | Keine **englischen** Reste |
| **Beobachtet** | - |
| **OK?** | ☐ |

**Englische Reste in der deutschen Ansicht:**

# Protokoll — EN Sprachcheck

## Startseite

![[Pasted image 20261027100416.png]]

- Instanzen ist Deutsch

## Profil Page

![[Pasted image 20261027095139.png]]

- UserID rutscht aus der Box raus

## Apps

![[Pasted image 20261027095320.png]]

- Beschreibung der VM's liegt nur auf Deutsch vor aktuell
- Sollten da auch ne englische Version abspeichern

## Deployments

- bei Wechsel von EN auf DE kommen Bindestriche hinzu -> sollten wir Einheitlich machen

![[Pasted image 20261027102312.png]]

- Release Tag

## Allgemein

- Bei der deutschen Version sollten wir schauen ob wir etwas weniger Anglizismen verwenden
- Datum ändert sich in der Deployment detailpage nicht auf das deutsche Format
