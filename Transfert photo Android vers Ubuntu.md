---
tags:
  - photos
  - ubuntu
---
ADB est plus fiable que MTP et prend en charge les scripts. Vous devez activer le débogage USB sur le téléphone.

### Activer ADB sur Android[](https://oneuptime.com/blog/post/2026-03-02-transfer-files-android-ubuntu/view#enabling-adb-on-android)

   1 Accéder à Paramétres > Sécurité et confidentialité
      Bloqueur automatique -> déactiver 
    puis
1. Accédez à Paramètres > À propos du téléphone
2. Appuyez sept fois sur « Numéro de version » pour activer les options pour les développeurs.
3. Accédez à Paramètres > Options pour les développeurs > Activer le débogage USB

### Installation d'ADB sur Ubuntu[](https://oneuptime.com/blog/post/2026-03-02-transfer-files-android-ubuntu/view#installing-adb-on-ubuntu)

Copie

`# Install ADB from the Ubuntu repositories sudo apt install adb -y # Verify ADB can see the phone adb devices`

Votre appareil devrait apparaître dans la liste. Si le message « non autorisé » s'affiche, vérifiez si une boîte de dialogue de jumelage apparaît sur votre téléphone et acceptez-la.

### Transfert de fichiers avec ADB[](https://oneuptime.com/blog/post/2026-03-02-transfer-files-android-ubuntu/view#transferring-files-with-adb)

```# Copy a file from Ubuntu to the phone
adb push ~/Documents/file.pdf /sdcard/Documents/

# Copy a file from the phone to Ubuntu
adb pull /sdcard/DCIM/Camera/photo.jpg ~/Pictures/

# Copy an entire directory from the phone
adb pull /sdcard/DCIM/Camera ~/Pictures/

# Push an entire directory to the phone
adb push ~/Music/album /sdcard/Music/

# List files on the phone
adb shell ls /sdcard/

# Delete a file on the phone
adb shell rm /sdcard/unwanted_file.txt

```

pour copier les photos : adb pull /sdcard/DCIM/Camera ~/Pictures/
