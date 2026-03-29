---
link: https://www.glukhov.org/fr/post/2025/11/go-cli-applications-with-cobra-and-viper/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Découvrez comment créer des applications de ligne de commande
  professionnelles en Go en utilisant Cobra pour la structure CLI et Viper pour
  la gestion de la configuration. Guide complet avec des exemples pratiques et
  des bonnes pratiques.
slurped: 2026-03-22T12:37
title: Création d'applications CLI en Go avec Cobra et Viper
---

Les applications d’interface en ligne de commande (CLI) sont des outils essentiels pour les développeurs, les administrateurs système et les professionnels DevOps. Deux bibliothèques Go sont devenues le standard de facto pour le développement CLI en Go : [Cobra pour la structure des commandes et Viper pour la gestion de la configuration](https://www.glukhov.org/fr/post/2025/11/go-cli-applications-with-cobra-and-viper/ "Développement CLI en Go").

Go est devenu un excellent langage pour construire des outils CLI en raison de ses performances, de son déploiement simple et de son support multiplateforme.

## Pourquoi choisir Go pour les applications CLI

Go offre des avantages convaincants pour le développement CLI :

- **Distribution d’un seul binaire** : Aucun dépendance d’exécution ou gestionnaire de paquets requis
- **Exécution rapide** : La compilation native offre de bonnes performances
- **Support multiplateforme** : Compilation facile pour Linux, macOS, Windows et plus
- **Bibliothèque standard puissante** : Outils riches pour l’E/S de fichiers, le réseau et le traitement du texte
- **Concurrence** : Goroutines intégrées pour les opérations parallèles
- **Typage statique** : Détecter les erreurs à la compilation

Les outils CLI populaires construits avec Go incluent Docker, Kubernetes (kubectl), Hugo, Terraform et GitHub CLI. Si vous êtes nouveau dans Go ou avez besoin d’un rappel rapide, consultez notre [Feuille de triche Go](https://www.glukhov.org/fr/post/2022/golang-cheatsheet/ "Feuille de triche Golang") pour la syntaxe essentielle et les modèles Go.

## Introduction à Cobra

Cobra est une bibliothèque qui fournit une interface simple pour créer des applications CLI puissantes et modernes. Créé par Steve Francia (spf13), le même auteur derrière Hugo et Viper, Cobra est utilisé dans de nombreux projets Go les plus populaires.

### Fonctionnalités clés de Cobra

**Structure de commande** : Cobra implémente le modèle de commande, vous permettant de créer des applications avec des commandes et des sous-commandes (comme `git commit` ou `docker run`).

**Gestion des drapeaux** : Drapeaux locaux et persistants avec un parsing automatique et une conversion de type.

**Aide automatique** : Génère automatiquement du texte d’aide et des informations d’utilisation.

**Suggestions intelligentes** : Fournit des suggestions lorsque les utilisateurs commettent des fautes de frappe (“Avez-vous voulu dire ‘status’ ?”).

**Complétions de shell** : Génère des scripts de complétion pour bash, zsh, fish et PowerShell.

**Sortie flexible** : Fonctionne sans problème avec des formateurs personnalisés et des styles de sortie.

## Introduction à Viper

Viper est une solution complète de configuration pour les applications Go, conçue pour fonctionner de manière fluide avec Cobra. Il gère la configuration provenant de plusieurs sources avec un ordre de priorité clair.

### Fonctionnalités clés de Viper

**Plusieurs sources de configuration** :

- Fichiers de configuration (JSON, YAML, TOML, HCL, INI, envfile, Java properties)
- Variables d’environnement
- Drapeaux de ligne de commande
- Systèmes de configuration distants (etcd, Consul)
- Valeurs par défaut

**Ordre de priorité de configuration** :

1. Appels explicites à Set
2. Drapeaux de ligne de commande
3. Variables d’environnement
4. Fichier de configuration
5. Magasin clé/valeur
6. Valeurs par défaut

**Surveillance en temps réel** : Surveille les fichiers de configuration et recharge automatiquement lorsqu’ils changent.

**Conversion de type** : Conversion automatique vers divers types Go (chaîne, entier, booléen, durée, etc.).

## Démarrage : Installation

Tout d’abord, initialisez un nouveau module Go et installez les deux bibliothèques :

```
go mod init myapp
go get -u github.com/spf13/cobra@latest
go get -u github.com/spf13/viper@latest
```

Optionnellement, installez le générateur de CLI Cobra pour la génération de modèles :

```
go install github.com/spf13/cobra-cli@latest
```

## Création de votre première application CLI

Construisons un exemple pratique : un outil CLI de gestion des tâches avec un support de configuration.

### Structure du projet

```
mytasks/
├── cmd/
│   ├── root.go
│   ├── add.go
│   ├── list.go
│   └── complete.go
├── config/
│   └── config.go
├── main.go
└── config.yaml
```

### Point d’entrée principal

```
// main.go
package main

import "mytasks/cmd"

func main() {
    cmd.Execute()
}
```

### Commande principale avec intégration Viper

```
// cmd/root.go
package cmd

import (
    "fmt"
    "os"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var cfgFile string

var rootCmd = &cobra.Command{
    Use:   "mytasks",
    Short: "Un simple outil CLI de gestion des tâches",
    Long: `MyTasks est un outil CLI de gestion des tâches qui vous aide à organiser 
vos tâches quotidiennes avec facilité. Construit avec Cobra et Viper.`,
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}

func init() {
    cobra.OnInitialize(initConfig)
    
    rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", 
        "fichier de configuration (par défaut est $HOME/.mytasks.yaml)")
    rootCmd.PersistentFlags().String("db", "", 
        "emplacement du fichier de base de données")
    
    viper.BindPFlag("database", rootCmd.PersistentFlags().Lookup("db"))
}

func initConfig() {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        home, err := os.UserHomeDir()
        if err != nil {
            fmt.Fprintln(os.Stderr, err)
            os.Exit(1)
        }
        
        viper.AddConfigPath(home)
        viper.AddConfigPath(".")
        viper.SetConfigType("yaml")
        viper.SetConfigName(".mytasks")
    }
    
    viper.SetEnvPrefix("MYTASKS")
    viper.AutomaticEnv()
    
    if err := viper.ReadInConfig(); err == nil {
        fmt.Println("Utilisation du fichier de configuration :", viper.ConfigFileUsed())
    }
}
```

### Ajout de sous-commandes

```
// cmd/add.go
package cmd

import (
    "fmt"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var priority string

var addCmd = &cobra.Command{
    Use:   "add [description de la tâche]",
    Short: "Ajouter une nouvelle tâche",
    Args:  cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        task := args[0]
        db := viper.GetString("database")
        
        fmt.Printf("Ajout de la tâche : %s\n", task)
        fmt.Printf("Priorité : %s\n", priority)
        fmt.Printf("Base de données : %s\n", db)
        
        // Ici vous implémenteriez le stockage réel des tâches
    },
}

func init() {
    rootCmd.AddCommand(addCmd)
    
    addCmd.Flags().StringVarP(&priority, "priority", "p", "medium", 
        "Priorité de la tâche (faible, moyenne, élevée)")
}
```

### Commande Liste

```
// cmd/list.go
package cmd

import (
    "fmt"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var showCompleted bool

var listCmd = &cobra.Command{
    Use:   "list",
    Short: "Lister toutes les tâches",
    Run: func(cmd *cobra.Command, args []string) {
        db := viper.GetString("database")
        
        fmt.Printf("Liste des tâches depuis : %s\n", db)
        fmt.Printf("Afficher les tâches terminées : %v\n", showCompleted)
        
        // Ici vous implémenteriez le listing réel des tâches
    },
}

func init() {
    rootCmd.AddCommand(listCmd)
    
    listCmd.Flags().BoolVarP(&showCompleted, "completed", "c", false, 
        "Afficher les tâches terminées")
}
```

### Implémentation de la persistance des données

Pour un outil de gestion de tâches CLI en production, vous devrez implémenter un stockage réel des données. Bien que nous utilisions ici une configuration simple du chemin de la base de données, vous avez plusieurs options pour persister les données :

- **SQLite** : Base de données légère et sans serveur parfaite pour les outils CLI
- **PostgreSQL/MySQL** : Bases de données complètes pour des applications plus complexes
- **Fichiers JSON/YAML** : Stockage basé sur des fichiers simple pour des besoins légers
- **Bases de données embarquées** : BoltDB, BadgerDB pour le stockage clé-valeur

Si vous travaillez avec des bases de données relationnelles comme PostgreSQL, vous voudrez utiliser un ORM ou un générateur de requêtes. Pour une comparaison complète des bibliothèques de base de données Go, consultez notre guide sur [Comparaison des ORMs Go pour PostgreSQL : GORM vs Ent vs Bun vs sqlc](https://www.glukhov.org/fr/app-architecture/data-access/comparing-go-orms-gorm-ent-bun-sqlc/ "Comparaison des ORMs Go pour PostgreSQL").

## Configuration avancée avec Viper

### Exemple de fichier de configuration

```
# .mytasks.yaml
database: ~/.mytasks.db
format: table
colors: true

notifications:
  enabled: true
  sound: true

priorities:
  high: red
  medium: yellow
  low: green
```

### Lecture de la configuration imbriquée

```
func getNotificationSettings() {
    enabled := viper.GetBool("notifications.enabled")
    sound := viper.GetBool("notifications.sound")
    
    fmt.Printf("Notifications activées : %v\n", enabled)
    fmt.Printf("Son activé : %v\n", sound)
}

func getPriorityColors() map[string]string {
    return viper.GetStringMapString("priorities")
}
```

### Variables d’environnement

Viper lit automatiquement les variables d’environnement avec le préfixe configuré :

```
export MYTASKS_DATABASE=/tmp/tasks.db
export MYTASKS_NOTIFICATIONS_ENABLED=false
mytasks list
```

### Rechargement de la configuration en temps réel

```
func watchConfig() {
    viper.WatchConfig()
    viper.OnConfigChange(func(e fsnotify.Event) {
        fmt.Println("Fichier de configuration modifié :", e.Name)
        // Recharger les composants dépendants de la configuration
    })
}
```

## Bonnes pratiques

### 1. Utilisez le générateur Cobra pour la cohérence

```
cobra-cli init
cobra-cli add serve
cobra-cli add create
```

### 2. Organisez les commandes dans des fichiers séparés

Gardez chaque commande dans son propre fichier sous le répertoire `cmd/` pour une maintenance facilitée.

### 3. Utilisez les drapeaux persistants

Utilisez les drapeaux persistants pour les options qui s’appliquent à toutes les sous-commandes :

```
rootCmd.PersistentFlags().StringP("output", "o", "text", 
    "Format de sortie (texte, json, yaml)")
```

### 4. Implémentez une gestion d’erreur appropriée

```
var rootCmd = &cobra.Command{
    Use:   "myapp",
    Short: "Mon application",
    RunE: func(cmd *cobra.Command, args []string) error {
        if err := doSomething(); err != nil {
            return fmt.Errorf("échec de l'exécution : %w", err)
        }
        return nil
    },
}
```

### 5. Fournissez un texte d’aide significatif

```
var cmd = &cobra.Command{
    Use:   "deploy [environnement]",
    Short: "Déployer l'application vers l'environnement spécifié",
    Long: `Déployer les builds et déployer votre application vers 
l'environnement spécifié. Environnements pris en charge :
  - développement (dev)
  - staging
  - production (prod)`,
    Example: `  myapp deploy staging
  myapp deploy production --version=1.2.3`,
}
```

### 6. Définissez des valeurs par défaut sensibles

```
func init() {
    viper.SetDefault("port", 8080)
    viper.SetDefault("timeout", "30s")
    viper.SetDefault("retries", 3)
}
```

### 7. Validez la configuration

```
func validateConfig() error {
    port := viper.GetInt("port")
    if port < 1024 || port > 65535 {
        return fmt.Errorf("port invalide : %d", port)
    }
    return nil
}
```

## Test des applications CLI

### Test des commandes

```
// cmd/root_test.go
package cmd

import (
    "bytes"
    "testing"
)

func TestRootCommand(t *testing.T) {
    cmd := rootCmd
    b := bytes.NewBufferString("")
    cmd.SetOut(b)
    cmd.SetArgs([]string{"--help"})
    
    if err := cmd.Execute(); err != nil {
        t.Fatalf("exécution de la commande échouée : %v", err)
    }
}
```

### Test avec différentes configurations

```
func TestWithConfig(t *testing.T) {
    viper.Set("database", "/tmp/test.db")
    viper.Set("debug", true)
    
    // Exécutez vos tests
    
    viper.Reset() // Nettoyage
}
```

## Génération des complétions de shell

Cobra peut générer des scripts de complétion pour divers shells :

```
// cmd/completion.go
package cmd

import (
    "os"
    "github.com/spf13/cobra"
)

var completionCmd = &cobra.Command{
    Use:   "completion [bash|zsh|fish|powershell]",
    Short: "Générer un script de complétion",
    Long: `Pour charger les complétions :

Bash :
  $ source <(mytasks completion bash)

Zsh :
  $ source <(mytasks completion zsh)

Fish :
  $ mytasks completion fish | source

PowerShell :
  PS> mytasks completion powershell | Out-String | Invoke-Expression
`,
    DisableFlagsInUseLine: true,
    ValidArgs: []string{"bash", "zsh", "fish", "powershell"},
    Args: cobra.ExactValidArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        switch args[0] {
        case "bash":
            cmd.Root().GenBashCompletion(os.Stdout)
        case "zsh":
            cmd.Root().GenZshCompletion(os.Stdout)
        case "fish":
            cmd.Root().GenFishCompletion(os.Stdout, true)
        case "powershell":
            cmd.Root().GenPowerShellCompletionWithDesc(os.Stdout)
        }
    },
}

func init() {
    rootCmd.AddCommand(completionCmd)
}
```

## Construction et distribution

### Construire pour plusieurs plateformes

```
# Linux
GOOS=linux GOARCH=amd64 go build -o mytasks-linux-amd64

# macOS
GOOS=darwin GOARCH=amd64 go build -o mytasks-darwin-amd64
GOOS=darwin GOARCH=arm64 go build -o mytasks-darwin-arm64

# Windows
GOOS=windows GOARCH=amd64 go build -o mytasks-windows-amd64.exe
```

### Réduire la taille du binaire

```
go build -ldflags="-s -w" -o mytasks
```

### Ajouter des informations de version

```
// cmd/version.go
package cmd

import (
    "fmt"
    "github.com/spf13/cobra"
)

var (
    Version   = "dev"
    Commit    = "none"
    BuildTime = "unknown"
)

var versionCmd = &cobra.Command{
    Use:   "version",
    Short: "Afficher les informations de version",
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Printf("Version : %s\n", Version)
        fmt.Printf("Commit : %s\n", Commit)
        fmt.Printf("Construit : %s\n", BuildTime)
    },
}

func init() {
    rootCmd.AddCommand(versionCmd)
}
```

Construire avec des informations de version :

```
go build -ldflags="-X 'mytasks/cmd.Version=1.0.0' \
    -X 'mytasks/cmd.Commit=$(git rev-parse HEAD)' \
    -X 'mytasks/cmd.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ)'" \
    -o mytasks
```

## Exemples concrets

Outils open source populaires construits avec Cobra et Viper :

- **kubectl** : Outil CLI Kubernetes
- **Hugo** : Générateur de site statique
- **GitHub CLI (gh)** : CLI officiel de GitHub
- **Docker CLI** : Gestion des conteneurs
- **Helm** : Gestionnaire de paquets Kubernetes
- **Skaffold** : Outil de workflow de développement Kubernetes
- **Cobra CLI** : Auto-hébergé - Cobra s’utilise lui-même !

Au-delà des outils DevOps traditionnels, les capacités CLI de Go s’étendent aux applications d’intelligence artificielle et de machine learning. Si vous êtes intéressé par la création d’outils CLI qui interagissent avec des modèles de langage de grande envergure, consultez notre guide sur [Contraindre les LLM avec une sortie structurée en utilisant Ollama et Go](https://www.glukhov.org/fr/llm-performance/ollama/llm-structured-output-with-ollama-in-python-and-go/ "Contraindre les LLM avec une sortie structurée"), qui montre comment construire des applications Go qui travaillent avec des modèles d’IA.

## Ressources utiles

- [Dépôt GitHub de Cobra](https://github.com/spf13/cobra)
- [Dépôt GitHub de Viper](https://github.com/spf13/viper)
- [Guide utilisateur de Cobra](https://github.com/spf13/cobra/blob/main/user_guide.md)
- [Documentation de Viper](https://github.com/spf13/viper#viper)
- [Meilleures pratiques pour les applications CLI en Go](https://golang.org/doc/code)

## Conclusion

Cobra et Viper ensemble offrent une base puissante pour construire des applications CLI professionnelles en Go. Cobra gère la structure des commandes, le parsing des drapeaux et la génération d’aide, tandis que Viper gère la configuration provenant de plusieurs sources avec une priorité intelligente.

Cette combinaison vous permet de créer des outils CLI qui sont :

- Faciles à utiliser avec des commandes intuitives et une aide automatique
- Flexibles avec plusieurs sources de configuration
- Professionnels avec des complétions de shell et une gestion appropriée des erreurs
- Maintenables avec une organisation de code propre
- Portables sur différentes plateformes

Quel que soit le type d’outils que vous construisez, qu’ils soient des outils de développement, des utilitaires système ou des outils d’automatisation DevOps, Cobra et Viper offrent la base solide nécessaire pour créer des applications CLI que les utilisateurs adoreront.