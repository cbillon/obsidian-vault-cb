Les pannes hardware sont difficiles à diagnostiquer car les symptômes sont variés. Ce guide détaille les outils essentiels pour tester RAM, CPU, disques et identifier les défaillances matérielles via les logs kernel.

#### Plan

- Symptômes de pannes hardware
- Tests RAM : memtest86+ et stress-ng
- Tests CPU : stress-ng et torture
- Tests disques : badblocks et SMART
- Analyse logs kernel : dmesg, MCE, EDAC
- Tests PSU et surchauffe
- Procédures de diagnostic méthodiques (voir [monitoring matériel](https://www.shpv.fr/blog/hardware-monitoring/))
- Conclusion

---

#### Symptômes de pannes hardware

##### Pannes RAM

**Symptômes :**

- Kernel panics aléatoires
- Segmentation faults fréquents
- Applications crash sans raison
- Corruption de données
- Freeze système aléatoire

**Logs typiques :**

```
kernel: MCE: CPU 0: Machine Check Exception
kernel: Memory failure at address 0x7f8b4c000000
kernel: segfault at 7fff12345678 ip 00007f9abc123456
```

##### Pannes CPU

**Symptômes :**

- Calculs incorrects
- Crash sous charge élevée
- Throttling thermique constant
- Kernel panics pendant stress tests
- Instabilité overclocking

**Logs typiques :**

```
kernel: CPU0: Package temperature above threshold
kernel: CPU0: Core temperature above threshold
kernel: mce: [Hardware Error]: Machine check events logged
```

##### Pannes disques

**Symptômes :**

- I/O errors fréquents
- Lenteur soudaine
- Fichiers corrompus
- Bad sectors
- Clics mécaniques (HDD)

**Logs typiques :**

```
kernel: ata1.00: exception Emask 0x0 SAct 0x0 SErr 0x0 action 0x0
kernel: ata1.00: BMDMA stat 0x24
kernel: sd 0:0:0:0: [sda] Sense Key : Medium Error [current]
kernel: end_request: I/O error, dev sda, sector 1234567
```

##### Pannes PSU/alimentation

**Symptômes :**

- Reboots spontanés
- Freeze sous charge élevée
- Sous-voltage détecté
- Composants s'éteignent aléatoirement

**Logs typiques :**

```
kernel: ACPI: Battery voltage too low
kernel: Power supply unit failure detected
```

---

#### Tests RAM : memtest86+ et stress-ng

##### memtest86+ (test exhaustif)

```bash
# Installer
apt install memtest86+

# Mettre à jour GRUB
update-grub

# Reboot et sélectionner memtest86+ au boot
reboot
```

**Test memtest86+ :**

- Laisser tourner minimum 8 heures
- 1 pass complet = ~2-4h selon RAM
- Objectif : 3+ passes sans erreur
- **1 seule erreur = RAM défectueuse**

**Interpréter résultats :**

```
Pass: 1
Test: 1/11 (Address test, walking ones, no cache)
Errors: 0

Pass: 1
Test: 2/11 (Address test, own address)
Errors: 3  ← RAM DÉFECTUEUSE

Test 2: Barrette slot 2, adresses 0x7f8b4c000000-0x7f8b4c003fff
Action: Retirer barrette slot 2, relancer test
```

##### stress-ng (test RAM en production)

```bash
# Installer
apt install stress-ng

# Test RAM uniquement (4GB, 60s)
stress-ng --vm 4 --vm-bytes 1G --timeout 60s --metrics-brief

# Test intensif (80% RAM, 300s)
TOTAL_RAM=$(free -g | awk '/^Mem:/{print $2}')
TEST_RAM=$((TOTAL_RAM * 80 / 100))
stress-ng --vm 8 --vm-bytes ${TEST_RAM}G --timeout 300s --verify
```

**Options importantes :**

- `--vm` : nombre de workers
- `--vm-bytes` : mémoire par worker
- `--verify` : vérifier intégrité données
- `--vm-method` : all/flip/inc/rand/…

```bash
# Test exhaustif avec vérification
stress-ng --vm 4 --vm-bytes 2G --vm-method all --verify --timeout 600s

# Si erreurs détectées
stress-ng: error: [12345] vm instance 2: memory corruption detected
# → RAM défectueuse
```

##### Identifier barrette défectueuse

```bash
# Voir layout RAM
dmidecode --type 17

# Exemple sortie :
# Handle 0x0011, DMI type 17, 40 bytes
# Memory Device
#   Total Width: 64 bits
#   Size: 16 GB
#   Locator: DIMM_A1
#   Bank Locator: BANK 0

# Test par barrette (retirer toutes sauf 1, tester)
# 1. Retirer toutes barrettes sauf slot 1
# 2. memtest86+ 8h
# 3. Si OK, tester slot 2
# 4. Répéter jusqu'à trouver barrette défectueuse
```

---

#### Tests CPU : stress-ng et torture

##### stress-ng (stress multi-composants)

```bash
# Test CPU tous les cores (60s)
stress-ng --cpu $(nproc) --timeout 60s --metrics-brief

# Test intensif avec température monitoring
stress-ng --cpu $(nproc) --timeout 300s --metrics-brief &
watch -n 1 sensors
```

**Si crash pendant stress CPU → CPU défectueux ou refroidissement**

##### mprime / Prime95 (torture test)

```bash
# Télécharger mprime (version Linux de Prime95)
wget https://www.mersenne.org/ftp_root/gimps/p95v3019b20.linux64.tar.gz
tar xzf p95v3019b20.linux64.tar.gz
cd mprime

# Lancer torture test
./mprime -t

# Options :
# 1. Small FFTs (stress CPU, chauffe max)
# 2. In-place large FFTs (stress CPU + RAM)
# 3. Blend (stress CPU + RAM + cache)

# Laisser tourner 24h minimum
# 0 erreur = stable
# Erreurs = CPU instable ou RAM défectueuse
```

##### Tests spécifiques

**Test AVX2/AVX-512 (instructions modernes) :**

```bash
# stress-ng avec AVX
stress-ng --cpu $(nproc) --cpu-method ackermann --timeout 300s

# Si CPU crash avec AVX mais pas sans
# → Instabilité AVX (souvent overclocking trop agressif)
```

**Test multi-threaded :**

```bash
# Compiler stress test custom
cat > cpu_test.c << 'EOF'
#include <stdio.h>
#include <math.h>
#include <pthread.h>

void* worker(void* arg) {
    long id = (long)arg;
    double result = 0.0;
    for (long i = 0; i < 1000000000; i++) {
        result += sqrt(i) * sin(i);
    }
    printf("Thread %ld done: %f\n", id, result);
    return NULL;
}

int main() {
    pthread_t threads[32];
    for (long i = 0; i < 32; i++) {
        pthread_create(&threads[i], NULL, worker, (void*)i);
    }
    for (int i = 0; i < 32; i++) {
        pthread_join(threads[i], NULL);
    }
    return 0;
}
EOF

gcc -o cpu_test cpu_test.c -lm -lpthread
./cpu_test
```

---

#### Tests disques : badblocks et SMART

##### badblocks (test exhaustif)

```bash
# Test lecture seule (non destructif)
badblocks -sv /dev/sda

# Test écriture (DESTRUCTIF, efface données)
badblocks -wsv /dev/sda

# Test avec pattern spécifique
badblocks -wsv -t 0xaa /dev/sda
badblocks -wsv -t 0x55 /dev/sda

# Logs
# Testing with pattern 0xaa: done
# Testing with pattern 0x55: done
# Testing with pattern 0xff: done
# Testing with pattern 0x00: done
# Pass completed, 0 bad blocks found.
```

**Si bad blocks détectés :**

```
# Marquer bad blocks dans filesystem
badblocks -sv /dev/sda > /tmp/badblocks.txt
e2fsck -l /tmp/badblocks.txt /dev/sda1

# Mais ATTENTION : >5 bad blocks = remplacer disque immédiatement
```

##### SMART extended test

```bash
# Test court (2 min)
smartctl -t short /dev/sda

# Test long (2-4h selon taille)
smartctl -t long /dev/sda

# Voir progression
smartctl -a /dev/sda | grep "Self-test execution status"

# Résultats
smartctl -l selftest /dev/sda
# Num  Test_Description    Status                  Remaining  LifeTime(hours)
# 1    Extended offline    Completed without error       00%      1234
```

**Attributs SMART critiques :**

```bash
smartctl -A /dev/sda | grep -E "Reallocated|Pending|Uncorrectable|UDMA_CRC"

# 5   Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always  -  0
# 197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always  -  0
# 198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline -  0

# Si valeur RAW > 0 : disque en fin de vie, remplacer
```

##### Test performance disque

```bash
# Test lecture séquentielle
hdparm -t /dev/sda
# Timing buffered disk reads: 350 MB in  3.01 seconds = 116.28 MB/sec

# Si < 50 MB/s sur HDD SATA : disque défaillant
# Si < 300 MB/s sur SSD SATA : problème

# Test avec dd
dd if=/dev/sda of=/dev/null bs=1M count=1000 status=progress
# 1048576000 bytes (1.0 GB, 1000 MiB) copied, 8.5 s, 123 MB/s
```

---

#### Analyse logs kernel : dmesg, MCE, EDAC

##### dmesg (kernel ring buffer)

```bash
# Voir tous les logs kernel
dmesg | less

# Filtrer erreurs hardware
dmesg | grep -i "error\|fail\|hardware\|mce\|edac"

# Filtrer par composant
dmesg | grep -i "memory"
dmesg | grep -i "cpu"
dmesg | grep -i "ata\|sd"
dmesg | grep -i "pci"

# Logs temps réel
dmesg -w
```

**Exemples erreurs critiques :**

```bash
# Erreur RAM
kernel: EDAC MC0: CE page 0x7f8b4c, offset 0x1000, grain 4096, syndrome 0x0086
kernel: EDAC MC0: CE - no information available: DIMM Label: "DIMM_A1"

# Erreur CPU
kernel: mce: [Hardware Error]: Machine check events logged
kernel: mce: [Hardware Error]: CPU 0: Machine Check: 4 Bank 5: be00000000000108

# Erreur disque
kernel: ata1.00: exception Emask 0x0 SAct 0x7 SErr 0x0 action 0x0
kernel: ata1.00: failed command: READ FPDMA QUEUED

# Erreur PCIe
kernel: pcieport 0000:00:01.0: PCIe Bus Error: severity=Corrected, type=Physical Layer
```

##### MCE (Machine Check Exception)

```bash
# Installer mcelog
apt install mcelog

# Voir erreurs MCE
mcelog --client

# Exemple sortie :
# Hardware event. This is not a software error.
# MCE 0
# CPU 0 BANK 5
# MISC 86000000000086 ADDR 7f8b4c001000
# TIME 1642345678 Wed Jan 16 12:34:38 2026
# MCG status:
# MCi status: Corrected error
# MCi_MISC register valid
# MCi_ADDR register valid
# Memory read error
# Transaction: Memory
# MemCtrl ID = 0, Channel = 1, DIMM = 0

# Logs continus
mcelog --daemon
journalctl -u mcelog -f
```

**Interpréter MCE :**

- Corrected error : erreur corrigée par ECC, surveiller
- Uncorrected error : erreur non corrigée, critique
- Memory read error : problème RAM
- CPU/Cache error : problème CPU

##### EDAC (Error Detection And Correction)

```bash
# Charger modules EDAC
modprobe edac_core
modprobe edac_mce_amd  # AMD
modprobe ie31200_edac  # Intel

# Voir erreurs détectées
cat /sys/devices/system/edac/mc/mc*/ce_count
# 0 = OK, >0 = erreurs corrigées détectées

# Détails par DIMM
grep . /sys/devices/system/edac/mc/mc*/csrow*/ch*_ce_count
# mc0/csrow0/ch0_ce_count:0
# mc0/csrow0/ch1_ce_count:5  ← 5 erreurs corrigées sur channel 1

# Logs EDAC
dmesg | grep EDAC
```

##### journalctl (logs système)

```bash
# Logs hardware depuis boot
journalctl -b | grep -i "hardware\|error\|fail"

# Logs kernel uniquement
journalctl -k

# Logs par priorité (error et plus grave)
journalctl -p err

# Logs temps réel
journalctl -f
```

---

#### Tests PSU et surchauffe

##### Test charge PSU

```bash
# Stress complet système (CPU + RAM + I/O)
stress-ng --cpu $(nproc) --vm 4 --vm-bytes 2G --io 4 --hdd 2 --timeout 600s --metrics-brief

# Monitorer pendant test
watch -n 1 'sensors; uptime; free -h'
```

**Si reboot pendant stress → PSU insuffisant ou défectueux**

##### Monitoring températures

```bash
# Installer lm-sensors
apt install lm-sensors
sensors-detect

# Températures en continu
watch -n 1 sensors

# Script alerte température
cat > /tmp/temp_monitor.sh << 'EOF'
#!/bin/bash
while true; do
    CPU_TEMP=$(sensors | grep "Package id 0" | awk '{print $4}' | sed 's/+//;s/°C//')
    if (( $(echo "$CPU_TEMP > 85" | bc -l) )); then
        echo "CRITICAL: CPU temp ${CPU_TEMP}°C"
        stress-ng --cpu 0 --timeout 1s  # Kill stress test
    fi
    sleep 1
done
EOF

bash /tmp/temp_monitor.sh &
```

##### Test ventilateurs

```bash
# Voir vitesse ventilateurs
sensors | grep fan

# Si 0 RPM ou < 500 RPM : ventilateur défectueux

# Test manuel ventilateur (si controllable)
echo 255 > /sys/class/hwmon/hwmon0/pwm1  # Max speed
sleep 5
sensors | grep fan
# Fan devrait être à max RPM

echo 128 > /sys/class/hwmon/hwmon0/pwm1  # 50% speed
```

---

#### Procédures de diagnostic méthodiques

##### Procédure panne aléatoire

```bash
# 1. Vérifier logs kernel
dmesg | tail -100
journalctl -p err -n 100

# 2. Test RAM (8h)
reboot # sélectionner memtest86+

# 3. Si RAM OK, test CPU
stress-ng --cpu $(nproc) --timeout 3600s

# 4. Si CPU OK, test disques
for disk in /dev/sd[a-z]; do
    smartctl -H $disk
    smartctl -l selftest $disk
done

# 5. Si tout OK, vérifier PSU et température
stress-ng --cpu $(nproc) --vm 4 --io 4 --timeout 600s
watch sensors
```

##### Procédure corruption données

```bash
# 1. Test RAM PRIORITAIRE
stress-ng --vm 4 --vm-bytes 2G --vm-method all --verify --timeout 3600s

# 2. Si erreurs mémoire
stress-ng: error: memory corruption detected
# → Test memtest86+ 8h minimum
# → Identifier barrette défectueuse

# 3. Test disque
smartctl -t long /dev/sda
# Attendre fin puis :
smartctl -l selftest /dev/sda

# 4. Test filesystem
umount /dev/sda1
e2fsck -fv /dev/sda1  # ext4
xfs_repair -v /dev/sda1  # xfs
```

##### Procédure crash sous charge

```bash
# 1. Monitoring température pendant crash
sensors > /tmp/temps_before.txt
stress-ng --cpu $(nproc) --timeout 60s &
watch -n 1 "sensors | tee -a /tmp/temps_during.txt"

# 2. Si throttling ou >85°C
# → Problème refroidissement
# → Nettoyer radiateur, changer pâte thermique

# 3. Si température OK
# → Test PSU
stress-ng --cpu $(nproc) --vm 4 --io 4 --hdd 2 --timeout 600s

# 4. Si reboot pendant stress
# → PSU insuffisant ou défectueux
# → Vérifier wattage PSU vs consommation
```

---

#### Checklist diagnostic hardware

✅ **Tests RAM :**

- [ ]  memtest86+ 8h minimum
- [ ]  stress-ng avec --verify
- [ ]  MCE/EDAC logs vérifiés
- [ ]  Barrettes testées individuellement si erreur

✅ **Tests CPU :**

- [ ]  stress-ng CPU 1h
- [ ]  mprime torture 24h
- [ ]  Températures surveillées
- [ ]  MCE errors vérifiés

✅ **Tests disques :**

- [ ]  SMART health OK
- [ ]  SMART extended test
- [ ]  badblocks lecture
- [ ]  Performance test (hdparm/dd)
- [ ]  dmesg I/O errors vérifiés

✅ **Tests système :**

- [ ]  PSU stress test complet
- [ ]  Ventilateurs vérifiés (RPM)
- [ ]  Températures sous charge
- [ ]  Logs kernel analysés

✅ **Documentation :**

- [ ]  Résultats tests documentés
- [ ]  Erreurs logs sauvegardées
- [ ]  Composants défectueux identifiés
- [ ]  Procédure remplacement planifiée

---

#### Scripts automatiques

##### Script diagnostic complet

```bash
#!/bin/bash
# /usr/local/bin/hardware-diag.sh

LOG_DIR="/var/log/hardware-diag"
mkdir -p $LOG_DIR
DATE=$(date +%Y%m%d_%H%M%S)

echo "=== Hardware Diagnostic Started: $DATE ===" | tee $LOG_DIR/diag_${DATE}.log

# 1. System info
echo -e "\n=== System Info ===" | tee -a $LOG_DIR/diag_${DATE}.log
lscpu | tee -a $LOG_DIR/diag_${DATE}.log
free -h | tee -a $LOG_DIR/diag_${DATE}.log
lsblk | tee -a $LOG_DIR/diag_${DATE}.log

# 2. Kernel errors
echo -e "\n=== Kernel Errors ===" | tee -a $LOG_DIR/diag_${DATE}.log
dmesg | grep -i "error\|fail\|hardware" | tail -50 | tee -a $LOG_DIR/diag_${DATE}.log

# 3. MCE errors
if command -v mcelog &> /dev/null; then
    echo -e "\n=== MCE Errors ===" | tee -a $LOG_DIR/diag_${DATE}.log
    mcelog --client | tee -a $LOG_DIR/diag_${DATE}.log
fi

# 4. SMART status
echo -e "\n=== SMART Status ===" | tee -a $LOG_DIR/diag_${DATE}.log
for disk in /dev/sd[a-z] /dev/nvme[0-9]n[0-9]; do
    if [ -e "$disk" ]; then
        echo "--- $disk ---" | tee -a $LOG_DIR/diag_${DATE}.log
        smartctl -H $disk | tee -a $LOG_DIR/diag_${DATE}.log
        smartctl -A $disk | grep -E "Reallocated|Pending|Uncorrectable" | tee -a $LOG_DIR/diag_${DATE}.log
    fi
done

# 5. Memory test (quick)
echo -e "\n=== Memory Quick Test ===" | tee -a $LOG_DIR/diag_${DATE}.log
stress-ng --vm 2 --vm-bytes 1G --verify --timeout 60s 2>&1 | tee -a $LOG_DIR/diag_${DATE}.log

# 6. CPU test (quick)
echo -e "\n=== CPU Quick Test ===" | tee -a $LOG_DIR/diag_${DATE}.log
stress-ng --cpu $(nproc) --timeout 60s --metrics-brief 2>&1 | tee -a $LOG_DIR/diag_${DATE}.log

# 7. Temperatures
echo -e "\n=== Temperatures ===" | tee -a $LOG_DIR/diag_${DATE}.log
sensors | tee -a $LOG_DIR/diag_${DATE}.log

echo -e "\n=== Diagnostic Complete ===" | tee -a $LOG_DIR/diag_${DATE}.log
echo "Log saved to: $LOG_DIR/diag_${DATE}.log"
```

```bash
# Rendre exécutable
chmod +x /usr/local/bin/hardware-diag.sh

# Lancer
/usr/local/bin/hardware-diag.sh
```

---

#### Conclusion

Le diagnostic hardware nécessite une approche méthodique et des tests exhaustifs. Les outils memtest86+, stress-ng, badblocks et l'analyse des logs kernel permettent d'identifier précisément les composants défectueux.

**Points clés :**

- RAM : memtest86+ 8h minimum
- CPU : stress-ng + mprime 24h
- Disques : SMART tests + badblocks
- Logs kernel : dmesg, MCE, EDAC essentiels
- Tests charge complète pour PSU

**Symptômes vs composants :**

- Crash aléatoire → RAM
- Crash sous charge → PSU/refroidissement
- Corruption données → RAM/disque
- Calculs incorrects → CPU
- I/O errors → disque

**Actions prioritaires :**

1. Sauvegarder logs kernel avant tests
2. Commencer par RAM (cause #1)
3. Tests exhaustifs (8h+ RAM, 24h+ CPU)
4. Remplacer proactivement composants douteux
5. Documenter tous les tests