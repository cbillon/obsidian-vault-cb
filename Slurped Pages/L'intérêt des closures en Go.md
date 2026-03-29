---
link: https://www.metal3d.org/blog/2016/lint%C3%A9r%C3%AAt-des-closures-en-go/
site: Metal3d
date: 2016-03-20T11:58
excerpt: "Vous avez entendu parler des “closures” et “generators” qui sont implémentés par plusieurs langages (comme le Python). Mais je sais aussi que beaucoup n’ont pas conscience de l’intérêt particulier de ce pattern. Et comme je me suis retrouvé dans une situation où un generator m’a fait gagné beaucoup de temps, je vais vous montrer à quoi ça sert avec deux exemples: un qui explique le fonctionnement, et un autre qui peut vous rendre service, à savoir “itérer indéfiniement”."
tags:
  - golang
slurped: 2026-03-28T10:13
title: L'intérêt des closures en Go
---

Vous avez entendu parler des “closures” et “generators” qui sont implémentés par plusieurs langages (comme le Python). Mais je sais aussi que beaucoup n’ont pas conscience de l’intérêt particulier de ce pattern. Et comme je me suis retrouvé dans une situation où un generator m’a fait gagné beaucoup de temps, je vais vous montrer à quoi ça sert avec deux exemples: un qui explique le fonctionnement, et un autre qui peut vous rendre service, à savoir “itérer indéfiniement”.

Une closure est une fonction. Si si, je vous assure ce n’est ni plus ni moins que cela. A la différence près que la closure peut garder son état en mémoire. Mais vous allez me dire que quand on a appelé une fonction et qu’on a traité la valeur la de retour alors on perd son état. Oui, effectivement. Sauf dans le cas où ce que vous retournez est une fonction.

Le premier exemple est celui que je donne souvent pour montrer ce qu’est une closure de type “generator”. Elle va initialiser un compteur et retourner une fonction qui permet d’incrémenter ce dernier, et le retourner.

```
// Counter retourne une fonction qui retourne un entier
func Counter() func() int {
    count := 0
    return func() int{
        count++
        return count
    }
}
```

Voilà, on va s’en servir de cette manière:

```
c := Counter()
fmt.Println(c())
fmt.Println(c())
fmt.Println(c())
```

A chaque appel de `"c()"`, la variable “counter” est incrémentée et on l’affiche. L’intérêt est aussi de pouvoir avoir plusieurs compteurs:

```
c1 := Counter()
c2 := Counter()
fmt.Println(c1())
fmt.Println(c2())
fmt.Println(c1())
fmt.Println(c2())
fmt.Println(c2())
```

Cela va afficher:

```
1
1
2
2
3
```

Bien. C’est un générateur simple, pas super utile en soi mais qui a le mérite de vous montrer le principe. Passons à l’exemple qui, lui, va vous permettre de réalier un générateur plus intéressant.

Imaginons que nous ayons un “map” dont les clefs sont des chaines. Et imaginons que nous voulions itérer sur les clefs indéfiniement. Quand nous atteignons la fin de la liste, il faut revenir au début. J’explique:

```
items := map[string]int{
    "A" : 0
    "B" : 0
    "C" : 0
}
```

Je veux donc être capable d’itérer en ayant, coup par coup:

```
A
B
C
A
B
C
...
```

Il existe plusieurs façons de faire, mais celle que j’apprécie est le generator. Je vous montre étape par étape.

L’algo de rotation infini est à peu près:

```
for {
    for key, _ := range items {
        // faire un truc avec key
    }
}
```

Vous avez compris, je fais une boucle “for” sans sortie et une autre qui itère sur les “items”. Ainsi, quand la boucle interne qui itère sur les clefs a terminé, et bien on recommence.

Mais il faut “faire un truc” avec la clef. Si j’assigne indéfiniement, je vais faire boucler ma fonction comme un bourrin et à l’arrivée on sera pas plus avancée. L’idée pas trop débile est donc d’écrire la valeur dans un canal qui va bloquer tant qu’on l’a pas lu.

```
// attendez, ça marchera pas encore

func Loop(c chan string){
    for {
        for k, _ := range items {
            c <- k
        }
    }
}

// et plus loin:

c := make(chan string)
go Loop(c)
fmt.Println(<-c)
fmt.Println(<-c)
fmt.Println(<-c)
```

C’est déjà pas mal. Mais c’est pas “élégant”.

Pourquoi je trouve pas ça élégant ? nous devons nous même créer le canal, puis lancer la goroutine (tâche en concurrence) ce qui me chagrine un peu.

On va donc faire autremement. Faire en sorte d’avoir un générateur qui va lancer la goroutine, fournir les valeurs au canal et permettre d’avoir la valeur suivante à chaque appel.

```

// generateur
func Loop() func() string {
    // le canal qui prend les valeurs
    c:=make(chan string)
    
    // on lance ça en tache de fond
    go func(){
        for {
            for k, _ := range items {
                c <- k
            }
        }
    }()
    
    // on retourne une fonction qui retourne
    // la valeur lue depuis le canal
    return func() string {
        return <-c
    }
}
```

Et on va l’utiliser de cette manière:

```
g := Loop()
fmt.Println(g())
fmt.Println(g())
fmt.Println(g())
fmt.Println(g())
fmt.Println(g())
//...
```

Vous pouvez tester, ça fonctionne très bien. Chaque appel à “`g()`” débloque le canal, la goroutine interne insert alors la prochaine clef du map dans le canal. Si on atteind la fin de la liste, on recommence.

L’exemple se trouve ici: [http://play.golang.org/p/a_xC4l4NoL](http://play.golang.org/p/a_xC4l4NoL)

Une question vous tourmente, je le sais. Pouquoi je retourne pas le canal ? Effectivement:

```
// generateur
func Loop() <-chan string {
    // le canal qui prend les valeurs
    c:=make(chan string)
    
    // on lance ça en tache de fond
    go func(){
        for {
            for k, v := range items {
                c <- k
            }
        }
    }()
    
    // on retourne le canal
    return c
}

// plus loin

c := Loop()
fmt.Println(<-c)
fmt.Println(<-c)
fmt.Println(<-c)
```

Ça marche aussi très bien ! **MAIS !**

Si jamais un jour je me rend compte que j’ai besoin de changer la valeur lue, ou qu’un opération doit être effectué avant que mon programme lise effectivement la prochaine entrée, je suis coincé. Alors qu’avec un generator je sais que je peux manipuler la fonction retournée. Par exemple, si je veux écrire un log lors de la lecture:

```
func Loop() func() string {
    // le canal qui prend les valeurs
    c:=make(chan string)
    
    // on lance ça en tache de fond
    go func(){
        for {
            for k, v := range items {
                c <- k
            }
        }
    }()
    
    // on retourne une fonction qui retourne
    // la valeur lue depuis le canal
    return func() string {
        // je décide de faire un truc avant
        log.Println("Je fais un truc avant")
        return <-c
    }
}
```

Ce n’est pas possible avec l’exemple qui retourne le canal. Oui, on pourrait le faire dans la boucle qui écrit la valeur dans le canal, mais dans ce cas le “truc fait avant” est effectué quand la valeur est insérée dans le canal, et non pas quand on la lit. La différence peut être très importante.

Bref, je vous invite à utiliser les générateurs quand vous le pouvez. C’est un pattern puissant qui trouve sa place en Go, en Python, mais aussi en Javascript…