# Kickr Steuerung – Proof of Concept

Minimale Single-Page-Web-App zur Steuerung eines Wahoo Kickr Core per Web Bluetooth (FTMS – Fitness Machine Service). Kein Build-Schritt, keine Abhängigkeiten – alles steckt in `index.html`.

## GitHub Pages aktivieren

1. Dieses Verzeichnis (mit `index.html` und `README.md`) in ein GitHub-Repository pushen, z. B. auf den Branch `main`.
2. Im Repository zu **Settings → Pages** gehen.
3. Unter **Build and deployment** die Quelle auf **Deploy from a branch** stellen.
4. Branch `main` und Ordner `/ (root)` auswählen, speichern.
5. Nach kurzer Zeit ist die App unter `https://<benutzername>.github.io/<repository>/` erreichbar.

Web Bluetooth benötigt einen sicheren Kontext (HTTPS). GitHub Pages liefert automatisch HTTPS aus, daher ist keine weitere Konfiguration nötig.

## Testen in Bluefy auf dem iPad

1. **Wichtig:** Wahoo-App, Zwift, TrainerRoad, Garmin Connect & Co. auf dem iPad vollständig schließen (auch aus dem App-Switcher entfernen). Nur eine App darf gleichzeitig eine aktive Bluetooth-Verbindung zum Kickr halten.
2. Den Kickr Core einschalten (z. B. kurz in die Pedale treten, bis die LED leuchtet).
3. [Bluefy](https://apps.apple.com/app/bluefy-web-ble-browser/id1492822055) auf dem iPad öffnen (Bluefy unterstützt Web Bluetooth, Safari nicht).
4. In Bluefy die GitHub-Pages-URL der App aufrufen.
5. Auf **„Mit Trainer verbinden“** tippen und in der erscheinenden Geräteliste den Kickr auswählen.
6. Nach erfolgreicher Verbindung erscheinen Live-Werte für Leistung, Kadenz und Geschwindigkeit, sobald getreten wird.
7. Zielleistung über die Buttons `-10 / -5 / +5 / +10` anpassen oder direkt mit **„Setzen“** senden. **„Stop“** pausiert den ERG-Modus.
8. Im Bereich **„Workout“** die eigene FTP eintragen und eines der drei vorbereiteten Workouts (LIT, MIT, HIT) auswählen und starten. Während des Workouts zeigt ein Diagramm das gesamte Leistungsprofil inkl. Fortschrittslinie, dazu Zielwatt, %FTP und verbleibende Zeit. Die Zielleistung wird automatisch passend zum Profil an den Trainer gesendet; „Pause“ hält nur den Timer an, „Abbrechen“ beendet das Workout und stoppt den ERG-Modus.
9. Das Debug-Log am unteren Rand (aufklappbar) zeigt alle gesendeten/empfangenen Rohdaten (Hex) sowie Control-Point-Antworten – hilfreich bei Verbindungsproblemen.

## Dinge, die sich nur am echten Trainer prüfen lassen

- Ob der Kickr sich unter dem Namen/Filter `fitness_machine` überhaupt im `requestDevice`-Dialog findet (manche Firmware-Versionen zeigen zusätzliche/abweichende Service-UUIDs).
- Ob Request Control (`0x00`) und Start/Resume (`0x07`) tatsächlich in dieser Reihenfolge mit Erfolgscode `0x01` quittiert werden, oder ob der Kickr eine andere Reihenfolge/Wartezeit erwartet.
- Das tatsächliche Flags-Bitmuster der Indoor-Bike-Data-Pakete des Kickr Core (welche Felder er wirklich sendet) und ob die Offsets/Auflösungen (insb. Kadenz × 0,5, Geschwindigkeit × 0,01) korrekt interpretiert werden.
- Das Zeitverhalten beim Setzen der Zielleistung: reagiert der Trainer sofort, oder gibt es eine spürbare Verzögerung/Rampe bis zur neuen Leistung?
- Verhalten bei schnellen, wiederholten Zieländerungen (Debounce von 300 ms) – ob die Queue-Serialisierung ausreicht, um „GATT operation already in progress“-Fehler zu vermeiden.
- Ob Stop/Pause (`0x08`, Parameter `0x01`) den Widerstand tatsächlich freigibt oder ob der Kickr danach einen erneuten Start-Befehl erwartet.
- Verhalten bei Verbindungsabbruch während der Fahrt (`gattserverdisconnected`) und ob „Erneut verbinden“ zuverlässig wieder funktioniert, ohne dass der Kickr vorher manuell zurückgesetzt werden muss.
- Ob die Fitness Machine Status-Characteristic (`0x2ADA`) vom Kickr Core überhaupt angeboten wird.
- Wake-Lock-Verhalten in Bluefy über einen längeren Zeitraum (bleibt der Bildschirm wirklich dauerhaft an?).
- Tatsächliche Signalreichweite/-stabilität zwischen iPad und Kickr am Trainingsplatz.
