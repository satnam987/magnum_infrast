# Uptime Kuma met Ansible: Jouw Automatische Monitoringserver

Dit project helpt je om Uptime Kuma makkelijk op een server te zetten. Uptime Kuma is een gratis programma dat websites en servers in de gaten houdt. Dit project gebruikt Ansible om alles automatisch te installeren en in te stellen.

Dit project is gemaakt voor de Magnum Opus opdracht "Infrastructure-as-code" van AP Hogeschool.

## Wat kan dit project?

- **Uptime Kuma installeren**: Zet Uptime Kuma op met Docker. Docker helpt om programma's netjes apart te houden.
- **Server instellen**: Kiest een naam (hostname) en tijdzone voor je server.
- **Firewall (brandmuur)**: Stelt een firewall in. Deze laat alleen de nodige verbindingen toe.
- **SSL-Certificaten (slotje)**: Kan zorgen voor een veilige HTTPS-verbinding met Let's Encrypt. Dit is optioneel.
- **Extra Beveiliging**: Kan Fail2Ban installeren. Dit programma blokkeert hackers die proberen in te loggen.
- **Veilige SSH-toegang**: Stelt in dat je veilig kunt inloggen op de server met een speciale sleutel (SSH key).

## Wat heb je nodig?

Voor je begint, zorg dat je dit hebt:

- Een server met **Rocky Linux 9**. Dit moet een kale, minimale installatie zijn.
- Deze server moet een **IP-adres** hebben en verbonden zijn met het internet.
- Je hebt een **SSH-sleutelpaar** nodig om veilig in te loggen (vooral voor de `root`-gebruiker).
- Minimale serverkracht:
  - 2GB RAM (geheugen)
  - 2 CPU-kernen (processorkracht)
  - 20GB vrije schijfruimte

## Hoe werkt het?

De installatie gebeurt met Ansible. Je past een paar instellingen aan, en Ansible doet de rest.

### 1. Instellingen aanpassen

Alle instellingen staan in één bestand: `inventory/group_vars/all.yml`. Open dit bestand en verander wat nodig is.
De belangrijkste instellingen zijn:

```yaml
# Servernaam die je wilt gebruiken
hostname: "jouw-uptime-kuma-server"

# Jouw tijdzone
timezone: "Europe/Brussels"

# Poort waarop Uptime Kuma bereikbaar is (standaard 3001)
uptime_kuma_port: 3001

# Docker image versie voor Uptime Kuma (meestal "latest")
uptime_kuma_version: "latest"

# ----- SSL (HTTPS) Instellingen -----
# Zet op 'true' als je een slotje (HTTPS) wilt met Let's Encrypt
enable_ssl: false

# Jouw e-mailadres (nodig voor Let's Encrypt)
ssl_email: "jouw.email@voorbeeld.com"

# De domeinnaam voor je Uptime Kuma (bv. status.jouwdomein.nl)
# Deze MOET naar het IP-adres van je server wijzen!
ssl_domain: "kuma.jouwdomein.nl"

# ----- Firewall Instellingen -----
# Netwerken die via SSH mogen verbinden (optioneel)
# allowed_ssh_networks:
#   - "192.168.1.0/24" # Voorbeeld: alleen lokaal netwerk
#   - "jouw.thuis.ip/32" # Voorbeeld: alleen jouw IP

# ----- SSH-sleutel voor Root -----
# PLAK HIER JE PUBLIEKE SSH SLEUTEL VOOR ROOT TOEGANG
# Deze vind je meestal in ~/.ssh/id_rsa.pub op je eigen computer.
# Voorbeeld: "ssh-rsa AAAAB3Nz.... jouwnaam@jouwcomputer"
root_ssh_public_key: "VERVANG DIT MET JE ECHTE PUBLIEKE SSH SLEUTEL"
```

**Belangrijk voor SSL (HTTPS):**
Als je `enable_ssl: true` zet, moet je `ssl_domain` correct instellen. Deze domeinnaam moet via DNS verwijzen naar het IP-adres van de server waar je Uptime Kuma op installeert. Anders kan Let's Encrypt geen certificaat maken.

### 2. Installatie starten

Als je de instellingen hebt aangepast:

1.  **Kopieer dit project** naar je server, of zorg dat Ansible vanaf jouw computer de server kan bereiken.
    - Zie `INSTALLATIE.md` voor details over hoe je dit lokaal op de server zelf kunt draaien.
2.  **Start Ansible** met dit commando (vanuit de map van dit project):

    ```bash
    ansible-playbook -i inventory/hosts.yml site.yml
    ```

    Als Ansible vraagt om een `BECOME password`, geef dan het `sudo` wachtwoord van de gebruiker op de server.

### 3. Klaar!

Als Ansible klaar is, kun je Uptime Kuma openen in je browser:

- Zonder SSL: `http://IP_ADRES_SERVER:{{ uptime_kuma_port }}`
- Met SSL: `https://{{ ssl_domain }}`

Vervang `IP_ADRES_SERVER` met het IP-adres van je server, en de `{{ ... }}` met de waarden uit je configuratie.

## Onderdelen van dit Project

- **Uptime Kuma**: De monitoringssoftware zelf.
- **Ansible**: Het programma dat alles automatisch installeert.
- **Docker & Docker Compose**: Programma's om Uptime Kuma in een "container" te draaien.
- **Nginx**: Webserver die helpt als je SSL (HTTPS) gebruikt (optioneel).
- **Let's Encrypt**: Voor gratis SSL-certificaten.
- **Firewalld**: De firewall software op de server.
- **Fail2Ban**: Extra beveiliging tegen inbraakpogingen (optioneel).
- **Systeem Hardening**: Verschillende aanpassingen om de server veiliger te maken.

## Inspiratie en Bronnen (Voorbeeld)

Dit project is gebouwd met kennis uit verschillende hoeken. Hieronder enkele voorbeelden van bronnen die inspirerend of nuttig waren:

- **OFFICIËLE UPTIME KUMA DOCUMENTATIE**: De primaire bron voor alles over Uptime Kuma zelf. [https://github.com/louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)
- **ANSIBE DOCUMENTATIE**: Voor het correct gebruiken van Ansible modules en best practices. [https://docs.ansible.com/](https://docs.ansible.com/)
- **DOCKER DOCUMENTATIE**: Voor het begrijpen en configureren van Docker en Docker Compose. [https://docs.docker.com/](https://docs.docker.com/)
- **LET'S ENCRYPT DOCUMENTATIE**: Voor informatie over het aanvragen en vernieuwen van SSL certificaten. [https://letsencrypt.org/docs/](https://letsencrypt.org/docs/)
- **DIGITALOCEAN COMMUNITY TUTORIALS**: Veel praktische handleidingen over serverbeheer, Ansible, en beveiliging. (Voorbeeld: Zoek naar "Ansible Rocky Linux tutorial DigitalOcean")
- **RED HAT ENTERPRISE LINUX DOCUMENTATIE**: Voor diepgaande informatie over systeemcomponenten zoals Firewalld, SELinux op Rocky Linux (dat hierop gebaseerd is).
- **CIS BENCHMARKS**: Als inspiratie voor beveiligingsmaatregelen en systeem hardening. (Voorbeeld: Zoek naar "CIS benchmarks Linux")
