# 🛏️ Schlafenszeit per RFID-Tag – Home Assistant Blueprint

Ein Blueprint, das beim Scannen eines RFID/NFC-Tags (z. B. am Bett) automatisch
Lichter ausschaltet, Heizzonen in den Eco-Modus zwingt und Rollläden schließt.

## Features

- **Frei wählbarer Tag** – jede beliebige registrierte Tag-ID als Auslöser
- **Frei wählbare Lichter** – werden ausgeschaltet
- **Frei wählbare Rollläden/Jalousien** – werden vollständig geschlossen
- **Frei wählbare Heizzonen** – Eco-Modus wird erzwungen
- Leere Auswahlen werden übersprungen, es entsteht kein Fehler
- `mode: single`, keine unnötige Logik – ein Scan, drei einfache Service-Calls

## 🌱 Wichtig: Wie der Eco-Modus funktioniert

Dieses Blueprint schaltet **keine Climate-Entitäten direkt**. Viele Heizungs-Setups
(z. B. Zigbee2MQTT-Thermostate) unterstützen keinen `climate.set_preset_mode`. Stattdessen
wird vorausgesetzt, dass die Heizungs-Automation pro Raum (z. B. das verbreitete
[`advanced_heating_control`](https://github.com/panhans/HomeAssistant/blob/main/blueprints/automation/panhans/advanced_heating_control.yaml)-Blueprint)
einen **Force-Eco-`input_boolean`-Helfer** über den Input `input_force_eco_temperature`
kennt. Dieses Blueprint schaltet genau diese Helfer per `input_boolean.turn_on` ein –
die jeweilige Heizungs-Automation übernimmt daraufhin selbst den Wechsel in den Eco-Modus.

**Voraussetzung:** Lege pro gewünschter Heizzone einen `input_boolean`-Helfer an und
hinterlege ihn in der zugehörigen Heizungs-Automation als `input_force_eco_temperature`.
Falls deine Heizungslösung `climate.set_preset_mode` unterstützt, kannst du das Blueprint
leicht anpassen und stattdessen direkt die Climate-Entitäten ansteuern.

## Installation

### Über HACS (empfohlen)

1. HACS öffnen → **Automations** → Drei-Punkte-Menü → **Custom repositories**
2. URL: `https://github.com/Tschaegged/schlafenszeit-blueprint` – Kategorie: **Automation**
3. Blueprint suchen und importieren

### Manuell

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/Tschaegged/schlafenszeit-blueprint/main/blueprints/automation/schlafenszeit_rfid.yaml)

Oder direkt in Home Assistant:
**Einstellungen → Automationen → Blueprints → Blueprint importieren** und folgende URL eingeben:

```
https://raw.githubusercontent.com/Tschaegged/schlafenszeit-blueprint/main/blueprints/automation/schlafenszeit_rfid.yaml
```

## Voraussetzungen

- Home Assistant **2024.10** oder neuer
- Ein registrierter RFID/NFC-Tag (`tag:`-Integration, standardmäßig über `default_config:` aktiv)
- Optional für den Eco-Teil: eine Heizungs-Automation pro Raum mit einem
  `input_force_eco_temperature`-Input (`input_boolean`- oder `binary_sensor`-Helfer)

## Optionen

| Option | Zweck | Pflicht |
|---|---|---|
| RFID/NFC-Tag | Tag-ID, deren Scan die Routine auslöst | ✅ |
| Lichter | Werden beim Scan ausgeschaltet | – |
| Heizungen (Eco erzwingen) | Force-Eco-`input_boolean`-Helfer der gewünschten Heizzonen | – |
| Rollläden | Werden vollständig geschlossen | – |

## Lizenz

[MIT](LICENSE)
