Cloudflare Tunnel Setup Checkliste
Diese Checkliste führt durch die Einrichtung eines Cloudflare Tunnels, um einen lokalen Dienst sicher im Internet bereitzustellen. Diese Methode macht offene Eingangsports in der Firewall überflüssig und verbirgt die IP-Adresse Ihres Servers.

1. Vorbereitung & Installation
[ ] Voraussetzungen prüfen:
Sie haben ein aktives Cloudflare-Konto.
Die DNS-Verwaltung Ihrer Domain erfolgt über Cloudflare.
Ihre Webanwendung läuft lokal (z. B. auf http://localhost:8080).
Sie haben sudo- oder Root-Zugriff auf den Server.
[ ] cloudflared-Daemon installieren:
Folgen Sie der offiziellen Anleitung für Ihr Betriebssystem. Für Debian/Ubuntu:
Bash

# Cloudflare GPG-Schlüssel hinzufügen
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null
# Cloudflare APT-Repository hinzufügen
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
# Paketlisten aktualisieren und cloudflared installieren
sudo apt update
sudo apt install cloudflared
2. Authentifizierung & Tunnel-Erstellung
[ ] cloudflared authentifizieren:
Dieser Befehl verbindet den Daemon mit Ihrem Cloudflare-Konto.
Bash

cloudflared login
Ein Browserfenster wird geöffnet. Melden Sie sich an und autorisieren Sie die gewünschte Domain.
[ ] Benannten Tunnel erstellen:
Dieser Befehl registriert einen neuen, dauerhaften Tunnel bei Cloudflare.
Bash

cloudflared tunnel create ihr-portfolio-tunnel
WICHTIG: Notieren Sie sich die ausgegebene Tunnel-ID (UUID) und den Pfad zur Zertifikatsdatei (<UUID>.json). Bewahren Sie die Zertifikatsdatei sicher auf.
3. Konfiguration & Routing
[ ] Konfigurationsdatei erstellen:
Erstellen Sie eine Konfigurationsdatei unter /etc/cloudflared/config.yml. Diese Datei weist den Tunnel an, wie der Verkehr weitergeleitet werden soll.
YAML

# /etc/cloudflared/config.yml
# Die Tunnel-UUID aus dem 'create'-Befehl
tunnel: IHRE-TUNNEL-UUID-HIER
# Pfad zur Zertifikatsdatei
credentials-file: /root/.cloudflared/IHRE-TUNNEL-UUID-HIER.json

# Ingress-Regeln leiten den Verkehr von einem öffentlichen Hostnamen zu einem lokalen Dienst
ingress:
  # Regel 1: Ihre Hauptanwendung
  - hostname: portfolio.ihredomain.de
    service: http://localhost:8080

  # Regel 2: Eine Catch-all-Regel, die für unbekannte Hostnamen einen 404-Fehler zurückgibt
  - service: http_status:404
[ ] DNS-Routing zum Tunnel einrichten:
Dieser Befehl erstellt einen CNAME-Eintrag in Ihren Cloudflare DNS-Einstellungen, der Ihren öffentlichen Hostnamen auf den Tunnel verweist.
Bash

cloudflared tunnel route dns ihr-portfolio-tunnel portfolio.ihredomain.de
4. Service-Einrichtung & Verifizierung
[ ] cloudflared als Service installieren:
Dies stellt sicher, dass der Tunnel dauerhaft läuft und beim Systemstart automatisch gestartet wird.
Bash

sudo cloudflared service install
[ ] Service starten und aktivieren:
Bash

sudo systemctl start cloudflared
sudo systemctl enable cloudflared
[ ] Tunnel-Status überprüfen:
Vergewissern Sie sich, dass der Tunnel aktiv und mit dem Cloudflare-Edge verbunden ist.
Bash

cloudflared tunnel list
Ihr Tunnel sollte mit dem Status HEALTHY aufgeführt sein. Sie können den Dienst auch mit sudo systemctl status cloudflared überprüfen.
[ ] Abschließender Test:
Öffnen Sie https://portfolio.ihredomain.de in einem Browser. Ihre lokale Website sollte nun öffentlich erreichbar sein.
5. Sicherheit & Wartung
[ ] Cloudflare WAF aktivieren:
Es wird dringend empfohlen, die Web Application Firewall (WAF) von Cloudflare für Ihre Domain zu aktivieren und zu konfigurieren, um sich vor gängigen Bedrohungen zu schützen.
[ ] Server-Firewall (z. B. UFW) überprüfen:
Stellen Sie sicher, dass Ihre lokale Firewall aktiv ist und so konfiguriert ist, dass sie standardmäßig allen eingehenden Verkehr blockiert. Für den Tunnel sind keine offenen Eingangsports auf Ihrem Server erforderlich.
[ ] cloudflared aktuell halten:
Führen Sie regelmäßig Systemupdates durch, um sicherzustellen, dass Sie die neueste Version von cloudflared mit allen Sicherheitspatches verwenden.
[ ] (Optional) Cloudflare Access verwenden:
Für private Anwendungen können Sie Cloudflare Access-Richtlinien über den Tunnel legen, um eine Benutzerauthentifizierung zu erzwingen, bevor Ihr Dienst erreicht werden kann.