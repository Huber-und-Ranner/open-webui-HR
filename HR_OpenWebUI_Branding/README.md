# Huber & Ranner Rebranding für Open WebUI

Diese Anleitung dokumentiert alle Anpassungen, die ausgehend vom Originalzustand von Open WebUI für das Huber-&-Ranner-Rebranding vorgenommen werden müssen.

Ziel ist, dass das Repository auch von einer anderen Person gepflegt und das Rebranding nach einem Update oder einer Neuinstallation nachvollziehbar wiederhergestellt werden kann.


## 1. Branding-Dateien kopieren

Kopiere den Inhalt dieses Ordners – mit Ausnahme der `README.md` – nach:

```text
open_webui/static/static/
```

Dabei werden die vorhandenen Open-WebUI-Branding-Dateien durch die Huber-&-Ranner-Versionen ersetzt bzw. ergänzt.


## 2. Docker-Konfiguration anpassen

Füge in der `docker-compose.yml` unter `environment` folgende Einträge hinzu:

```yaml
- 'WEBUI_NAME=Huber&Ranner KI'
- 'ENABLE_PERSISTENT_CONFIG=False'
```

`ENABLE_PERSISTENT_CONFIG=False` sorgt dafür, dass die Werte aus der Konfiguration bzw. den Umgebungsvariablen verwendet werden und nicht durch bereits persistent gespeicherte Einstellungen überschrieben werden.


## 3. Anwendungsnamen anpassen

Passe in:

```text
backend/open_webui/env.py
```

die Behandlung von `WEBUI_NAME` so an, dass Open WebUI keinen zusätzlichen Namenszusatz anhängt.

Der relevante Bereich soll folgendermaßen aussehen:

```python
if WEBUI_NAME != 'Open WebUI':
    pass

# WEBUI_NAME += ' (Open WebUI)'
```

In Kombination mit:

```yaml
WEBUI_NAME=Huber&Ranner KI
```

wird dadurch der gewünschte Name der Anwendung verwendet.


## 4. Alte `her`-Theme-Blöcke aus `src/app.html` entfernen

Öffne:

```text
src/app.html
```

Entferne den alten `her`-Sonderfall in den Zeilen 66–68:

```javascript
} else if (localStorage.theme === 'her') {
    document.documentElement.classList.add('her');
    metaThemeColorTag.setAttribute('content', '#983724');
```

Entferne außerdem in den Zeilen 107–111 die Sonderbehandlung:

```javascript
if (document.documentElement.classList.contains('her')) {
    return;
}
```

Im CSS-Bereich am Ende der Datei alle Regeln entfernen, die mit `html.her` arbeiten. Im Original betrifft das insbesondere die Zeilen 207–209 sowie 223–248.

Die Huber-&-Ranner-Farben werden ausschließlich über `static/static/custom.css` gesteuert. Das alte `her`-Theme wird daher nicht mehr benötigt.


## 5. Hinweise bei Build-Problemen

Falls beim Build Speicherprobleme auftreten, kann in der `Dockerfile` folgende Zeile vorübergehend auskommentiert werden:

```dockerfile
ENV NODE_OPTIONS="--max-old-space-size=4096"
```

Diese Änderung nur dann vornehmen, wenn sie tatsächlich erforderlich ist.

## 6. Nach Änderungen

Nach Änderungen an Branding-Dateien, Frontend-Dateien oder Docker-Konfiguration Open WebUI neu bauen und neu starten.

Nach Änderungen an Favicons oder anderen statischen Dateien zusätzlich einen Hard-Reload im Browser durchführen, da diese Dateien häufig gecacht werden.