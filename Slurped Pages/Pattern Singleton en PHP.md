---
link: https://guillaume-richard.fr/le-design-pattern-singleton-en-php/
byline: Guillaume RICHARD
site: Portfolio Guillaume RICHARD
date: 2025-01-29T11:00
excerpt: Le design pattern Singleton garantit une instance unique d'une classe en PHP, idéale pour la gestion de connexions BD, logs, etc...
slurped: 2026-06-15T08:02
title: Le Design Pattern Singleton en PHP - Portfolio Guillaume RICHARD
tags:
  - php
---

Le **design pattern Singleton** est un **patron de conception** qui garantit qu’une classe n’a **qu’une seule instance** et fournit un **point d’accès global** à cette instance. Ce pattern est souvent utilisé lorsque **une seule instance d’une classe doit être partagée** à travers une application. Par exemple pour gérer une connexion à une base de données, un logger ou une configuration.

#### **Caractéristiques du Singleton**

1. **Instance unique** : Une seule instance de la classe est créée durant l’exécution du programme.
2. **Contrôle d’accès** : L’accès à cette instance se fait via une méthode statique.
3. **Constructeur privé** : Empêche l’instanciation de la classe depuis l’extérieur.
4. **Stockage en variable statique** : L’instance unique est stockée dans une variable statique.

---

### **Exemple concret en PHP : Singleton pour une connexion PDO**

Prenons un exemple où nous utilisons le **Singleton** pour gérer une connexion **PDO** à une base de données **MySQL**.

#### **Implémentation du Singleton**

![Implémentation du Design Pattern Singleton en PHP](https://guillaume-richard.fr/wp-content/uploads/2025/01/image-2-734x1024.png)

---

### **Explication du code**

1. **Attribut statique `$instance`**
    - Il stocke l’instance unique de la classe.
2. **Constructeur privé `__construct()`**
    - Empêche l’instanciation depuis l’extérieur.
3. **Méthode statique `getInstance()`**
    - Vérifie si une instance existe déjà. Si non, elle en crée une nouvelle.
    - Retourne toujours la même instance.
4. **Méthode `getConnection()`**
    - Fournit l’objet **PDO** pour interagir avec la base de données.
5. **Test d’égalité (`var_dump($db1 === $db2)`)**
    - Vérifie que les deux instances obtenues sont identiques.

---

### **Avantages du Singleton**

1. **Optimisation mémoire** : Évite d’instancier plusieurs objets inutiles.  
2. **Accès global contrôlé** : Fournit un point d’accès unique et centralisé.  
3. **Facilite le partage de ressources** : Utile pour la gestion des connexions, des logs, etc.

### **Inconvénients du Singleton**

1. **Difficulté à tester** : Les tests unitaires sont plus compliqués, car le Singleton crée une dépendance cachée.  
2. **Risque de surcharge mémoire** : Si l’instance unique conserve trop d’informations.  
3. **Couplage fort** : Peut rendre le code moins flexible et moins évolutif.

---

### **Alternatives**

- **Injection de dépendances (DI)** : Permet de gérer les instances via un conteneur de services.
- **Registre global** : Stocker des instances partagées dans un registre centralisé.

#### **Résumé**

Le **Singleton en PHP** est un bon choix pour gérer **une seule instance d’un service**, comme une **connexion à une base de données**. Toutefois, son utilisation doit être réfléchie pour éviter un couplage excessif et des problèmes de testabilité.