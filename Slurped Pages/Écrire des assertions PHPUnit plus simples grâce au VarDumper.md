---
link: https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper
byline: Grégoire Pineau
site: JoliCode
date: 2024-11-20T00:00
excerpt: Les tests unitaires sont une étape cruciale pour garantir la qualité de
  votre code, mais parfois, les répétitions peuvent devenir lassantes. Avez-vous
  déjà soupiré en enchaînant des appels à $this->assertXXX() pour valider des
  structures complexes ? Heureusement, il existe une
twitter: https://twitter.com/JoliCode
slurped: 2024-12-22T10:54
title: Écrire des assertions PHPUnit plus simples grâce au VarDumper
---

Les tests unitaires sont une étape cruciale pour garantir la qualité de votre code, mais parfois, les répétitions peuvent devenir lassantes. Avez-vous déjà soupiré en enchaînant des appels à `$this->assertXXX()` pour valider des structures complexes ? Heureusement, il existe une solution élégante pour simplifier tout cela : le composant **Symfony/VarDumper**.

Voyons ensemble comment l’utiliser pour rendre vos tests plus concis et lisibles tout en conservant leur efficacité.

## [Section intitulée un-exemple-concret](https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper#un-exemple-concret)Un exemple concret

Imaginons que nous souhaitions tester certaines propriétés d’une classe comme celle-ci :

```
final readonly class MyObject
{
    public function __construct(
        public string $aPublicProperty = 'a public property',
        protected string $aProtectedProperty = 'a protected property',
        private string $aPrivateProperty = 'a private property',

        public string $anotherPublicProperty = 'another public property',
        protected string $anotherProtectedProperty = 'another protected property',
        private string $anotherPrivateProperty = 'another private property',
    ) {}
}
```

Lorsque vous _dumpez_ un objet instancié avec ses valeurs par défaut, voici le résultat obtenu :

```
PhpunitVarDumper\MyObject^ {#325
  +aPublicProperty: "a public property"
  #aProtectedProperty: "a protected property"
  -aPrivateProperty: "a private property"
  +anotherPublicProperty: "another public property"
  #anotherProtectedProperty: "another protected property"
  -anotherPrivateProperty: "another private property"
}
```

## [Section intitulée simplifier-les-tests](https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper#simplifier-les-tests)Simplifier les tests

Traditionnellement, on utiliserait plusieurs appels à `$this->assertSame()` pour valider ces propriétés. Cependant, grâce à **Symfony/VarDumper**, nous pouvons alléger ce processus en utilisant une assertion sur le dump directement.

Commencez par installer le composant en mode développement :

```
composer require --dev symfony/var-dumper
```

Ensuite, importez le trait `VarDumperTestTrait` et utilisez la méthode `assertDumpEquals()` dans vos tests :

```
use PHPUnit\Framework\TestCase;
use PhpunitVarDumper\MyObject;
use Symfony\Component\VarDumper\Test\VarDumperTestTrait;

final class MyObjectTest extends TestCase
{
    use VarDumperTestTrait;

    public function test()
    {
        $myObject = new MyObject();

        $this->assertDumpEquals(
            <<<'DUMP'
            PhpunitVarDumper\MyObject {
              +aPublicProperty: "a public property"
              #aProtectedProperty: "a protected property"
              -aPrivateProperty: "a private property"
              +anotherPublicProperty: "another public property"
              #anotherProtectedProperty: "another protected property"
              -anotherPrivateProperty: "another private property"
            }
            DUMP,
            $myObject,
        );
    }
}
```

N’est-ce pas plus clair et élégant ? 🎉

## [Section intitulée aller-plus-loin-avec-les-casters](https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper#aller-plus-loin-avec-les-casters)Aller plus loin avec les Casters

Dans certains cas, toutes les propriétés d’un objet ne sont pas pertinentes pour vos tests, ou vous ne souhaitez pas les mettre à jour à chaque modification de la classe. C’est ici que les **casters** entrent en jeu.

Un _caster_ est une fonction de rappel qui personnalise la représentation d’un objet _dumpé_. Elle prend en paramètre :

1. L’objet en cours de _dump_ ;
2. Un tableau représentant l’état de l’objet ;
3. (d’autres paramètres que nous n’utilisons pas ici).

Pour ajouter un _caster_, utilisez la méthode `setUpVarDumper()` dans votre test :

```
final class MyObjectTest extends TestCase
{
    use VarDumperTestTrait;

    protected function setUp(): void
    {
        $this->setUpVarDumper(
            [
                MyObject::class => static function (MyObject $myObject, array $a): array {
                    return $a;
                },
            ],
        );
    }
}
```

