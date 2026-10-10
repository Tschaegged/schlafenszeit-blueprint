# 🛏️ Schlafenszeit per RFID-Tag – Home Assistant Blueprint

Ein Blueprint, das beim Scannen eines RFID/NFC-Tags (z. B. am Bett) automatisch
Lichter ausschaltet, Heizzonen in den Eco-Modus zwingt und Rollläden schließt.

## Features

- **Frei wählbarer Tag** – jede beliebige registrierte Tag-ID als Auslöser
- **Frei wählbare Lichter, Gruppen und Schalter** – werden ausgeschaltet
- **Frei wählbare Rollläden/Jalousien** – werden vollständig geschlossen
- **Frei wählbare Heizzonen** – Eco-Modus wird erzwungen
- **Automatische Eco-Freigabe am Morgen** – zur eingestellten Uhrzeit werden alle gewählten
  Eco-Helfer wieder ausgeschaltet; eine wegen Neustart verpasste Freigabe wird nachgeholt
- Leere Auswahlen werden übersprungen, es entsteht kein Fehler
- `mode: single`, keine unnötige Logik – ein Scan, drei einfache Service-Calls

## 💡 Gemischte Auswahl beim Ausschalten

Beim Ausschalten sind neben einzelnen Lichtern auch **Gruppen** (`group.…`) und
**Schalter** (`switch.…`) erlaubt – praktisch für Lampen, die an einer Steckdose
hängen, oder für eine einzige Sammelgruppe „alle Lampen".

Ausgeschaltet wird über `homeassistant.turn_off`. Das ist bei gemischten Gruppen
entscheidend: `light.turn_off` nimmt eine `group.…` zwar entgegen, filtert danach
aber auf die `light`-Mitglieder – Steckdosen und Schalter in derselben Gruppe
blieben unbemerkt an.

> **Hinweis zu gemischten Gruppen:** Ein über die Oberfläche angelegter
> Gruppen-Helfer deckt immer nur *eine* Domain ab. Für eine Gruppe aus Lampen
> *und* lampensteuernden Steckdosen braucht es eine klassische YAML-`group:`
> in der `configuration.yaml`.

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

## ⏰ Automatische Eco-Freigabe am Morgen

Force-Eco hat in der Heizungs-Automation Vorrang vor dem Zeitplan. Würde der Helfer nie
wieder ausgeschaltet, käme kein Raum mehr auf Komforttemperatur. Deshalb schaltet das
Blueprint zur Uhrzeit **„Eco freigeben um“** (Standard 04:30) **alle** gewählten Eco-Helfer
wieder aus – unabhängig davon, wie viele Heizzonen eingetragen sind. Wähle die Zeit kurz
vor dem frühesten Heizbeginn aller Zonen.

Lief Home Assistant zur Freigabezeit nicht (Neustart, Update), wird die Freigabe beim
nächsten Start nachgeholt – sofern der letzte Scan des Tags vor der verpassten Freigabe
lag. Der Scan-Zeitpunkt kommt aus der Tag-Entität, denn `last_changed` der Helfer wird
bei einem Neustart auf die Startzeit gesetzt und taugt dafür nicht.

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
| Lichter, Gruppen & Schalter | Werden beim Scan ausgeschaltet (`light`, `group`, `switch`) | – |
| Heizungen (Eco erzwingen) | Force-Eco-`input_boolean`-Helfer der gewünschten Heizzonen | – |
| Eco freigeben um | Uhrzeit, zu der alle gewählten Eco-Helfer wieder ausgeschaltet werden (Standard 04:30) | – |
| Rollläden | Werden vollständig geschlossen | – |

## Lizenz

[MIT](LICENSE)
