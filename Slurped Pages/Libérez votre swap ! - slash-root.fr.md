---
link: https://slash-root.fr/liberez-votre-swap/
byline: Julien LOUIS
site: slash-root.fr
date: 2024-12-01T05:09
excerpt: Libérez le swap de vos debian !
slurped: 2025-01-17T14:45
title: Libérez votre swap ! - slash-root.fr
---

## Préambule

Envie de libérer Britney ? Ou plutôt... votre swap ? 🎵

Si vous cherchez à optimiser l'utilisation de votre mémoire virtuelle sur une machine Debian, vous êtes au bon endroit !

---

### Le `swappiness`, c'est quoi ?

Le `swappiness` est un paramètre du kernel Linux qui contrôle la "proportion" du système à utiliser le swap. Une valeur faible favorise l'utilisation de la RAM, tandis qu'une valeur élevée incite à swapper rapidement sur le disque.

---

## Quelques commandes essentielles

### 1. Vérifier la valeur actuelle du `swappiness`

```
cat /proc/sys/vm/swappiness
```

### 2. Modifier dynamiquement le `swappiness`

```
sudo sysctl vm.swappiness=20
```

> ⚠️ Cette modification n'est pas persistante et sera perdue au redémarrage.

### 3. Rendre la configuration persistante

Ajoutez ou modifiez la ligne suivante dans `/etc/sysctl.conf` :

```
vm.swappiness=20
```

Appliquez ensuite la configuration :

```
sudo sysctl -p
```

### 4. Comprendre les valeurs du `swappiness`

- **100** : Le système swap intensément.
- **0** : Swap uniquement en cas de RAM complètement saturée.
- **Défaut** : Généralement **60** sur Debian.

### 5. Désactiver et réactiver le swap

```
sudo swapoff -a && sudo swapon -a
```

> ⚠️ Assurez-vous que la RAM disponible est suffisante pour accueillir les données du swap avant de désactiver ce dernier.

### 6. Identifier les processus utilisant le swap

Pour examiner les processus gourmands en swap, utilisez cette commande :

```
for file in /proc/*/status ; do \
    awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file; \
    done | \
sort -k 2 -n -r | less
```

---

## Automatisation avec un script Bash

Pour simplifier la gestion du swap, voici un script bash prêt à l'emploi :

### Script : `free-swap.sh`

```
#!/bin/bash

check_command() {
    if [[ $? -ne 0 ]]; then
        echo "Erreur lors de l'exécution de la commande: $1"
        exit 1
    fi
}

free_data=$(free)
check_command "free"
mem_data=$(echo "$free_data" | grep 'Mem:')
free_mem=$(echo "$mem_data" | awk '{print $4}')
buffers=$(echo "$mem_data" | awk '{print $6}')
cache=$(echo "$mem_data" | awk '{print $7}')
total_free=$((free_mem + buffers + cache))

if echo "$free_data" | grep -q 'Échange:'; then
    used_swap=$(echo "$free_data" | grep 'Échange:' | awk '{print $3}')
else
    used_swap=$(echo "$free_data" | grep 'Swap:' | awk '{print $3}')
fi

echo -e "Free memory:\t$total_free kB ($((total_free / 1024)) MB)\nUsed swap:\t$used_swap kB ($((used_swap / 1024)) MB)"

if [[ $used_swap -eq 0 ]]; then
    echo "Félicitations ! Aucun swap n'est utilisé."
elif [[ $used_swap -lt $total_free ]]; then
    echo "Libération du swap..."
    sudo swapoff -a
    check_command "swapoff"
    sudo swapon -a
    check_command "swapon"
else
    echo "Pas assez de mémoire libre pour libérer le swap. Arrêt."
    exit 1
fi
```

### Instructions :

1. Enregistrez le script dans un fichier, par exemple : `free-swap.sh`.
2. Rendez-le exécutable :

```
chmod u+x free-swap.sh
```

1. Exécutez-le :

```
./free-swap.sh
```

> ⚠️ Ne pas utiliser ce script si **ZRAM** est activé sur votre système, car cela pourrait entraîner des comportements inattendus.

---

## Conclusion

Vous venez de découvrir comment optimiser la gestion du swap sous Debian. Que ce soit via des commandes ou un script Bash, vous êtes désormais armé pour reprendre le contrôle de votre mémoire virtuelle.

Et surtout, souvenez-vous : le swap, c'est bien... mais la RAM, c'est mieux ! 😉