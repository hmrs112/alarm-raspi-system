
# Alarm-RasPI-System (ARS)


**WICHTIG** 

Dieses Projekt ist noch nicht vollständig fertiggestellt! Es fehlt noch ein entscheidender Teil der Alarmdruckfunktionalität und es ist auch noch eine Adminoberfläche geplant.

---

## Vorwort
Ansible Playbook für das Alarm-RasPI-System (ARS). Das ARS ist zur Alarmsignalisierung und Verteilung von Informationen im Alarmfall. 
Dazu kann ein Alarmmonitor (z.B. Divera247) über den HDMI-Port angezeigt werden. Außerdem können ein oder mehrere Netzwerkdrucker als Alarmdrucker angeschlossen werden. 

### Technologiestack
- Ansible - Zum automatisierten Installieren von einem oder mehreren Raspberry Pi
- Docker - Betrieb eines Reverse-Proxy und CUPS-Server (Admin-Oberfläche geplant)
- CUPS - Open-Source Drucksystem mit Weboberfläche
- Traefik - Reverse Proxy, leitet die unterschiedlichen Hostnamen auf die HTTP-Endpunkte
- Chromium - Browser zum Anzeigen der Alarmmonitore

### Vorraussetzungen
1. RaspberryPI 4 (getestet mit der 2GB Variante)
1. RaspianOS mit Desktopumgebung (getestet mit 2021-03-04)
1. Aktivierung des SSH-Servers auf RaspianOS (raspi-config)
1. Kopieren eines SSH-Public-Key auf den ´pi´ Benutzer (ssh-keygen, ssh-copy-id)
1. Mehrere DNS-Einträge mit Auflösung auf IP des Raspberry PI (cups..., etc.)
1. Lokale Ansible-Installation (ansible-playbook)

## Installation
1. Playbook herunterladen (z.B. git clone ...)
1. Konfiguration und Inventory anpassen (s. Konfiguration und Inventory)
1. Ausführen und Installieren des Ansible-Playbooks (s. Ausführen und Installieren)


### Konfiguration und Inventory
#### Inventory
  ```yaml
  all:
  vars:
    ansible_port: 22
    ansible_user: pi
    ansible_python_interpreter: /usr/bin/python3
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  hosts:
    raspi-hostname01:
      ansible_host: 192.168.100.101
      chrome_url: "https://www.divera247.de/monitor/1.html?autologin=ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
      traefik_hostname: traefik.lz01.meine-feuerwehr.tld
      admin_hostname: admin.lz01.meine-feuerwehr.tld
      cups_hostname: cups.lz01.meine-feuerwehr.tld
  ```

#### Grundsystem
Der `admin_username` und das `admin_password` wird aktuell noch nicht verwendet. Die angebenden Ports `allowed_ports` werden in der Firewall freigegeben. Als Firewall wird UFW verwendet.

```yaml
admin_username: superadmin
admin_password: supersicherespasswort
allowed_ports:
- 80
- 443
- 22
```

#### Alarmanzeige
Mit `display_output` ist beim Raspberry Pi einer der beiden HDMI-Anschlüsse gemeint. Die Display-Rotation kann mit `display_rotate` eingestellt werden. Der `viewer_user` ist der Benutzer, der beim Login den Brower anzeigt und über HDMI ausgibt.

```yaml
display_output: HDMI-1 # HDMI-1, HDMI-2
display_rotate: normal # normal, right, left, inverted
viewer_user: display-viewer
```

#### Alarmdrucker
Die Zugangsdaten für den CUPS-Admin sind der Benutzer *admin* und das über den Parameter `cups_admin_password` festgelegte Passwort. Der Wert von `cups_image` sollte nur verändert werden, wenn man weiß, was man macht!
```yaml
cups_admin_password: supersicherespasswort
cups_image: hmrs112/cupsd:0.1.2
```

### Ausführen und Installieren
```bash
# Komplett ausführen
ansible-playbook -i inventory.yml playbook.yml

# Nur Webseite des Displays aktualisieren
ansible-playbook -i inventory.yml playbook.yml --tags display_update
```