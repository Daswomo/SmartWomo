# SmartWomo 8-Kanal-Dimmer (PlatformIO / Arduino)

Eigenständige Firmware für Wemos D1 mini ESP32 + PCA9685 (I2C SDA 18 / SCL 19).
WLAN- und MQTT-Daten werden **nicht** in den Code geschrieben, sondern nach dem Flashen im Browser eingegeben.

## Funktionen
- 8 dimmbare Ausgänge OUT1–OUT8 (PCA9685 Kanal 0–7, 250 Hz, Gamma 2.8 wie ESPHome)
- 8 Eingänge IN1–IN8 (PCF8574, 0x20): schalten OUT1–OUT8 um, Modus je Eingang wählbar (Aus / Schalter / Taster / Nur melden), in HA als Binärsensor
- **Eigene Namen** für alle Ausgänge und Eingänge (Setup-Seite) – erscheinen in Home Assistant und auf der Startseite
- Zwei WLANs möglich (z. B. Womo und Zuhause) – wechselt automatisch
- **Setup-Portal als Access Point** – WLAN (mit Netzwerksuche), MQTT, feste IP, Gerätename, Admin-Passwort
- Home Assistant per **MQTT Auto-Discovery**: 8 Lichter (nur Helligkeit), Neustart-Button, Spannungssensor
- Letzter Zustand (Ein/Aus + Helligkeit) bleibt nach Stromausfall erhalten
- Ausgänge funktionieren auch ohne WLAN/MQTT – im Setup-AP auch per Handy schaltbar
- OTA-Update über WLAN – aus VS Code **oder im Browser** unter `http://<IP>/update` (Datei `smartwomo-dimmer-ota.bin` hochladen)

## Flashen – eine Datei
`firmware/smartwomo-dimmer-factory.bin` enthält alles (Bootloader, Partitionen, Firmware).

**Im Browser (Chrome/Edge, ohne Installation):**
1. ESP32 per USB anschließen.
2. https://espressif.github.io/esptool-js/ öffnen → *Connect* → Port wählen.
3. Flash Address **0x0**, Datei `smartwomo-dimmer-factory.bin` wählen → *Program*.
   Beim ersten Mal vorher *Erase Flash*.

Alternativ: https://web.esphome.io → *Connect* → *Install* → Datei wählen.

**Per Kommandozeile:**
`esptool.py --chip esp32 write_flash 0x0 smartwomo-dimmer-factory.bin`

## Einrichten
1. Nach dem Flashen öffnet der ESP das WLAN **SmartWomo-Setup-XXXX**, Passwort `smartwomo`.
2. Mit dem Handy verbinden – die Setup-Seite öffnet sich automatisch
   (sonst http://192.168.4.1 aufrufen).
3. WLAN suchen und auswählen, Passwort, MQTT-Broker (z. B. IP des Home Assistant), Benutzer/Passwort eintragen.
4. *Speichern & neu starten* – das Gerät erscheint in HA unter MQTT als „SmartWomo Dimmer“.

Leere Passwortfelder behalten den gespeicherten Wert.

## Setup-AP wieder öffnen
- WLAN 60 s nicht erreichbar → AP startet automatisch (Ausgänge laufen weiter).
- **3× kurz hintereinander Strom aus/an** (jeweils innerhalb von 10 s) → AP für 10 Minuten.
- Im Heimnetz: `http://<IP>/setup` (mit Admin-Passwort, Benutzer `admin`).
- *Werkseinstellungen* auf der Setup-Seite löscht alles.

## Selbst bauen
- PlatformIO: `pio run -e usb` → erzeugt automatisch `firmware/smartwomo-dimmer-factory.bin`
  und `firmware/smartwomo-dimmer-ota.bin` (für OTA-Updates).
- Direkt flashen: `pio run -e usb -t upload`
- Per WLAN: in `platformio.ini` bei `[env:ota]` `--auth=` = Admin-Passwort, dann `pio run -e ota -t upload`
- Seriell: `pio device monitor` (115200 Baud)
- GitHub: `.github/workflows/build.yml` baut bei jedem Push; ein Tag `v1.1.0` erzeugt ein Release mit den .bin-Dateien.

Hardware-Einstellungen (Pins, Adresse, Frequenz, AP-Passwort, Sensor) in `include/config.h`.

## Home Assistant
Voraussetzung: MQTT-Broker (z. B. Mosquitto-Add-on) und MQTT-Integration.
**Vorher die alte ESPHome-Version in HA entfernen**, sonst gibt es die Ausgänge doppelt.
Entitäten: `light.<geräte-id>_out1` … `_out8`.

## MQTT-Topics (Geräte-ID `smartwomo`)
| Zweck | Topic |
|---|---|
| Verfügbarkeit | `smartwomo/status` (online/offline) |
| Befehl Ausgang | `smartwomo/out1/set` – JSON `{"state":"ON","brightness":128}` oder `ON`/`OFF` |
| Zustand Ausgang | `smartwomo/out1/state` |
| Neustart | `smartwomo/restart/set` |
| Sensor | `smartwomo/adc/state` |
| Eingang | `smartwomo/in1/state` (ON/OFF) |

## HTTP-API
`/api/state`, `/api/set?ch=1&on=1&bri=128`

## Freischaltung

Jedes Gerät hat einen eigenen **8-stelligen Freischaltcode**, gebunden an seine MAC-Adresse.
**Die Freischaltung ist kostenlos.** Sie dient nur dazu, dass ich weiß, wer die Software nutzt:
Namen und MAC-Adresse an SmartWomoHA (Telegram-Gruppe) schicken, den Code bekommst du zurück.
Wer damit nicht einverstanden ist, nutzt die Software bitte nicht.
Die MAC steht im Setup unter **Freischaltung**; dort wird auch der Code eingetragen.
Ohne Code ist der Dimmer verbunden (Weboberfläche, MQTT, Eingänge werden angezeigt), aber **kein Ausgang lässt sich schalten** – alle bleiben aus.

Zum Bauen mit Freischaltung braucht es `include/license_secret.h` (geheim, steht in `.gitignore`).
Fehlt die Datei, lässt sich die Firmware trotzdem bauen; das Gerät bleibt dann gesperrt.
Logik: `include/license.h`.

## Nutzungsbedingungen

Die Software ist kostenlos und wird **ohne Haftung** bereitgestellt. Die Hardware kaufst du selbst und baust sie
selbst auf – auf eigenes Risiko. Unterstützung gibt es gern und freiwillig in der Telegram-Gruppe **SmartWomoHA**.
Details: [`NUTZUNGSBEDINGUNGEN.md`](NUTZUNGSBEDINGUNGEN.md)

## SBOM
`sbom.cdx.json` (CycloneDX 1.6) listet alle Komponenten mit Version und Lizenz.
Versionen sind in `platformio.ini` gepinnt und im Build verifiziert; bei Bibliotheks-Updates SBOM anpassen.

## Versionen

| Version | Änderungen |
|---|---|
| 1.5.0 | **Freischaltcode** (8 Zeichen, an die MAC gebunden, kostenlos; ohne Code verbunden, aber gesperrt), Nutzungsbedingungen |
| 1.4.1 | Stand vor der Freischaltung |
