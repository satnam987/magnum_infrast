# Installatie Uptime Kuma (Lokale Ansible op VM)

Deze handleiding installeert Uptime Kuma door Ansible direct op de doel-VM uit te voeren.

## ⚠️ BELANGRIJKE VOORWAARDEN

1. **Rocky Linux 9 VM** met een gebruiker die `sudo` rechten heeft
2. **Internetverbinding** op de VM
3. **Direct toegang tot de VM console** (niet via SSH - dat wordt later geconfigureerd)

---

## STAP 1: Basis Setup (op de VM console)

**🔴 VOER DEZE STAPPEN UIT OP DE VM ZELF (niet via SSH)**

### 1.1 Systeem Updaten

```bash
sudo dnf update -y
```

### 1.2 Git Installeren en Project Klonen

```bash
# Git installeren
sudo dnf install -y git

# Project klonen
git clone https://github.com/satnam987/magnum_infrast.git
cd magnum_infrast
```

### 1.3 SSH Server Installeren en Configureren

**⚠️ KRITISCH: Zonder deze stap kun je niet via SSH verbinden!**

```bash
# SSH server installeren
sudo dnf install -y openssh-server

# SSH service starten en inschakelen
sudo systemctl start sshd
sudo systemctl enable sshd

# Firewall configureren voor SSH
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

# Controleren of SSH correct draait
sudo systemctl status sshd
sudo ss -tlnp | grep :22
```

**✅ Test SSH vanaf je Windows PC:**

```cmd
ssh gebruikersnaam@IP_ADRES_VM
```

### 1.4 Ansible Installeren

```bash
# EPEL repository installeren
sudo dnf install -y epel-release

# Ansible en Python installeren
sudo dnf install -y ansible python3-pip

# Controleren of Ansible geïnstalleerd is
ansible --version
```

---

## STAP 2: Configuratie Aanpassen (optioneel)

**📍 Je bent nu in de map: `~/magnum_infrast`**

Pas indien gewenst de instellingen aan in `inventory/group_vars/all.yml`:

### Belangrijke configuratie-opties:

- `hostname`: Gewenste hostnaam (standaard: `localhost`)
- `timezone`: Tijdzone (standaard: `Europe/Brussels`)
- `uptime_kuma_port`: Poort voor Uptime Kuma (standaard: `3001`)
- `enable_ssl`: SSL inschakelen (standaard: `false`)
- `fail2ban_enable`: Fail2Ban beveiliging (standaard: `true`)

### Voor SSH toegang configuratie:

- `root_ssh_public_key`: Voeg je publieke SSH-sleutel toe voor veilige root toegang

---

## STAP 3: Installatie Uitvoeren

**📍 Zorg ervoor dat je in de `magnum_infrast` map bent:**

```bash
# Controleer huidige map
pwd
# Moet zijn: /home/gebruikersnaam/magnum_infrast

# Als je niet in de juiste map bent:
cd ~/magnum_infrast
```

**🚀 Playbook uitvoeren:**

```bash
ansible-playbook -i inventory/hosts.yml site.yml -K
```

**⚠️ Belangrijke opmerkingen:**

- Ansible vraagt om het `BECOME password` - dit is je **sudo wachtwoord**
- De installatie kan 5-15 minuten duren
- **Laat de installatie volledig aflopen** - onderbreek niet!

---

## STAP 4: Verificatie

### 4.1 Controleer of services draaien

```bash
# Docker status
sudo systemctl status docker

# Uptime Kuma container
sudo docker ps -a
sudo docker logs uptime-kuma
```

### 4.2 Test toegang tot Uptime Kuma

**🌐 Vanaf je Windows PC (aanbevolen):**

```
http://IP_ADRES_VAN_VM:3001
```

**Bijvoorbeeld:** `http://172.20.10.15:3001`

**💻 Vanaf de VM zelf:**

```
http://IP_ADRES_VAN_VM:3001
```

**❌ NIET GEBRUIKEN:** `http://localhost:3001` (kan verbindingsproblemen geven)

---

## STAP 5: Eerste Setup Uptime Kuma

1. **Open de web interface** in je browser
2. **Maak een admin account aan** bij eerste toegang
3. **Configureer je monitoring doelen**

---

## 🔧 Probleemoplossing

### SSH verbinding lukt niet

```bash
# Op de VM: controleer SSH service
sudo systemctl status sshd
sudo systemctl restart sshd

# Controleer firewall
sudo firewall-cmd --list-services | grep ssh
```

### Ansible commando niet gevonden

```bash
# Installeer Ansible opnieuw
sudo dnf install -y epel-release ansible
ansible --version
```

### Uptime Kuma niet bereikbaar

```bash
# Controleer Docker
sudo systemctl status docker
sudo docker ps -a

# Controleer Uptime Kuma logs
sudo docker logs uptime-kuma

# Controleer firewall poort
sudo firewall-cmd --list-ports
```

### IP-adres van VM vinden

```bash
# Op de VM
ip addr show
# Of
hostname -I
```

---
