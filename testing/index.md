# Tests

Dans le monde du développement informatique, les methodologies de tests sont sujets à débat.

Ce document a pour but de répondre à la question suivante:

> Quels sont les différents types de tests ? 

## Definition

Trouver des définitions n'est pas chose facile. 

Les pages Wikipédia ne sont elles-mêmes pas correctement sourcées, c'est pour dire !

Nous utilisereons ici les definitions de Atlassian.

Dans chacun des points suivants, nous citerons chacune des définitions, en les détaillants pour éviter toutes confusions.

Lien: [Atlassian - Les différents types de tests logiciels](https://www.atlassian.com/fr/continuous-delivery/software-testing/types-of-software-testing)

### Test unitaire


> Les tests unitaires sont de très bas niveau, près de la source de votre application.
> Ils consistent à tester les méthodes et fonctions individuelles des classes, des composants ou des modules utilisés par votre logiciel.
> Les tests unitaires sont en général assez bon marché à automatiser et peuvent être exécutés très rapidement par un serveur d'intégration continue.

la definition est simple, mais il manque un detail:

 * Un test unitaire doit limiter au maximum l'utilisation de resources externe: Réseaux, File system, Database.

De plus voici une precision pour la partie suivante

> les méthodes et fonctions individuelles des classes, des composants ou des modules utilisés par votre logiciel

En `java` par exemple, il n'est pas question de tester toutes les methodes d'une classe, mais uniquement les methodes `public`.

Il est tout a fait possible d'avoir une classe avec 9 methodes privées et une seule methode public. Dans ce cas *la classe represente le module a tester*.

Toujours en `java`, il est tout a fait possible d'avoir un package, avec une classe `public` qui appelerait d'autres classes `package-private`.

Le point d'entré du test unitaire serait dans ce cas la classe `public`. Dans ce cas *Le package représente le module à tester*

Un test unitaire peut donc englober des fonctionnalités plus ou moins grosse, tant que "l'unité" sous test représente un *module coherent*.

Que ce soit un package, une classe ou une simple fonction il faut toujours considerer: 

* Pourquoi avons-nous besoin de ce nouveau bout de code ? Regle métier, besoin technique, ...
* Quels sont les differents usages de ce nouveau bout de code ? Cas simple, cas limite
* Comment le code est-il destiné à etre appellé ? Interface

Ne pas considérer ces trois questions pendant la création d'un test, peut conduire à des tests couteux dans le temps voire inutiles.

Exemple: Un test unitaire pour une classe `Ticket` qui leve une exception si le prix n'est pas valide lors de la création de l'objet.

### Test d'integration

> Les tests d'intégration vérifient que les différents modules ou services utilisés par votre application fonctionnent bien ensemble.
> Par exemple, ils peuvent tester l'interaction avec la base de données ou s'assurer que les microservices fonctionnent ensemble comme prévu.
> Ces types de tests sont plus coûteux à exécuter, car ils nécessitent que plusieurs parties de l'application soient fonctionnelles.

Ici deux nouvelles choses sont possibles: 

* Il est autorisé de faire des requetes sur le reseaux
* Il est autorisé de tester plusieurs modules d'un coup

Il faut cepandant faire attention au deuxieme point.

Bien qu'il soit aurotisée de tester plusieurs modules d'un coup, je ne le recommande pas personellement (Voir Note 2 dans la partie exemple).

En effet c'est ce point qui cause une incomprehension entre les tests fonctionnels et d'integrations

Ici on souhaite surtout nous assurer que nos interactions se font correctement avec le monde exterieur (DB, API d'un partenaire, File system).

Je recommande, de créer un module dédié à cette dependance externe, et d'effectuer les tests d'integrations necessaire.

Exemple: Un test d'integration qui demarre une image docker MongoDB et qui arrive à faire des operations dessus.

### Test fonctionnel

> Les tests fonctionnels se concentrent sur les exigences métier d'une application.
> Ils vérifient uniquement la sortie d'une action et non les états intermédiaires du système lors de l'exécution de cette action.

Exemple: Un test fonctionnel qui via des appels API, créer un ticket. Dans cet exemple en `java`, on peut utiliser SpringBootTest pour demarer l'application ainsi qu'une base de donnée.

### Exemple Complet

Nous avons besoin une API qui:
  * Valide et sauvegarde des tickets de stationnement en base de donnée.
  * Permet de récuperer un ticket par son identifiant.

Quels sont les tests à mettre en place ?

Proposition:

* Plusieurs test unitaire sur la validation du ticket, isolé de la base de donné et de l'API:
  * Création de l'objet `Ticket`
  * Logique métier: Date
  * Logique métier: Type de vehicule
  * Logique métier: Prix
  * Etc

> Note 1:
> La liste de test unitaire ici est voué à grandir avec le projet

* Deux tests d'integration pour l'interaction avec la base de donnée:
  * Sauvegarde d'une entité en DB
  * Récuperation d'une entité en DB

> Note 2:
> Il est toujours preferable d'avoir un module isolé lorsqu'il s'agit d'interaction externe.
> Ici la communication à la base de donnée doit etre independante à la logique metier.
> Il est donc recommandé de parler "d'entité" et pas de `Ticket` à ce moment la.
> Si ce test était en relation avec les Tickets, cela impliquerait que toutes nos entités en base necessiteraient un test d'integration dédié.
> Les tests d'integration etant couteux, il faut eviter ce scenario.
> On pourra imaginer une interface `Entity` qui sera implementée par la classe `Ticket` afin de pouvoir sauvegarder un ticket.

> Note 3:
> Les tests d'integration sont relativement stable dans le temps, peu de changement sont censé avoir lieux.
> Un test d'integration qui change indique une evolution externe: Maj de la BDD, montée de version de l'API d'un partenaire, etc...

* Trois tests fonctionnels:
  * Un pour la création de tickets via l'api
  * Un pour la recuperation de tickets via l'api
  * Un avec un cas d'erreur basique. Si le systeme est correctement concu, il n'est pas necessairement utile de retester tous les cas d'erreur ici. Les tests unitaires sont la pour cela, et les exception levée par la logique metier des tickets doivent toutes etre traitée de la meme maniere

### Exemple: Evolution

En considerant le point precedant.

Votre chef de projet viens vous voir en demandant une evolution.

```
Nous voulons etre capable, via la meme API, d'aller créer et récupérer un ticket chez un partenaire, en fonction d'une configuration en BDD.
```

Comment tester cette evolution ?

* Test unitaires
  * Creation de `TicketConfiguration`
  * Flag qui represente un partenaire dans `TicketConfiguration`

```
Note: Test relativement simple, voué à grandir...
```

* Test d'integration:
  * Nouveau module `TicketPartner`
  * Requete de creation via `TicketPartner`
  * Requete de recuperation via `TicketPartner`

```
Note: Pas besoin de test pour l'interaction entre `TicketConfiguration` et la BDD, nous avons déjà un test BDD isolé et générique.
```

* Tests fonctionnels:
  * Création d'un ticket avec une configuration partenaire en BDD
  * Recuperation d'un tickets avec une configuration partenaire en BDD

```
Note: Bien que les tests d'integrations et fonctionnels se ressemblent, le test d'integration isole unitement la partie partenaire, la ou le test fonctionnel viens valider la demande métier complete du chef de projet.
```

### Exemple: Folder structure

Nous avons des exemples sur quels tests implementer. Mais comment cela est-il separé dans la base de code ?

```
|
|-- ticket
|   |
|   |-- api
|   |   |-- TicketApi.java              # Point d'entré du test fonctionnel
|   |
|   |-- entity
|   |   |-- Ticket.java                 # Point d'entré pour les tests unitaires
|   |   |-- TicketConfiguration.java    # Point d'entré pour les tests unitaires
|   |
|   |-- partner
|       |-- PartnerA.java               # Point d'entré pour les tests d'integration
|       |-- PartnerB.java               # Point d'entré pour les tests d'integration
|
|-- database
|   |-- MongoDbDatabase.java            # Point d'entré pour les tests d'integration
|
|-- SpringApplication.java
```

