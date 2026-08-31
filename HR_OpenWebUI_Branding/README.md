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


## 6. Docker Image erstellen und nach GitHub (GHCR) pushen

Um das fertige Image in die GitHub Container Registry (GHCR) zu übertragen, sind ein Zugriffstoken (PAT) sowie die passenden Build-Befehle notwendig.

### Schritt 6.1: GitHub Personal Access Token (PAT) erstellen
1. Melde dich bei GitHub an und gehe zu den **Settings** (Einstellungen deines Profils).
2. Scrolle links ganz nach unten zu **Developer Settings**.
3. Wähle **Personal access tokens** -> **Tokens (classic)**.
4. Klicke auf **Generate new token** -> **Generate new token (classic)**.
5. Gib dem Token einen Namen (z. B. `GHCR Access`) und wähle die Berechtigungen **write:packages** und **read:packages** (das Recht `repo` wird oft automatisch mit ausgewählt).
6. Klicke unten auf **Generate token** und kopiere das Token sofort an einen sicheren Ort (es wird danach nicht mehr angezeigt).

### Schritt 6.2: Im Terminal bei GHCR anmelden
Nutze das kopierte Token, um dich im Terminal bei der GitHub Registry anzumelden:
```bash
echo "DEIN_KOPIERTES_TOKEN" | docker login ghcr.io -u DEIN_GITHUB_BENUTZERNAME --password-stdin
```

### Schritt 6.3: Image Version im Dockerfile anpassen
Passe vor dem Build die Image-Version im `Dockerfile` an:
```dockerfile
ghcr.io/markusknauer-hr/open-webui-hr:VersionsNummer
```
*Ersetze `VersionsNummer` durch die aktuell gewünschte Version.*

### Schritt 6.4: Image lokal bauen und pushen
Erstelle das Image lokal (ersetze `NAMESPACE` durch den Namen deines persönlichen GitHub-Kontos oder der Organisation):
```bash
docker build -t ghcr.io/NAMESPACE/open-webui-hr:VersionsNummer .
```

Pushe das fertige Image im Anschluss in das GitHub Repository:
```bash
docker push ghcr.io/NAMESPACE/open-webui-hr:VersionsNummer
```


## 7. Nach Änderungen

Nach Änderungen an Branding-Dateien, Frontend-Dateien oder Docker-Konfiguration Open WebUI neu bauen und neu starten.

Nach Änderungen an Favicons oder anderen statischen Dateien zusätzlich einen Hard-Reload im Browser durchführen, da diese Dateien häufig gecacht werden.