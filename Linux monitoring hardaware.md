---
tags:
  - linux
---
La surveillance hardware est critique pour anticiper les pannes. Ce guide détaille comment monitorer températures, ventilateurs, santé des disques (SMART) et utiliser IPMI/iDRAC pour un contrôle out-of-band complet.

#### Plan

- Pourquoi monitorer le hardware ?
- lm-sensors : températures et ventilateurs
- SMART monitoring des disques
- IPMI et contrôleurs BMC
- iDRAC/iLO pour serveurs Dell/HP
- Alertes automatiques et intégration [Prometheus + Grafana](https://www.shpv.fr/blog/deployer-stack-monitoring/)
- Troubleshooting pannes hardware (voir [diagnostics matériel](https://www.shpv.fr/blog/hardware-diagnostics/))
- Conclusion

---

#### Pourquoi monitorer le hardware ?

##### Pannes évitables avec monitoring proactif

**Signaux d'alerte typiques :**

- Température CPU >80°C → Ventilateur défaillant
- SMART reallocated sectors >0 → Disque en fin de vie
- Ventilateur en dessous des 1000 RPM → Panne imminente
- PSU voltage hors plage → Alimentation défectueuse

**Coûts d'une panne non anticipée :**

- Downtime : 1-24h
- Perte de données (disque)
- Intervention urgente : 2-5x prix normal
- Impact business : 5000-50000 €/h selon activité

**Bénéfices monitoring :**

- Détection précoce : remplacement planifié
- Downtime réduit : -90 %
- Coûts maintenances : -60 %
- Durée de vie hardware : +30 %

---

#### lm-sensors : températures et ventilateurs

##### Installation et configuration

```bash
# Installer lm-sensors
apt install lm-sensors  # Debian/Ubuntu
dnf install lm_sensors   # RHEL/Rocky

# Détecter capteurs disponibles
sensors-detect
# Répondre YES à toutes les questions
# Les modules kernel sont chargés automatiquement

# Vérifier capteurs détectés
sensors
```

**Sortie typique :**

```
coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +45.0°C  (high = +80.0°C, crit = +100.0°C)
Core 0:        +43.0°C  (high = +80.0°C, crit = +100.0°C)
Core 1:        +44.0°C  (high = +80.0°C, crit = +100.0°C)
Core 2:        +45.0°C  (high = +80.0°C, crit = +100.0°C)
Core 3:        +46.0°C  (high = +80.0°C, crit = +100.0°C)

it8728-isa-0a30
Adapter: ISA adapter
in0:          +1.82 V  (min =  +0.00 V, max =  +3.06 V)
in1:          +2.02 V  (min =  +0.00 V, max =  +3.06 V)
in2:          +2.02 V  (min =  +0.00 V, max =  +3.06 V)
fan1:        1245 RPM  (min =    0 RPM)
fan2:        1189 RPM  (min =    0 RPM)
fan3:           0 RPM  (min =    0 RPM)
temp1:        +35.0°C  (low  = +127.0°C, high = +127.0°C)
temp2:        +38.0°C  (low  = +127.0°C, high = +127.0°C)
```

##### Configuration personnalisée

```bash
# Éditer /etc/sensors3.conf (ou /etc/sensors.d/custom.conf)

chip "coretemp-isa-0000"
    label temp1 "CPU Package"
    set temp1_max 80
    set temp1_crit 95

chip "it8728-isa-*"
    label fan1 "CPU Fan"
    label fan2 "Chassis Fan"
    set fan1_min 800
    set fan2_min 600

    label temp1 "Motherboard"
    label temp2 "Chipset"
    set temp1_max 60
    set temp2_max 70
```

```bash
# Recharger config
sensors -s

# Vérifier avec labels custom
sensors
```

##### Monitoring continu

```bash
# Watch en temps réel
watch -n 2 sensors

# Logs continus
while true; do
    date >> /var/log/sensors.log
    sensors >> /var/log/sensors.log
    sleep 300  # Toutes les 5 minutes
done &
```

##### Script d'alerte températures

```bash
#!/bin/bash
# /usr/local/bin/check-temps.sh

EMAIL="admin@example.com"
CPU_TEMP_MAX=80
MB_TEMP_MAX=60

# Extraire température CPU
CPU_TEMP=$(sensors | grep "Package id 0" | awk '{print $4}' | sed 's/+//;s/°C//')

# Extraire température MB
MB_TEMP=$(sensors | grep "Motherboard" | awk '{print $2}' | sed 's/+//;s/°C//')

# Vérifier CPU
if (( $(echo "$CPU_TEMP > $CPU_TEMP_MAX" | bc -l) )); then
    mail -s "⚠️ HIGH CPU Temperature: ${CPU_TEMP}°C" $EMAIL << EOF
WARNING: CPU temperature is ${CPU_TEMP}°C (threshold: ${CPU_TEMP_MAX}°C)

Current sensors output:
$(sensors)

Action required: Check CPU cooler and thermal paste.
EOF
fi

# Vérifier MB
if (( $(echo "$MB_TEMP > $MB_TEMP_MAX" | bc -l) )); then
    mail -s "⚠️ HIGH Motherboard Temperature: ${MB_TEMP}°C" $EMAIL << EOF
WARNING: Motherboard temperature is ${MB_TEMP}°C (threshold: ${MB_TEMP_MAX}°C)

Current sensors output:
$(sensors)

Action required: Check chassis ventilation.
EOF
fi

# Vérifier ventilateurs
FAN_ISSUE=$(sensors | grep "fan" | awk '{if ($2 == "0" || $2 < 500) print $0}')

if [ ! -z "$FAN_ISSUE" ]; then
    mail -s "🚨 CRITICAL: Fan failure detected" $EMAIL << EOF
CRITICAL: One or more fans are not spinning or spinning too slow.

$FAN_ISSUE

IMMEDIATE ACTION REQUIRED: Check and replace faulty fans.
EOF
fi
```

```bash
# Installer
chmod +x /usr/local/bin/check-temps.sh

# Cron toutes les 10 minutes
echo "*/10 * * * * /usr/local/bin/check-temps.sh" | crontab -
```

---

#### SMART monitoring des disques

##### Installation smartmontools

```bash
# Installer
apt install smartmontools

# Activer service
systemctl enable --now smartd

# Vérifier
smartctl --version
```

##### Vérifier santé disque

```bash
# Status général
smartctl -H /dev/sda
# SMART overall-health self-assessment test result: PASSED

# Informations complètes
smartctl -a /dev/sda

# Attributs critiques
smartctl -A /dev/sda | grep -E "Reallocated|Current_Pending|Offline_Uncorrectable|Temperature"
```

**Attributs SMART critiques :**

|ID|Attribut|Signification|Seuil alerte|
|---|---|---|---|
|5|Reallocated_Sector_Ct|Secteurs réalloués|>0|
|187|Reported_Uncorrect|Erreurs non corrigées|>0|
|188|Command_Timeout|Timeouts commandes|>0|
|197|Current_Pending_Sector|Secteurs en attente réallocation|>0|
|198|Offline_Uncorrectable|Secteurs non corrigibles|>0|
|194|Temperature_Celsius|Température|>50°C|

##### Tests SMART

```bash
# Test court (2 minutes)
smartctl -t short /dev/sda

# Test long (plusieurs heures)
smartctl -t long /dev/sda

# Voir résultat
smartctl -l selftest /dev/sda
```

##### Configuration smartd

```bash
# /etc/smartd.conf

# Monitorer tous les disques SATA
DEVICESCAN -a -o on -S on -s (S/../.././02|L/../../6/03) -m admin@example.com -M exec /usr/share/smartmontools/smartd-runner

# Explication :
# -a : activer tous les attributs
# -o on : activer offline tests
# -S on : activer autosave
# -s : schedule tests (Short daily 2h, Long Saturday 3h)
# -m : email alertes
# -M exec : script pour alertes

# Ou configuration par disque
/dev/sda -a -d sat -o on -S on -s (S/../.././02|L/../../6/03) -m admin@example.com -M test
/dev/sdb -a -d sat -o on -S on -s (S/../.././02|L/../../6/03) -m admin@example.com
```

```bash
# Tester config
smartd -q onecheck

# Redémarrer service
systemctl restart smartd
```

##### Script monitoring SMART custom

```bash
#!/bin/bash
# /usr/local/bin/check-smart.sh

EMAIL="admin@example.com"

for disk in $(lsblk -d -n -o NAME | grep -E "sd|nvme"); do
    DEVICE="/dev/$disk"

    # Vérifier santé globale
    HEALTH=$(smartctl -H $DEVICE | grep "result:" | awk '{print $NF}')

    if [ "$HEALTH" != "PASSED" ]; then
        mail -s "🚨 SMART FAILURE: $DEVICE" $EMAIL << EOF
CRITICAL: SMART health check FAILED for $DEVICE

Full SMART output:
$(smartctl -a $DEVICE)

IMMEDIATE ACTION REQUIRED: Backup data and replace disk.
EOF
    fi

    # Vérifier attributs critiques
    REALLOC=$(smartctl -A $DEVICE | grep "Reallocated_Sector" | awk '{print $10}')
    PENDING=$(smartctl -A $DEVICE | grep "Current_Pending_Sector" | awk '{print $10}')
    UNCORRECT=$(smartctl -A $DEVICE | grep "Offline_Uncorrectable" | awk '{print $10}')

    if [ ! -z "$REALLOC" ] && [ "$REALLOC" -gt 0 ]; then
        mail -s "⚠️ WARNING: Reallocated sectors on $DEVICE" $EMAIL << EOF
WARNING: $DEVICE has $REALLOC reallocated sectors.

This disk is showing signs of failure. Plan replacement soon.

$(smartctl -A $DEVICE | grep -E "Reallocated|Pending|Uncorrect")
EOF
    fi

    # Vérifier température
    TEMP=$(smartctl -A $DEVICE | grep "Temperature_Celsius" | awk '{print $10}')

    if [ ! -z "$TEMP" ] && [ "$TEMP" -gt 50 ]; then
        mail -s "⚠️ HIGH Disk Temperature: $DEVICE ${TEMP}°C" $EMAIL << EOF
WARNING: $DEVICE temperature is ${TEMP}°C (threshold: 50°C)

Check cooling and disk placement.
EOF
    fi
done
```

```bash
# Cron quotidien
echo "0 2 * * * /usr/local/bin/check-smart.sh" | crontab -
```

---

#### IPMI et contrôleurs BMC

##### Qu'est-ce que IPMI ?

**IPMI (Intelligent Platform Management Interface) :**

- Contrôle out-of-band (indépendant de l'OS)
- Accessible même si serveur éteint
- Monitoring hardware complet
- KVM over IP
- Power management

##### Installation ipmitool

```bash
# Installer
apt install ipmitool

# Charger module kernel
modprobe ipmi_devintf
modprobe ipmi_si

# Vérifier détection
dmidecode -t 38  # Info IPMI
```

##### Commandes IPMI essentielles

```bash
# Informations BMC
ipmitool bmc info

# Sensors (températures, voltages, ventilateurs)
ipmitool sensor list

# SDR (Sensor Data Records)
ipmitool sdr list

# FRU (Field Replaceable Units) info
ipmitool fru list

# Event log (historique hardware)
ipmitool sel list

# Clear event log
ipmitool sel clear
```

##### Monitoring via IPMI

```bash
# Températures CPU
ipmitool sensor get "CPU Temp" "CPU1 Temp" "CPU2 Temp"

# Ventilateurs
ipmitool sensor get "FAN1" "FAN2" "FAN3"

# Voltages PSU
ipmitool sensor get "12V" "5V" "3.3V"

# Power consumption
ipmitool dcmi power reading
```

##### Configuration réseau IPMI

```bash
# Voir config actuelle
ipmitool lan print 1

# Configurer IP statique
ipmitool lan set 1 ipsrc static
ipmitool lan set 1 ipaddr 192.168.1.100
ipmitool lan set 1 netmask 255.255.255.0
ipmitool lan set 1 defgw ipaddr 192.168.1.1

# Configurer user/password
ipmitool user set name 2 admin
ipmitool user set password 2 StrongPassword123
ipmitool user enable 2
ipmitool channel setaccess 1 2 link=on ipmi=on callin=on privilege=4
```

##### Accès IPMI distant

```bash
# Depuis un autre serveur
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sensor list

# Power control
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password power status
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password power on
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password power reset

# SOL (Serial Over LAN) - console texte
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sol activate
```

---

#### iDRAC/iLO pour serveurs Dell/HP

##### Dell iDRAC (Integrated Dell Remote Access Controller)

**Accès web :**

```
https://idrac-ip
Username: root
Password: calvin (défaut, à changer !)
```

**Configuration CLI (racadm) :**

```bash
# Installer racadm
wget https://linux.dell.com/repo/community/openmanage/...
dpkg -i srvadmin-racadm.deb

# Configurer réseau iDRAC
racadm config -g cfgLanNetworking -o cfgNicIpAddress 192.168.1.101
racadm config -g cfgLanNetworking -o cfgNicNetmask 255.255.255.0
racadm config -g cfgLanNetworking -o cfgNicGateway 192.168.1.1

# Changer password root
racadm set iDRAC.Users.2.Password "NewStrongPassword"

# Voir logs hardware
racadm getsel

# Clear logs
racadm clrsel
```

**Fonctionnalités iDRAC :**

- Virtual Console (KVM over IP)
- Virtual Media (monter ISO distant)
- BIOS/firmware updates
- Hardware inventory
- Alertes email/SNMP

##### HP iLO (Integrated Lights-Out)

**Accès web :**

```
https://ilo-ip
Username: Administrator
Password: (sur étiquette serveur)
```

**Configuration CLI (hponcfg) :**

```bash
# Installer
apt install hp-health hponcfg

# XML config file
cat > ilo-config.xml << 'EOF'
<RIBCL VERSION="2.0">
  <LOGIN USER_LOGIN="Administrator" PASSWORD="password">
    <RIB_INFO MODE="write">
      <MOD_NETWORK_SETTINGS>
        <IP_ADDRESS VALUE="192.168.1.102"/>
        <SUBNET_MASK VALUE="255.255.255.0"/>
        <GATEWAY_IP_ADDRESS VALUE="192.168.1.1"/>
      </MOD_NETWORK_SETTINGS>
    </RIB_INFO>
  </LOGIN>
</RIBCL>
EOF

hponcfg -f ilo-config.xml
```

---

#### Alertes automatiques et intégration Prometheus

##### Prometheus node_exporter avec hwmon

```bash
# Installer node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xzf node_exporter-1.7.0.linux-amd64.tar.gz

# Lancer avec collectors hardware
./node_exporter \
  --collector.hwmon \
  --collector.thermal_zone \
  --collector.systemd

# Métriques disponibles
curl localhost:9100/metrics | grep -E "hwmon|thermal"
```

##### IPMI exporter pour Prometheus

```bash
# Installer ipmi_exporter
docker run -d \
  --name ipmi-exporter \
  -p 9290:9290 \
  --device=/dev/ipmi0 \
  prometheuscommunity/ipmi-exporter

# prometheus.yml
scrape_configs:
  - job_name: 'ipmi'
    static_configs:
      - targets: ['localhost:9290']
```

##### Alertes Prometheus

```yaml
# /etc/prometheus/rules/hardware.yml
groups:
  - name: hardware
    rules:
      - alert: HighCPUTemperature
        expr: node_hwmon_temp_celsius{chip="coretemp"} > 80
        for: 5m
        annotations:
          summary: 'High CPU temperature on {{ $labels.instance }}'
          description: 'CPU temperature is {{ $value }}°C'

      - alert: FanFailure
        expr: node_hwmon_fan_rpm < 800
        for: 2m
        annotations:
          summary: 'Fan failure on {{ $labels.instance }}'
          description: 'Fan {{ $labels.sensor }} is at {{ $value }} RPM'

      - alert: DiskSMARTFailure
        expr: smartmon_device_smart_healthy == 0
        for: 1m
        annotations:
          summary: 'SMART failure on {{ $labels.disk }}'
          description: 'Disk {{ $labels.disk }} SMART health check failed'
```

##### Grafana dashboard

```json
{
	"panels": [
		{
			"title": "CPU Temperature",
			"targets": [
				{
					"expr": "node_hwmon_temp_celsius{chip=\"coretemp\"}",
					"legendFormat": "{{ sensor }}"
				}
			]
		},
		{
			"title": "Fan Speed",
			"targets": [
				{
					"expr": "node_hwmon_fan_rpm",
					"legendFormat": "{{ sensor }}"
				}
			]
		},
		{
			"title": "Disk Temperature",
			"targets": [
				{
					"expr": "smartmon_temperature_celsius_raw_value",
					"legendFormat": "{{ disk }}"
				}
			]
		}
	]
}
```

---

#### Troubleshooting pannes hardware

##### Diagnostiquer surchauffe CPU

```bash
# 1. Vérifier température actuelle
sensors | grep "Package id"

# 2. Vérifier throttling CPU
dmesg | grep -i "throttl"
# CPU0: Package temperature above threshold, cpu clock throttled

# 3. Vérifier fréquence CPU (si throttling)
cat /proc/cpuinfo | grep MHz

# 4. Vérifier ventilateur CPU
sensors | grep -i "cpu fan"

# 5. Stress test pour confirmer
apt install stress
stress --cpu 8 --timeout 60s
watch sensors  # Observer température
```

**Solutions :**

- Nettoyer poussière radiateur/ventilateur
- Remplacer pâte thermique
- Vérifier ventilateur fonctionne (>1000 RPM)
- Améliorer flux d'air chassis

##### Diagnostiquer disque défaillant

```bash
# 1. SMART health check
smartctl -H /dev/sda

# 2. Vérifier attributs critiques
smartctl -A /dev/sda | grep -E "Reallocated|Pending|Uncorrect"

# 3. Vérifier logs kernel
dmesg | grep -i "error" | grep sda

# 4. Test lecture complète
badblocks -nsv /dev/sda

# 5. Test SMART long
smartctl -t long /dev/sda
# Attendre 2-4h puis :
smartctl -l selftest /dev/sda
```

**Signaux de remplacement immédiat :**

- Reallocated sectors >10
- Current pending sectors >0
- Multiple SMART errors
- I/O errors dans dmesg

##### Diagnostiquer RAM défectueuse

```bash
# 1. Vérifier logs erreurs mémoire
dmesg | grep -i "memory\|edac"

# 2. Test memtest (reboot requis)
apt install memtest86+
# Reboot et sélectionner memtest au boot
# Laisser tourner 8+ heures

# 3. MCE (Machine Check Exception) errors
apt install mcelog
mcelog --client

# 4. EDAC (Error Detection And Correction)
modprobe edac_core
cat /sys/devices/system/edac/mc/mc*/csrow*/ch*_ce_count
# >0 = correctable errors
```

---

#### Checklist monitoring hardware

✅ **Températures :**

- [ ]  lm-sensors installé et configuré
- [ ]  Alertes si CPU >80°C
- [ ]  Alertes si MB >60°C
- [ ]  Monitoring Prometheus actif

✅ **Ventilateurs :**

- [ ]  Alertes si RPM inférieur à 800
- [ ]  Vérification mensuelle physique
- [ ]  Nettoyage poussière trimestriel

✅ **Disques :**

- [ ]  smartd activé
- [ ]  Tests SMART hebdomadaires
- [ ]  Alertes reallocated sectors >0
- [ ]  Température disques inférieure à 50°C

✅ **IPMI/BMC :**

- [ ]  Accès réseau configuré
- [ ]  Password changé (pas défaut)
- [ ]  Alertes email configurées
- [ ]  Tests power control mensuels

✅ **Monitoring centralisé :**

- [ ]  Prometheus + node_exporter
- [ ]  IPMI exporter si disponible
- [ ]  Grafana dashboards
- [ ]  Alertes configurées

---

#### Conclusion

Le monitoring hardware proactif réduit drastiquement les pannes imprévues et leurs coûts. lm-sensors, SMART monitoring et IPMI forment la base d'une surveillance complète.

**Points clés :**

- Surveiller températures CPU/MB/disques
- Monitorer santé disques (SMART)
- Utiliser IPMI pour out-of-band
- Alertes automatiques essentielles
- Tests réguliers pour validation

**Gains typiques :**

- Pannes imprévues : -85 %
- Downtime : -90 %
- Coûts maintenance : -60 %
- Durée de vie hardware : +30 %

**Actions prioritaires :**

1. Installer lm-sensors + smartd
2. Configurer alertes températures/ventilateurs
3. Activer IPMI si disponible
4. Implémenter monitoring Prometheus
5. Documenter seuils et procédures