# Installatie Uptime Kuma (Lokale Ansible op VM)

Deze handleiding installeert Uptime Kuma door Ansible direct op de doel-VM uit te voeren.

## Voorbereiding (op de VM)

1.  **Rocky Linux 9 VM:** Zorg voor een VM met `sudo` gebruiker en internet.
2.  **Project Klonen:**
    ```bash
    sudo dnf install -y git
    git clone https://github.com/satnam987/magnum_infrast.git
    cd magnum_infrast
    ```
3.  **Ansible Installeren:**
    ```bash
    sudo dnf update -y && sudo dnf install -y epel-release ansible python3-pip
    ```
4.  **(Optioneel) Variabelen Aanpassen:**
    Controleer/bewerk `inventory/group_vars/all.yml` voor instellingen zoals `hostname`, `uptime_kuma_port`, etc. De standaardinstellingen in `inventory/hosts.yml` zijn voor een lokale test.

## Configuratie

Voordat je de playbook uitvoert, configureer je de gewenste instellingen in `inventory/group_vars/all.yml`.

Belangrijke configuratie-opties:

- `hostname`: De gewenste hostnaam voor de server (standaard: `localhost`).
- `timezone`: De tijdzone voor de server (standaard: `Europe/Brussels`).
- `root_ssh_public_key`: **BELANGRIJK VOOR ROOT TOEGANG.** Vul hier je publieke SSH-sleutel in om root-toegang via SSH met een sleutel mogelijk te maken. Als deze sleutel is ingesteld, zal de playbook proberen wachtwoord-authenticatie voor root uit te schakelen.
  - Genereer een SSH keypair (indien je er nog geen hebt) met `ssh-keygen -t rsa -b 4096` op je lokale machine.
  - Kopieer de inhoud van je publieke sleutel (meestal `~/.ssh/id_rsa.pub`) naar deze variabele.
  - Voorbeeld: `ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC... user@host`
  - Laat de placeholder-waarde staan als je deze functionaliteit niet wilt gebruiken (root login blijft dan standaard, mogelijk met wachtwoord als elders geconfigureerd).
- `uptime_kuma_port`: De poort waarop Uptime Kuma extern bereikbaar zal zijn (standaard: `3001`).
- `enable_ssl`: Zet op `true` om SSL-configuratie met Nginx en een poging tot Let's Encrypt certificaat te activeren (standaard: `false`).
  - **Let op:** Voor een succesvolle Let's Encrypt validatie moet de VM publiek bereikbaar zijn op poort 80 en 443 via een domeinnaam die naar het publieke IP-adres van de VM wijst. Als dit niet het geval is (bv. bij een lokale VM zonder publiek IP/DNS), zal Nginx wel geconfigureerd worden voor SSL, maar zal Certbot **falen** om een certificaat te verkrijgen. Uptime Kuma zal dan mogelijk niet direct bereikbaar zijn via HTTPS.
  - In een dergelijk scenario (lokale test zonder publieke toegang), zal Nginx proberen te starten met een verwijzing naar niet-bestaande SSL-certificaten, wat kan leiden tot fouten bij het starten van Nginx of een niet-functionerende webserver op poort 443. Voor testdoeleinden zou je dan `enable_ssl` op `false` moeten laten of handmatig self-signed certificaten moeten genereren en de Nginx-configuratie daarop aanpassen (dit laatste valt buiten de standaard scope van deze playbook m.b.t. Let's Encrypt).
  - De playbook bevat logica om Certbot een certificaat te laten aanvragen. Als dit faalt, wordt er geen fallback naar self-signed certificaten gedaan.
- `ssl_email`: Je e-mailadres voor Let's Encrypt registratie en meldingen (vereist als `enable_ssl: true`).
- `ssl_domain_name`: De domeinnaam waarvoor het SSL-certificaat aangevraagd moet worden (vereist als `enable_ssl: true`).
- `fail2ban_enable`: Zet op `true` om Fail2Ban te installeren en configureren voor SSH-beveiliging (standaard: `true`).

## Installatie Uitvoeren (op de VM)

Navigeer naar de projectmap (bv. `~/magnum_infra`) en voer uit:

```bash
ansible-playbook -i inventory/hosts.yml site.yml -K
```

- **Wachtwoord:** Ansible zal vragen om het `BECOME password`. Voer hier het `sudo` wachtwoord in van de gebruiker waarmee je bent ingelogd.

Na de uitvoering is Uptime Kuma beschikbaar.

⚠️ **BELANGRIJK: Toegang tot Uptime Kuma** ⚠️

- **Vanaf een andere machine in hetzelfde netwerk (bv. je host PC):**
  Gebruik `http://IP_ADRES_VAN_VM:POORT` (vervang `IP_ADRES_VAN_VM` met het daadwerkelijke IP-adres van je VM, bv. `http://192.168.1.9:3001`).

- **Vanaf de VM zelf (in de browser op de VM):**
  - Gebruik bij voorkeur ook `http://IP_ADRES_VAN_VM:POORT` (bv. `http://192.168.1.9:3001`).
  - `http://localhost:3001` kan soms verbindingsproblemen geven in de browser, ook al draait de service correct. Als `localhost` niet werkt, gebruik dan altijd het IP-adres.

## Problemen?

- Check Ansible output voor fouten.
- Controleer Docker status: `sudo systemctl status docker`, `sudo docker ps -a`.
- Uptime Kuma logs: `sudo docker logs uptime-kuma` (of de container naam uit `all.yml`).
