---
link: https://mcorbin.fr/posts/2022-06-13-gin-golang/
site: "@_mcorbin"
excerpt: mcorbin Tech Blog
twitter: https://twitter.com/@_mcorbin
slurped: 2025-10-26T16:35
title: "API HTTP en Golang: Gin, logs, gestion d'erreurs et des paramètres, OpenAPI"
tags:
  - golang
  - openapi
---

_disclaimer_: Cet article traînait dans mes brouillons depuis un moment. J’ai décidé de le terminer rapidement, d’où son manque de structure. Tant pis, l’essentiel est là et je suis sûr que cela pourra aider des gens quand même. Il vaut mieux ça plutôt que de le laisser éternellement en brouillon (je sentais ma motivation pour le terminer baisser).

J’ai commencé récemment à travailler sur un nouveau projet personnel écrit en Golang, et comme souvent j’ai comme besoin d’écrire une API HTTP.  
J’ai longtemps utilisé le framrwork [Echo](https://echo.labstack.com/) pour cela lorsque je faisais des projets Go mais j’ai récemment eu l’occasion de travailler un peu avec [Gin](https://github.com/gin-gonic/gin), et j’ai finalement décidé pour ce projet d’utiliser ce framework.

C’est aussi son écosystème qui m’a attiré: je voulais en effet avoir plusieurs choses gérées par mon projet et grâce aux conseils de quelques personnes avisées j’ai découvert plusieurs libs intéressantes pour Gin que je présenterai dans cet article.

Le code de cet article n’a pas vocation à être parfait (gestion d’erreurs manquante par exemple), mon objectif étant de présenter les libs.

### Logging

J’utilise depuis longtemps [Zap](https://github.com/uber-go/zap) comme logger. Rapide, facile à utiliser, avec beaucoup d’options, un support correct pour faire des logs structurés…​ ça fait le job.

l’intérêt de Zap est aussi son écosystème. Et bien sûr, il s’intègre avec Gin via la lib [gin-contrib/zap](https://github.com/gin-contrib/zap).

```
import (
	"time"

	ginzap "github.com/gin-contrib/zap"
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
)

func main() {
	zapConfig := zap.NewProductionConfig()
	logger, _ := zapConfig.Build()
	gin.SetMode(gin.ReleaseMode)
	router := gin.New()
	router.Use(ginzap.Ginzap(logger, time.RFC3339, true))
        router.Use(ginzap.RecoveryWithZap(logger, true))
}
```

Je crée ici un logger Zap (qui peut être configuré comme vous le souhaitez). J’en profite pour passer gin en `ReleaseMode` pour enlever certains logs de debug. je crée ensuite mon router gin que je configure avec `ginzap.Ginzap`. Toutes mes requêtes seront maintenant logués en utilisant mon logger zap (et donc avec sa configuration).+

J’utilise aussi `ginzap.RecoveryWithZap` pour log tous les `panic` potentiels générés par mes handlers.

### Tonic

Les handlers Gin sont par défaut assez légers. Voici par exemple un exemple tiré de la documentation de Gin:

```
router.GET("/ping", func(c *gin.Context) {
	c.JSON(200, gin.H{
		"message": "pong",
	})
})
```

Une fonction prenant un `*gin.Context` est associé à une méthode HTTP et à une URL (ici `GET /ping`.

Toute la gestion de la sérialisation des paramètres de la requête (ces paramètres pouvant venir de l’url, être passé en query ou dans le body de la requête) est à gérer à la main.  
Par exemple, récupérer une variable depuis l’url de la requête (`/user/:id` par exemple) se fait via `c.Param("id")`. Récupérer un paramètre passé en query string (`?foo=bar` par exemple) se fait avec `c.Query("foo")`. Sérialiser le body d’une requête en JSON dans une struct Golang demande un appel à `c.ShouldBindJSON`…​

Constuire des réponses HTTP avec Gin se fait via des appels du type `c.JSON(http.StatusOK, repsonse)`: on passe donc notre code HTTP et `response` qui est une struct Go qui sera ensuite convertit en JSON.

Tout cela fonctionne mais est assez pénible et répétitif à faire. C''est notamment là qu’intervient la lib [Tonic](https://github.com/loopfz/gadgeto/tree/master/tonic).

**Handlers évolués**

Tonic fournit des handlers évolués. Là où en Gin classique un handler est comme vu précédemment une fonction ayant comme signature `func(c *gin.Context)`, Tonic permet de créer des handlers prenant une struct en paramètre et en retourtant une autre, avec optionnellement une erreur.

Voici par exemple un handler Tonic:

```
type MyInput struct {
	Name string `query:"bar" default:"foobar" validate:"required,max=255"`
        Foo int    `path:"foo" validate:"required,gt=10"`
        Baz string `json:"baz" validate:"required,email"`
}

type MyOutput struct {
        Message string `json:"message"`
}

func MyHandler(c *gin.Context, in *MyInput) (*MyOutput, error) {
	return &MyOutput{Message: fmt.Sprintf("Hello %s!", in.Name)}, nil
}
```

On remarque ici plusieurs choses:

- Le type `MyInput` a des tags `query`, `path`, `json`. C’est le premier intérêt de Tonic: une struct peut se construire instantanément depuis plusieurs types de paramètres (query, path, body, et même headers !). Plus besoin de répéter toute la logique d’extration de paramètres pour construire vos struct, Tonic le fait pour vous.
    
- Tonic supporte comme Gin le tag `validate` pour ajouter de la validation sur nos champs: taille du champ, champ obligatoire ou non, formats spécifiques comme `email`, enum…​ C’est ensuite la lib [go-playground/validator](https://github.com/go-playground/validator) qui se charge de valider les paramètres.
    
- Le Handler `MyHandler` prend un context Gin et une struct appelée `in` de type `*MyInput` en paramètre. Comme dit précédemmment cette struct sera automatiquement constuite par Tonic. Mon handler retourne ici une struct de type `*MyOutput` ou une erreur: contrairement aux handlers de base de Gin je n’ai pas à gérer la conversion de ma struct en JSON, et je retourner des erreurs facilement (on en reparlera).
    

L’ajout d’un handler Tonic à un router Gin se fait de cette façon:

```
r.GET("/hello/:name", tonic.Handler(MyHandler, 200))
```

`topic.Handler` prend en paramètre mon handler et un code HTTP qui sera le code retourné si le handler ne retourne pas d’erreurs.

### Gestion d’erreurs

Nos handlers Tonic peuvent donc retourner des erreurs. Par défaut ces erreurs passent dans un `hook` interne à Tonic permettant de convertir l’erreur en réponse HTTP. Tonic permet de personnaliser ce hook (qui n’est pas très intéressant par défaut), et c’est ce que nous allons faire ici.

Mais réfléchissons tout d’abord à notre cahier des charges. Lorsque je fais de la gestion d’erreur sur HTTP je veux notamment:

- Pouvoir spécifier un ou des messages d’erreurs à l’utilisateur.
    
- Pouvoir fournir un code HTTP pertinent en fonction de l’erreur (404 pour `not found`, 403 pour `forbidden`…​)
    
- Savoir si je peux exposer ou non le message de l’erreur à l’utilisateur. Il est en effet facile de se tromper et de retourner à l’utilisateur final des erreurs (stacktraces, messages…​) internes. Je souhaite pouvoir spécifier si les messages peuvent être exposés et retourner un message d’erreur par défaut si ce n’est pas le cas.
    

**Type d’erreur**

La première chose que je fais ici est de définir un nouveau type d’erreur:

```
type ErrorType int

const (
	BadRequest ErrorType = iota + 1
	Unauthorized
	Forbidden
	NotFound
	Conflict
	Internal
)

type Error struct {
	Messages  []string
	Cause     error
	Type      ErrorType
	Exposable bool
}
```

Ce type `Error` se compose de plusieurs champs:

- `Messages`: une liste de messages associés à cette erreur
    
- `Cause`: une autre erreur ayant potentiellement causée cette erreur. Cela me permet de garder l’erreur originelle dans mon nouveau type d’erreur.
    
- `Type`: Le type de l’erreur, qui est ici un type généré via `iota`. C’est de cette maière que je pourrai donner du contexte à mon erreur.
    
- `Exposable`: un boolean indiquant si cette erreur est exposable à l’utilisateur ou non.
    

Voici le reste du code pour créer ces erreurs:

```
func (e Error) Error() string {
        msg := strings.Join(e.Messages, " - ")
	if e.Cause != nil {
		msg = fmt.Sprintf("%s - Cause: ", e.Error())
	}
	return msg
}

func New(message string, t ErrorType, exposable bool) Error {
	return Error{
		Messages:  []string{message},
		Type:      t,
		Exposable: exposable,
	}
}

func Newf(message string, t ErrorType, exposable bool, params ...interface{}) Error {
	return Error{
		Messages:  []string{fmt.Sprintf(message, params...)},
		Type:      t,
		Exposable: exposable,
	}
}

func Wrap(e error, message string, t ErrorType, exposable bool) Error {
	return Error{
		Messages:  []string{message},
		Type:      t,
		Exposable: exposable,
		Cause:     e,
	}
}

func Wrapf(e error, message string, t ErrorType, exposable bool, params ...interface{}) Error {
	return Error{
		Messages:  []string{fmt.Sprintf(message, params...)},
		Type:      t,
		Exposable: exposable,
		Cause:     e,
	}
}
```

La fonction `Error()` me convertit mon erreur en `string`, en concaténant les messages de l’erreur et le message de l’erreur `Cause` si elle existe.

Les autres fonctions me permettent de construire facilement de nouvelles erreurs, avec ou sans erreurs originelles.

Imaginons par exemple que mon API HTTP vérifie si un utilisateur existe dans une base de données. Je pourrai construire une erreur de cette manière avec `Newf`:

```
Newf("User %s not found", NotFound, true, name)
```

Cette erreur contient toutes les informations nécessaires pour ensuite être transformée en réponse HTTP: j’ai un message d’erreur clair, son type (not found), je sais qu’elle est `exposable` à l’utilisateur.

`Wrap` et `Wrapf` peuvent s’utiliser de la même manière mais permettent de construire une erreur utilisant mon nouveau type depuis une autre erreur, sans perdre cette dernière. Cela est intéressant: je pourrai comme cela retourner une réponse HTTP correcte à mon utilisateur final mais quand même log par exemple l’erreur originelle qui n’est pas perdue.

**Hook Tonic**

Le code du hook est assez long et je l’ai donc mis en fin d’article. Il n’est pas très propre et n’est qu’un proof of concept mais il fonctionne à peu près.

La fonction `ErrorHook` (qui est mon hook) reçoit mon erreur et se charge de retourner une réponse HTTP où le body sera la représentation JSON de `ErrorResponse`.  
Cette fonction (qui retourne une autre fonction, le but étant ici de pouvoir injecter mon logger Zap dans mon hook final) va donc:

- Vérifier si mon erreur est de type `Error` (mon type d’erreur maison). Si oui, je regardes dans la map `HTTPCodes` à quel code HTTP correspond mon type d’erreur, et si mon erreur est `exposable` je retourne les messages d’erreurs (le champ `Messages`) Sinon, je retournerai un message d’erreur par défaut
    
- Je fais ensuite pas mal de magie pour sortir des messages d’erreurs pertinents lors de la validation de payloads. Je ne vais pas rentrer dans les détails ici (j’ai la flemme pour être honnête) mais le code n’est pas très dur à comprendre (mais m’a demandé un peu de lecture du code de Tonic pour comprendre comment gérer certains cas). Il peut également être amélioré, comme dit précédemment c’est un POC pour l’instant.
    

La fonction `DefaultBindingHookMaxBodyBytes` est un peu spéciale. C’est du code que j’ai repris de Tonic et que j’ai légèrement adapté car Tonic a par défaut un gros problème: en cas d’erreur durant la désérialisation de payloads en JSON l’erreur originelle produite par Golang est perdue et il n’est plus possible de retrouver quelle est la cause exacte de l’erreur.

J’utilise ensuite tout ça de cette manière pour intégrer Tonic avec Gin:

```
gin.SetMode(gin.ReleaseMode)
router := gin.New()
tonic.SetErrorHook(ErrorHook(logger))
tonic.SetBindHook(DefaultBindingHookMaxBodyBytes(tonic.DefaultMaxBodyBytes))
tonic.RegisterTagNameFunc(func(fld reflect.StructField) string {
		name := strings.SplitN(fld.Tag.Get("json"), ",", 2)[0]
		if name == "-" {
			return ""
		}
		return name
	})
router.Use(ginzap.Ginzap(logger, time.RFC3339, true))
router.Use(ginzap.RecoveryWithZap(logger, true))
```

`RegisterTagNameFunc` est également nécessaire pour faire comprendre les tags Golang `json` ajoutés sur vos structs à Tonic.

Voici par exemple quelques exemples de message d’erreurs sur un endpoint de création d’utilisateurs sur lequel je travaille pour un projet personnel:

```
// Passage d'un email invalide dans un payload représentant un utilisateur. CreateOrganizationInput est le type Golang (et donc le type OpenAPI) lié à cet endpoint, comme expliqué dans le chapitre suivant sur OpenAPI.
// Ici le payload ressemblait à quelque chose comme {"account": {"email": "invalid"}}, d'où le chemin CreateOrganizationInput.account.email qui me permet de savoir exactement où est le problème.
{"messages":["Invalid field email (path CreateOrganizationInput.account.email)"]}

// JSON invalide
{"messages":["Invalid parameters"]}

// Multiple erreurs de validation
{"messages":["Invalid field password (path CreateOrganizationInput.account.password)","Invalid field email (path CreateOrganizationInput.account.email)"]}
```

### OpenAPI avec Fizz

Parlons maintenant rapidement de [Fizz](https://github.com/wI2L/fizz/). Ce projet s’intègre avec Gin et Tonic et permet de générer automatiquement la spec [OpenAPI](https://www.openapis.org/) depuis les informations fournies par Tonic.

OpenAPI est un standard très intéressant, permettant de spécifier une API et générer automatiquement de la documentation ou des clients HTTP depuis cette spec. On trouve généralement deux façons de faire de l’OpenAPI aujourd’hui:

- Ecrire la spec OpenAPI (en YAML) manuellement puis générer le code de l’API (serveur HTTP) depuis la spec. Cela fonctionne généralement (il y a d’ailleurs un [générateur Gin](https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators/go-gin-server.md) de disponible que je n’ai jamais essayé) mais c’est très pénible: la spec OpenAPI est très verbeuse et selon moi impossible à écrire et maintenir correctement sur le long terme par des humains.
    
- Ecrire le code puis générer la spec. C’est ce que permet Fizz. De nombreux autres projets (comme Kubernetes par exemple) utilisent cette approche. Certaines personnes diront qu’écrire le code avant la spec est une erreur, mais comme dit précédemment la spec OpenAPI n’est pas faite pour être écrite manuellement.  
    De plus, il est possible de n’écrire que les types et routes de son code (et non son implémentation) pour générer la spec, ce qui limite le problème.
    

Je n’aborde pas ici la solution "j’écris la spec OpenAPI manuellement puis mon code à côté sans génération" car au final on pert complètement l’intérêt d’OpenAPI avec cette approche en plus d’avoir toutes les chances d’avoir une différence entre la spec et l’implémentation finale.  
Certaines entreprises ont aussi des solutions custom pour générer de l’OpenAPI mais ce n’est pas non plus le sujet de cet article.

Reparlons de Fizz. Son [README](https://github.com/wI2L/fizz/#readme) est clair donc je ne vais pas m’étendre dessus. En gros Fizz génère automatiquement la spec OpenAPI depuis les types passés aux handlers `Tonic`, et permet également d’ajouter des informations OpenAPI par endpoint (via `fizz.OperationOption`) si besoin.

Petite subtilité: par défaut Fizz va préfixer vos types OpenAPI par le nom du package Golang où se trouve les types de vos handlers Tonic. Vous pouvez faire `fizzInstance.Generator().OverrideTypeName(reflect.TypeOf(MyType{}), "MyType")` pour override le nom d’un type (le type Golang `MyType{}` s’appellera `MyType` donc sans préfixe dans OpenAPI).