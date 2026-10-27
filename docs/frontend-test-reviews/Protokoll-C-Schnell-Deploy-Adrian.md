# Protokoll C — Schnell-Deploy

**Name:** Adrian **Team:** Polzer **Datum:** 27.10.2026 **Dauer:** ca. 15 Min

## Ablauf

### 1. Den Knopf finden

| | |
|---|---|
| **Was du tust** | Zu "Apps". Ohne Hilfe den Schnell-Deploy suchen |
| **Erwartet** | Du findest ihn auf der App-Kachel |
| **Beobachtet** | |
| **OK?** | ÓK |

**Wie lange gesucht?** 1 Sekunden · **War klar, was er tut?** ☐ ja ☐ nein

Ja es war klar was er tut

### 2. Teilnehmer-Dialog

| | |
|---|---|
| **Was du tust** | Knopf drücken |
| **Erwartet** | Ein Dialog öffnet sich, in dem du **Kurs und Teilnehmer** wählst |
| **Beobachtet** | |
| **OK?** | OK |

**Was passiert, wenn du keinen Kurs wählst?**

Teilnehmerliste öffnet sich

### 3. Auswahl treffen

| | |
|---|---|
| **Was du tust** | Kurs wählen, dann mehrere Teilnehmer |
| **Erwartet** | Die Liste lädt; "Nur ich" ist ebenfalls wählbar und zeigt deinen Namen |
| **Beobachtet** | |
| **OK?** | OK |

### 4. Bestätigen

| | |
|---|---|
| **Was du tust** | Dialog bestätigen |
| **Erwartet** | Du landest direkt in der **Übersicht** (Schritt 4), nicht im Wizard-Anfang |
| **Beobachtet** | Ja schon außer für die Online-IDE für die noch kein quickdeploy angelegt war |
| **OK?** | OK |

### 5. Den Namen ansehen

| | |
|---|---|
| **Was du tust** | Den vorgeschlagenen Deployment-Namen lesen |
| **Erwartet** | So etwas wie `ubuntu-app-89me` — kein Datum darin |
| **Beobachtet** | |
| **OK?** | OK |

**Notiere den Namen:** ubuntu-app-34kj
**Findest du ihn brauchbar?** ☐ ja 

JA

### 6. Umbenennen

| | |
|---|---|
| **Was du tust** | Den Namen ändern, ohne in den Wizard zurückzugehen |
| **Erwartet** | Ein Stift neben dem Namen öffnet einen kleinen Dialog |
| **Beobachtet** | |
| **OK?** | OK, aber bei anpassung am ersten schritt muss man sich 4 mal ncohmal durch den wizzard klicken. Dies bitte ändern. |

**Hast du den Stift sofort gesehen?** ja 

Probiere dabei auch: **Enter zum Speichern**, **Abbrechen**, und einen **leeren Namen** eingeben.

| Variante | Erwartet | OK? |
|---|---|---|
| Enter drücken | speichert | OK |
| Abbrechen | Name bleibt wie vorher | OK |
| Leeren Namen speichern | wird abgelehnt | OK |

### 7. Übersicht prüfen

| | |
|---|---|
| **Was du tust** | Die ganze Übersicht durchlesen |
| **Erwartet** | Teams, Variablen und Werte stimmen; neben jeder Variable steht, ob sie zu Packer oder Terraform gehört |
| **Beobachtet** | |
| **OK?** | OK |

**Ist erkennbar, welche Teilnehmer du vorhin gewählt hast?** ☐ ja ☐ nein

Ja

### 8. Deployen

| | |
|---|---|
| **Was du tust** | Auf "Deployen" klicken |
| **Erwartet** | Es startet; du landest in der Detailansicht |
| **Beobachtet** | Während dem deployment stehen bei den LOGS noch Schritte wie Terraform init. Dies existiert nicht mehr und muss durch open tofu ersetzt werden. |
| **OK?** | Nein |

## Die eigentliche Frage

**Wie viele Klicks vom Apps-Bildschirm bis zum laufenden Deployment?** 3

**War es wirklich schneller als der volle Wizard?** Ja spürbar schneller


## Konsolenfehler

☐ nein

## Zum Schluss

**Wo hast du gezögert?**

**Was würdest du anders machen?**

Das mit Versionen beantragen einfügen?

## Screenshots

Mach ein Bild von **jeder Stelle, an der etwas nicht gestimmt hat**

Benennung: `<Buchstabe>-<Nummer>-<Stichwort>.png`, also etwa `A-01-leere-kursliste.png`. So lässt sich das Bild später dem Eintrag oben zuordnen.

| Datei | Gehört zu Schritt | Was ist darauf zu sehen |
|---|---|---|
| | | |
| | | |
| | | |
| | | |
| | | |

wizard anpassung

Terraform init usw. im deployment screen ändern