Pour le moment, notre _caster_ ne sert à rien. Observons donc ce que contient la variable `$a` :

```
^ array:6 [
  "aPublicProperty" => "a public property"
  "\x00*\x00aProtectedProperty" => "a protected property"
  "\x00PhpunitVarDumper\MyObject\x00aPrivateProperty" => "a private property"
  "anotherPublicProperty" => "another public property"
  "\x00*\x00anotherProtectedProperty" => "another protected property"
  "\x00PhpunitVarDumper\MyObject\x00anotherPrivateProperty" => "another private property"
]
```

Si vous avez déjà joué avec PHP et ses mécanismes de sérialisation, vous ne devriez pas être perdu. Sinon une petite explication s’impose :

- **Chaque clé représente une propriété de l’objet.**
- Les notations varient en fonction de la visibilité de la propriété :
    1. Pour une propriété **publique**, son nom apparaît directement en clair, comme `"aPublicProperty"`.
    2. Pour une propriété **protégée**, son nom est précédé par le préfixe spécial `\x00*\x00`, qui signale sa visibilité restreinte.
    3. Pour une propriété **privée**, le préfixe indique son contexte de déclaration : `\x00[NomDeLaClasse]\x00`. Ici, cela donne par exemple `\x00PhpunitVarDumper\MyObject\x00aPrivateProperty`.

Ces notations un peu cryptiques peuvent sembler complexes à première vue. Heureusement, Symfony fournit des **constantes** pour les manipuler plus facilement, comme `Caster::PREFIX_PROTECTED` ou `Caster::PATTERN_PRIVATE`. Utilisons-les pour simplifier notre code de _caster_ :

```
use Symfony\Component\VarDumper\Caster\Caster;

protected function setUp(): void
{
    $this->setUpVarDumper(
        [
            MyObject::class => static function (MyObject $myObject, array $a) {
                unset(
                    $a['anotherPublicProperty'],
                    $a[Caster::PREFIX_PROTECTED.'anotherProtectedProperty'],
                    $a[sprintf(Caster::PATTERN_PRIVATE, MyObject::class, 'anotherPrivateProperty')],
                );

                return $a;
            },
        ],
        AbstractDumper::DUMP_LIGHT_ARRAY,
    );
}
```

Ce _caster_ permet de supprimer les propriétés non pertinentes. Le test devient alors :

```
public function test()
{
    $myObject = new MyObject();

    $this->assertDumpEquals(
        <<<'DUMP'
        PhpunitVarDumper\MyObject {
            +aPublicProperty: "a public property"
            #aProtectedProperty: "a protected property"
            -aPrivateProperty: "a private property"
        }
        DUMP,
        $myObject,
    );
}
```

En cas d’erreur, PHPUnit affiche un diff très lisible, ce qui facilite le debug :

```
1) PhpunitVarDumper\Tests\MyObjectTest::test
Failed asserting that two strings are identical.
--- Expected
+++ Actual
@@ @@
 'PhpunitVarDumper\MyObject {
-  +aPublicProperty: "Hello you!"
+  +aPublicProperty: "a public property"
   #aProtectedProperty: "a protected property"
   -aPrivateProperty: "a private property"
 }'
```

## [Section intitulée quelques-astuces-supplementaires](https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper#quelques-astuces-supplementaires)Quelques astuces supplémentaires

1. **Affichage léger des tableaux** Utilisez `AbstractDumper::DUMP_LIGHT_ARRAY` comme deuxième paramètre de `setUpVarDumper()` pour des tableaux plus compacts.
    
    ```
    -array:3 [
    -  0 => "a"
    -  1 => "b"
    -  2 => "c"
    +[
    +  "a"
    +  "b"
    +  "c"
     ]
    ```
    
2. **Gestion des valeurs dynamiques** Préférez un _caster_ à `assertDumpMatchesFormat()`, car les diff générés avec des formats sont souvent difficiles à lire.
    

## [Section intitulée conclusion](https://jolicode.com/blog/ecrire-des-assertions-phpunit-plus-simples-grace-au-vardumper#conclusion)Conclusion

Le composant **VarDumper** est un allié précieux, tant pour le débogage que pour simplifier vos tests. Grâce à lui, vous gagnez en clarté et en efficacité.

Alors, dites adieu aux assertions fastidieuses, et concentrez-vous sur ce qui compte vraiment : la qualité de votre code. 🎉