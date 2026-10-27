# SmartWomo Truma

**Truma Heizung und Klimaanlage über den LIN-Bus steuern – ohne ESPHome, mit eigener Weboberfläche.**
Im Setup auswählen, was verbaut ist – Startseite, Home Assistant und MQTT passen sich automatisch an.

Ein Projekt der Telegram-Gruppe **SmartWomoHA** – *Help for free*.

> Das LIN-Protokoll stammt aus [truma_inetbox](https://github.com/Fabian-Schmidt/esphome-truma_inetbox)
> von **Fabian Schmidt** in der Fassung von [havanti/esphome-truma](https://github.com/havanti/esphome-truma).
> Die Dateien liegen unverändert in `lib/truma_inetbox`. Neu ist nur die Anbindung ohne ESPHome.
> Deshalb steht die ganze Firmware unter **GPL-3.0** (siehe `LICENSE`).

---

## Geräte – im Setup wählbar

| Auswahl | Auf der Startseite |
|---|---|
| Truma Combi 4/6 (Gas) | Heizung, Warmwasser |
| Truma Combi 4/6 E (Gas + 230 V) | + Energie: Gas / Mix 900·1800 W / Elektro 900·1800 W |
| Truma Combi D 4/6 (Diesel) | Heizung, Warmwasser, **Diesel ausbrennen** |
| Truma Combi D 6 E (Diesel + 230 V) | + Energie (Diesel / Mix / Elektro) und Diesel ausbrennen |
| Truma VarioHeat | Heizung mit Stufen Nacht / Auto / Boost |
| keine Heizung | – (z. B. nur Klimaanlage) |
| ☑ Truma Aventa | **Klimaanlage**: Aus / Lüften / Kühlen / Heizen / Auto, 16–31 °C, Lüfter, Licht |
| ☑ Klimaautomatik | CP Plus heizt und kühlt selbst (18–25 °C), experimentell |

Die Firmware vergleicht die Auswahl mit dem, was die CP Plus meldet, und zeigt einen Hinweis, wenn etwas nicht passt
(z. B. „Die CP Plus meldet eine Combi D“).

**Diesel ausbrennen** (Entkoken, aus dem havanti-Fork): Energie Diesel, 30 °C, Stufe High für 45 Minuten, danach aus.
Etwa einmal im Monat, dabei lüften. Startet der ESP währenddessen neu, schaltet er die Heizung danach sicher aus.
Wird die Heizung zwischendurch von Hand geändert, endet das Ausbrennen ohne Ausschalten.

Sendet die Anlage diese Werte, erscheinen sie zusätzlich: **Heizanforderung** und **Spannung an der Heizung** (Combi 4), **230 V an der Heizung** (Combi D), **Lüfterstufe** – reines Lüften 1–10 an der CP Plus wird angezeigt, lässt sich aber nicht schalten (siehe [`docs/lin-analyse-lueften.md`](docs/lin-analyse-lueften.md)).

## Funktionen

- **Heizung**: an/aus, Solltemperatur 5–30 °C, Stufe **Eco / High / Boost**
- **Warmwasser**: Aus / Eco 40 °C / Heiß 60 °C / Boost
- **E-Modelle**: Gas bzw. Diesel / Mix / Elektro mit 900 W oder 1800 W
- **Aventa**: Klimaanlage mit allen Betriebsarten, Lüfter und Licht
- Raum- und Wassertemperatur, Betriebsstatus, **Fehlercode** (z. B. E212)
- Die **CP Plus bleibt**: der ESP meldet sich dort als iNet-Box an
- **Setup-WLAN** zum Einrichten, **zweites WLAN** (z. B. Zuhause)
- **Home Assistant** per MQTT Auto-Discovery (Thermostat, Auswahlfelder, Sensoren)
- **Victron Cerbo GX / MQTT** über das MQTT-Schema (wie die SmartWomo-Gaswaage)
- **LIN-Log im Browser** – zur Fehlersuche ohne USB-Kabel
- **Nur mitlesen** (Observer) zum gefahrlosen Testen
- **OTA-Update** aus VS Code oder im Browser unter `/update`

## Hardware

| Teil | Hinweis |
|---|---|
| ESP32 (z. B. esp32dev / DevKit V1) | |
| LIN-Transceiver | z. B. TJA1020-Modul oder WomoLIN-Board |
| DC/DC 12 V → 5 V | für den ESP32 |
| RJ12-Kabel | zur freien Buchse an der CP Plus / am Verteiler |

| ESP32 | Transceiver |
|---|---|
| GPIO 17 (TX) | TXD |
| GPIO 16 (RX) | RXD |
| optional GPIO x | SLP / EN (im Setup als CS-Pin eintragen) |
| GND | GND |

Der Transceiver bekommt **12 V direkt vom Bordnetz** (nicht die 5 V).
Ausführliche Bauanleitung mit Bildern: [havanti/esphome-truma – Hardware](https://github.com/havanti/esphome-truma/tree/main/hardware).
Die Pins sind im Setup änderbar – Standard ist wie in der bisherigen ESPHome-YAML (TX 17, RX 16).

## Schnellstart

### 1. Flashen – eine Datei
`smartwomo-truma-factory.bin` aus den [Releases](../../releases) oder aus `firmware/` nach dem Build.

1. ESP32 per USB anschließen (Datenkabel).
2. In Chrome/Edge https://espressif.github.io/esptool-js/ öffnen → **Connect**.
3. **Erase Flash** (beim Umstieg von ESPHome unbedingt), dann Datei an **0x0** → **Program**.

### 2. Einrichten
1. Der ESP öffnet das WLAN **SmartWomo-Truma-XXXX** – Passwort **`smartwomo`**.
2. Handy verbinden, die Setup-Seite öffnet sich (sonst http://192.168.4.1).
3. WLAN wählen, MQTT-Broker eintragen, **MQTT-Format** wählen → **Speichern**.

Setup später: `http://<IP>/setup` (Benutzer `admin` + Admin-Passwort) –
oder **3 × kurz Strom aus/an** → Setup-WLAN für 10 Minuten.
Ohne WLAN startet das Setup-WLAN nach 60 s automatisch; die Heizung lässt sich dann über http://192.168.4.1 bedienen.

### 3. Erster Start an der CP Plus
Nach dem Einschalten meldet sich der ESP an der CP Plus an. Das dauert **bis zu einer Minute**.
Auf der Startseite steht dann *CP Plus: verbunden* und *Heizung: Combi …*.
Erst danach nimmt die Heizung Befehle an.

## Bedienung im Browser

`http://<IP>` – Raumtemperatur, Solltemperatur mit − / +, **An/Aus**, Stufe Eco/High/Boost,
Warmwasser, Status und Fehler. Ist die Heizung aus, merkt sich − / + nur die Temperatur für das nächste Einschalten.

## Home Assistant

Voraussetzung: MQTT-Integration und Broker (z. B. Mosquitto).
**Vorher die alte ESPHome-Version in HA löschen**, sonst ist alles doppelt.

| Entität | Typ |
|---|---|
| Heizung | Thermostat (aus/heizen, 5–30 °C, Lüfter = Eco/High/Boost) |
| Warmwasser | Auswahl: Aus / Eco 40 °C / Heiß 60 °C / Boost |
| Heizstufe | Auswahl: Eco / High / Boost |
| Energie | Auswahl (nur E-Modelle) |
| Diesel ausbrennen starten / abbrechen, Restzeit, läuft | Tasten, Sensor (nur Diesel) |
| Klimaanlage | Thermostat (Aus/Lüften/Kühlen/Heizen/Auto, 16–31 °C, Lüfter) |
| Aventa Licht | Zahl 0–5 |
| Klimaautomatik, Klimaautomatik aktiv | Zahl (0 = aus, 18–25 °C), Binärsensor |
| Raumtemperatur, Wassertemperatur | Sensor |
| Status, Fehlercode, Fehler (Text), Fehler – Abhilfe | Sensor |
| Heizungsfehler | Binärsensor |
| CP Plus verbunden, WLAN-Signal | Diagnose |
| Neustart | Taste |

MQTT direkt: `<geräte-id>/state` (JSON), Befehle an
`<geräte-id>/room_mode/set` (`heat`/`off`), `…/room_target/set` (`21`), `…/room_fan/set` (`Eco`/`High`/`Boost`),
`…/water/set` (`Aus`/`Eco 40 °C`/`Heiß 60 °C`/`Boost`), `…/mix/set`.

## Victron Cerbo GX / MQTT

Im Setup unter **MQTT-Format**: *MQTT-Bridge + Home Assistant*, *nur MQTT-Bridge* oder *nur Home Assistant*.
Broker = IP des Cerbo GX, **MQTT-Geräte-ID** = Seriennummer der MQTT-Bridge.

| Topic | Inhalt |
|---|---|
| `occ/<serial>/heater/truma_combi/config` | Selbstauskunft (retained) |
| `occ/<serial>/heater/truma_combi/state` | `room_on`, `room_target_c`, `room_temp_c`, `water_temp_c`, `heat_mode`, `water_mode`, `status`, `error`, `error_code`, `cp_plus_ok` |
| `occ/<serial>/heater/truma_combi/diagnostics` | Firmware, IP, WLAN, Heizungsmodell, LIN-Fehler |
| `occ/<serial>/heater/truma_combi/availability` | `online` / `offline` |
| `occ/<serial>/heater/truma_combi/cmd` | `{"room_on":true}` `{"room_target_c":21}` `{"heat_mode":"eco"}` `{"water_mode":"hot"}` `{"energy_mix":"fuel"}` `{"decoke":true}` |
| `occ/<serial>/aircon/truma_aventa/…` | Klimaanlage: `ac_mode`, `ac_target_c`, `ac_temp_c`, `ac_fan`, `ac_light`, `ac_auto_c` – Befehle per `cmd`, z. B. `{"ac_mode":"cool","ac_target_c":22}` |

Die Topic-Wurzel `occ` erwartet die MQTT-Bridge im Cerbo GX; sie steht als `MQTT_ROOT` in `include/config.h`.

Klassen (`heater`, `aircon`) und Modulnamen (`truma_combi`, `truma_aventa`) stehen in `include/config.h` und lassen sich dort an die MQTT-Bridge anpassen.

## Fehlercodes im Klartext

Meldet die Heizung einen Fehler, zeigt die Startseite oben eine rote Karte mit **Nummer, Ursache und Abhilfe**,
z. B. **E 517** – *Gasproblem: Flasche leer/zu, Regler vereist …* – *Abhilfe: Flasche wechseln/öffnen, EisEx …*.
Warnungen erscheinen als **W** (gelb), z. B. W 255 kurz nach dem Einschalten.

- Die Nummer kommt von der Heizung (zwei Bytes: high × 100 + low) – genau wie im Display der CP plus.
- Die Texte stammen aus der Truma-Fehlersuchanleitung (CP plus): Combi Gas (alt und ab 04/2018) und Combi D (Diesel)
  – getrennte Tabellen, weil z. B. 122 bei Gas und Diesel etwas anderes bedeutet. Tabelle: `include/error_codes.h`.
- Unbekannte Nummern werden trotzdem angezeigt („Unbekannter Code“).
- Die **Aventa** meldet über den LIN-Bus keinen Fehlercode – ihre Codes (1, 2, 4, 8, 17) stehen nur im Display der CP plus.

In Home Assistant: **Fehlercode** (z. B. „E 517“), **Fehler** (Text), **Fehler – Abhilfe** (Diagnose), **Heizungsfehler** (an/aus).
Per MQTT-Bridge: `error_code`, `error_number`, `error_text` im State.

## Nicht enthalten

Truma Cooler und Reifendruck-Sensoren (TPMS) aus dem havanti-Fork laufen über Bluetooth und brauchen ESPHome mit ESP-IDF – sie sind hier nicht dabei.

## LIN-Mitschnitt

Unter `http://<IP>/capture` (oder LIN-Log → **● Mitschnitt**) zeichnet die Firmware den LIN-Bus auf –
jeden Rahmen-Inhalt nur einmal je Abschnitt zwischen zwei Markierungen, dazu die Meldungen der Heizung.
Mit den Knöpfen setzt du Markierungen („Lüften Stufe 1“ … „10“, „Heizen Eco“ …), bei jeder Markierung wird
auch der erkannte Zustand mitgeschrieben. **⬇ Datei laden** liefert `truma-lin-mitschnitt.txt` (max. 47 KB).
So lassen sich unbekannte Funktionen der CP Plus nachvollziehen und einbauen.

## LIN-Log und Testen

- `http://<IP>/log` zeigt die letzten Meldungen live. Für die Fehlersuche *Alle LIN-Rahmen* wählen, danach zurück auf *Debug*.
- **Nur mitlesen**: Im Setup anhaken – der ESP hört auf dem Bus mit, sendet aber nichts.
  Gut zum ersten Test, ob die Verkabelung stimmt.

## Selbst bauen

VS Code mit **PlatformIO IDE**, Projektordner öffnen → **Build ✓**.

- `firmware/smartwomo-truma-factory.bin` – alles in einer Datei, an **0x0**
- `firmware/smartwomo-truma-ota.bin` – nur die App, für Updates

**Update per WLAN:** in `platformio.ini` unter `[env:ota]` IP und `--auth=` (Admin-Passwort) eintragen → *ota* → **Upload**.
Oder im Browser unter `/update` die `-ota.bin` hochladen.

## Projektstruktur

```
smartwomo-truma/
├── platformio.ini                 Build (USB und OTA), C++17
├── src/main.cpp                   WLAN, Setup, MQTT (HA + MQTT-Bridge), Web, Befehle
├── include/
│   ├── config.h                   Standardwerte (Pins, MQTT-Namen, Zeiten)
│   └── web_pages.h                Weboberfläche
├── lib/truma_inetbox/
│   ├── src/*.cpp|h                LIN-Protokoll (unverändert, GPL-3.0)
│   ├── src/esphome/…              kleine Ersatz-Schicht statt ESPHome
│   └── src/swlog.cpp              Logger für Serial und Browser
├── scripts/merge_bin.py           erzeugt die eine Flash-Datei
├── sbom.cdx.json                  Software-Stückliste (CycloneDX)
└── .github/workflows/build.yml    automatischer Build
```

## Fehlersuche

| Problem | Lösung |
|---|---|
| *CP Plus nicht verbunden* | 12 V am Transceiver? TX/RX vertauscht? Im LIN-Log „Alle LIN-Rahmen“ – kommen Rahmen an? |
| *Anmeldung an CP Plus …* bleibt stehen | bis 1 Minute warten; CP Plus kurz stromlos machen |
| Befehl kommt nicht an | Observer aus? Hinweis unter den Knöpfen beachten |
| Uhrzeit wird nicht übernommen | CP Plus C3 nimmt keine Uhrzeit an – Haken im Setup entfernen |
| Alles doppelt in Home Assistant | alte ESPHome-Integration löschen |

## Herkunft und Dank

Das LIN-Protokoll (CP Plus, Combi, Aventa) stammt aus
**[havanti/esphome-truma](https://github.com/havanti/esphome-truma)** – einem Fork von
**[Fabian-Schmidt/esphome-truma_inetbox](https://github.com/Fabian-Schmidt/esphome-truma_inetbox)**.
Übernommen ist Commit `c463d7d` (Komponente 1.0.46), unverändert in `lib/truma_inetbox` – Details in
[`lib/truma_inetbox/README.md`](lib/truma_inetbox/README.md). Danke an Fabian Schmidt und havanti!

Die Herkunft steht auch in der Firmware: auf der Startseite (unten), im Setup unter **Herkunft**,
beim Start im seriellen Log und in den MQTT-Diagnosedaten (`lin_source`).

## Freischaltung

Jedes Gerät hat einen eigenen **8-stelligen Freischaltcode**, gebunden an seine MAC-Adresse.
**Die Freischaltung ist kostenlos.** Sie dient nur dazu, dass ich weiß, wer die Software nutzt:
Namen und MAC-Adresse an SmartWomoHA (Telegram-Gruppe) schicken, den Code bekommst du zurück.
Wer damit nicht einverstanden ist, nutzt die Software bitte nicht.
Die MAC steht im Setup unter **Freischaltung**; dort wird auch der Code eingetragen.
Ohne Code verbindet sich der ESP mit der CP Plus und zeigt den Zustand an, aber **es lässt sich nichts steuern** (Web, MQTT, Home Assistant, Klimaautomatik).

Zum Bauen mit Freischaltung braucht es `include/license_secret.h` (geheim, steht in `.gitignore`).
Fehlt die Datei, lässt sich die Firmware trotzdem bauen; das Gerät bleibt dann gesperrt.
Logik: `include/license.h`.

## Nutzungsbedingungen

Die Software ist kostenlos und wird **ohne Haftung** bereitgestellt. Die Hardware kaufst du selbst und baust sie
selbst auf – auf eigenes Risiko. Unterstützung gibt es gern und freiwillig in der Telegram-Gruppe **SmartWomoHA**.
Details: [`NUTZUNGSBEDINGUNGEN.md`](NUTZUNGSBEDINGUNGEN.md)

## Versionen

| Version | Änderungen |
|---|---|
| 1.5.0 | **Freischaltcode** (8 Zeichen, an die MAC gebunden, kostenlos; ohne Code verbunden, aber gesperrt); Bezeichnungen vereinheitlicht: Setup, Weboberfläche, Log und Doku sagen jetzt „MQTT“ bzw. „MQTT-Bridge“; Funktion, Topics und gespeicherte Einstellungen bleiben gleich |
| 1.4.1 | Versuche (Heizmodus roh, Lüfterstufe einspeisen) wieder entfernt, Bibliothek wieder unverändert. Ergebnis: Lüften lässt sich am gemeinsamen Bus nicht schalten, weil die CP Plus jeden 0x20-Rahmen selbst mitsendet – siehe `docs/lin-analyse-lueften.md` |
| 1.4.0 | **Versuch:** Lüfterstufe 1–10 einspeisen – der ESP schickt nach jedem 0x20-Rahmen der CP Plus einen eigenen mit der gewählten Stufe (nur bei Raumheizung aus, max. 20 min); Mitschnitt zeigt die Rückmeldung der Heizung (0x21) |
| 1.3.3 | Knopf „Lüften aus“ auf der Startseite (die CP Plus nimmt den Aus-Befehl auch beim Lüften an); Ergebnis des Versuchs in `docs/lin-analyse-lueften.md` |
| 1.3.2 | Versuch auf der Mitschnitt-Seite: Heizmodus 0–15 roh an die CP Plus senden, um herauszufinden, ob sie darüber Lüften einstellt |
| 1.3.1 | Lüften an der CP Plus wird erkannt und angezeigt („Lüften Stufe 1–10“, HA, MQTT `vent_level`); Mitschnitt zeigt jeden Rahmen nur einmal je Abschnitt; Analyse in `docs/lin-analyse-lueften.md` |
| 1.3.0 | LIN-Mitschnitt mit Markierungen und Download, Spannung an der Heizung (PID 0x22, Combi 4) |
| 1.2.0 | Fehlercodes im Klartext mit Abhilfe (Gas und Diesel), W/E-Unterscheidung, eigene Fehlerkarte, HA-Sensoren „Fehler“ und „Fehler – Abhilfe“ |
| 1.1.1 | Herkunftshinweis auf Start- und Setup-Seite, im Log und in den MQTT-Diagnosedaten |
| 1.1.0 | Geräteauswahl im Setup (Combi, Combi E, Combi D, Combi D E, VarioHeat, Aventa, Klimaautomatik), Startseite/HA/MQTT passen sich an, Diesel ausbrennen, Hinweis bei abweichend erkannten Geräten |
| 1.0.0 | Erste Version: Heizung, Warmwasser, Combi E, Weboberfläche, LIN-Log, HA-Discovery, MQTT, OTA |

---

**SmartWomoHA** – Help for free · Fragen und Ideen gern in der Telegram-Gruppe.
