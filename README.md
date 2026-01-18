# 📘 device monitoring notification

Dieser Blueprint überwacht ein Gerät anhand eines Leistungssensors und erkennt zuverlässig:

- **Start**  
- **Fertig**  
- **Neustart von Home Assistant** (Zustand wird korrekt rekonstruiert)

Er eignet sich ideal für:

- Waschmaschine  
- Trockner  
- Geschirrspüler  
- Andere Geräte mit typischem Leistungsprofil  

Der Blueprint nutzt einen **Status-Helfer** (`input_select`) als interne Zustandsmaschine.

---

# 🧱 Voraussetzungen

Bevor der Blueprint verwendet werden kann, müssen **zwei Dinge** vorhanden sein:

---

## 1️⃣ Leistungssensor

Ein Sensor, der die aktuelle Leistung des Geräts misst, z. B.:

- `sensor.waschmaschine_power`
- `sensor.trockner_power`

Der Sensor muss **Watt-Werte** liefern.

---

## 2️⃣ Status-Helfer (input_select)

Der Blueprint benötigt einen eigenen `input_select`, der den aktuellen Zustand des Geräts speichert.

### So legst du ihn an:

**direkt im Blueprint** oder unter:

**Einstellungen → Geräte & Dienste → Helfer → Helfer hinzufügen → Auswahl (input_select)**

Name z. B.:

- *Waschmaschine OG Status*  
- *Trockner Keller Status*

### Optionen (diese müssen GENAU SO benannt sein!):

```
idle
running
finished
```

### Beispiel YAML (optional):

```yaml
input_select:
  waschmaschine_og_status:
    name: Waschmaschine OG Status
    options:
      - idle
      - running
      - finished
```

---

# ⚙️ Blueprint-Konfiguration

Beim Anlegen einer Automation aus diesem Blueprint müssen folgende Eingaben gesetzt werden:

---

## 🔧 Pflichtfelder

| Feld | Beschreibung |
|------|--------------|
| **Leistungssensor** | Sensor, der die aktuelle Leistung misst |
| **Status-Helfer** | Das zuvor angelegte `input_select` |

---

## 🔧 Optionale Einstellungen

| Feld | Standard | Beschreibung |
|------|----------|--------------|
| Start-Schwelle | 8 W | Ab welcher Leistung das Gerät als gestartet gilt |
| Start-Hysterese | 2 min | Wie lange die Leistung über der Start-Schwelle bleiben muss |
| Fertig-Schwelle | 5 W | Unter welcher Leistung das Gerät als fertig gilt |
| Fertig-Hysterese | 3 min | Wie lange die Leistung unter der Fertig-Schwelle bleiben muss |

---

## Abbildung:

![Blueprint Trockner](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/blueprint_trockner.png)

Dropdown-Menü erstellen (Helfer)

![Dropdown-Menü erstellen](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/Dropdown-Menue%20erstellen.png)

---

## 🔔 Aktionen

Der Blueprint bietet zwei Aktionsblöcke:

### Aktionen bei Start
Wird ausgeführt, wenn das Gerät sicher gestartet ist.  
Beispiele:

- Push-Nachricht  
- Licht einschalten  
- Log-Eintrag  

### Aktionen bei Fertig
Wird ausgeführt, wenn das Gerät sicher fertig ist.  
Beispiele:

- Push-Nachricht  
- Ton abspielen  
- Licht blinken  

---

# 🔄 Ablauf / Funktionsweise

1. **Start erkannt**  
   Leistung steigt über die Start-Schwelle → Status wird `running`.

2. **Fertig erkannt**  
   Leistung fällt unter die Fertig-Schwelle → Status wird `finished`.

3. **Automatisches Zurücksetzen**  
   Nach 2 Minuten im Zustand `finished` → Status wird wieder `idle`.

4. **Neustart von Home Assistant**  
   Der Blueprint prüft die aktuelle Leistung und setzt den Status korrekt:

   - über Start-Schwelle → `running`  
   - unter Fertig-Schwelle → `finished`  
   - sonst → `idle`

---

# 📝 Hinweise

- Der Status-Helfer muss **pro Gerät** einmal angelegt werden.  
- Mehrere Geräte können parallel überwacht werden (jeweils eigene Automation + eigener Helfer).  
- Der Blueprint erzeugt **keine** Entities automatisch — Home Assistant erlaubt das nicht.  

---

# 🧩 Attribution

Teile dieses Blueprints wurden inspiriert durch:  
https://gist.github.com/sbyx/6d8344d3575c9865657ac51915684696

---

# Typische Einstellwerte
z.B. Wärmepumpentrockner:

![Wärmepumpentrockner Leistung](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/waermepumpentrockner_leistung.png)

Einstellwerte:

![Blueprint Trockner](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/blueprint_trockner.png)

z.B. Waschmaschine:

![Waschmaschine Leistung](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/waschmaschine_leistung.png)

Einstellwerte:

![Blueprint Waschmaschine](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/blueprint_waschmaschine.png)

---

# Benachrichtigungsgruppen:

In configuration.yaml z.B. folgendes für eine Benachrichtigungsgruppe eintragen:

```
notify:
  - platform: group
    name: notify_home
    services:
      - service: mobile_app_21081111rg 
      - service: alexa_media_echo_dot
```

Achtung! Das Mobiltelefon mit der home assistant App und die Alexa Geräte müssen exakt wie vorhanden (aber ohne notify.) benannt werden!
Am besten funktioniert das unter Entwicklerwerkzeuge - Aktionen. Dann notify eintragen und das Gerät in der Liste suchen.

![Benachrichtigungsgruppe Namen finden](https://raw.githubusercontent.com/miau08/device-monitoring-notification/main/benachrichtigungsgruppe_namen%20finden.png)
