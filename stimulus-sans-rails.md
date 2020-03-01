Une intro au framework StimulusJS.

## Stimulus JS: qu'est-ce que c'est ?

Stimulus a été créé par [Basecamp](https://basecamp.com/). C'est un framework de front, plus simple à prendre en main qu'Angular, React ou Vue. Il n'a pas conçu pour rendre du HTML, mais pour l'augmenter.<br/>
StimulusJS fonctionne selon le principe de __separation of concerns__: on segmente l'application en plusieurs parties afin que chacune d’entre elles isole et gère un aspect précis. De la même façon qu'on sépare le contenu de la présentation avec le CSS, Stimulus permet de séparer le contenu du comportement. Cet arrangement aide à construire des composants réutilisables.<br/>

## Un premier exemple

Nous allons commencer par un exemple simple d'implémentation :)<br/>
Nous allons nous servir de **glitch** qui permet de créer et voir une application en ligne sans passer par beaucoup d'installations en local.<br/>
Stimulus a mis à disposition une application simple pour tester ses fonctionnalités, que vous pouvez ouvrir [ici](https://glitch.com/edit/#!/flaxen-screeching-sole?path=README.md:1:0)<br/>
Si vous cliquez sur "Show" en haut puis "Next to the code" vous allez voir à droite du code une petite fenêtre où vous pouvez visualiser votre site.<br/>
Stimulus a déjà mis un petit exemple, mais nous allons reprendre depuis le début ! <br/>


### Avant de commencer

Dans __public/index.html__, vous allez trouver les lignes suivantes: 

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <link rel="stylesheet" href="main.css">
  <script src="bundle.js" async></script>
</head>
<body>
  <h1 data-controller="example"></h1>
</body>
</html>
```

Vous allez enlever le contenu déjà présent dans le `body` comme ceci: 

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <link rel="stylesheet" href="main.css">
  <script src="bundle.js" async></script>
</head>
<body>

</body>
</html>
```

/!\ À partir de maintenant, tous les éléments html devront être écrits entre les balises <body></body> !

### Hello, world !

Nous allons tester pour la première fois l'implémentation de StimulusJS dans votre application :tada: <br/>


1. Un petit bonjour

Dans __public/index.html__ nous allons commencer par du HTML simple :
```html
<div>
  <input type="text">
  <button>Greet</button>
</div>
```
Dans l'application vous trouverez un premier controller Stimulus __src/controllers/example_controller.js__ donc nous allons nous servir.<br/>
Modifiez-le pour qu'il ressemble à ceci:
```js
import { Controller } from "stimulus"

export default class extends Controller {
}
```
Le principe même de Stimulus est de connecter automatiquement les événements DOM aux controllers Stimulus qui sont des objets JavaScript. <br/>
Maintenant nous allons placer un identifiant *data-controller* à notre div.
```html
<div data-controller="example">
  <input type="text">
  <button>Greet</button>
</div>
``` 
Stimulus fonctionne en monitorant la page, attendant que l'attribut 
```
data-controller
```
apparaisse. <br/>
*data-controller* connecte et déconnecte les controllers Stimulus. Vous pouvez vous le représenter comme ça: *class* fait le lien entre HTML et CSS, *data-controller* fait le lien entre HTML et JavaScript.<br/>
Pour vérifier que votre HTML est bien connecté à votre controller Stimulus nous allons ajouter ces lignes au controller:
```js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```
 
C'est le moment de le voir en action !<br/>
`rails s`<br/>
Rendez-vous sur `http://localhost:3000/` et ouvrez votre console :)<br/>


2. Une action qui répond aux évenements DOM

Pour connecter notre bouton à notre méthode, nous allons utiliser l'attribut *data-action* comme ceci dans notre HTML:
```html
<div data-controller="example">
  <input type="text">
  <button data-action="click->example#greet">Greet</button>
</div>
```
```html
data-action
```
décrit comment les événements de la page doivent déclencher les méthodes de controllers. <br/>
Sa valeur *click->example#greet* est appelée un __*action descriptor*__.<br/>
Ici ce descriptor indique que "click" est le nom de de l'événement, "example" est le nom du controller et "greet" le nom de la méthode à déclencher.<br/>
Rechargez votre page et cliquez sur le bouton, n'oubliez pas de regarder dans votre console !</br>

3. Hello, you <3
Nous allons maintenant faire en sorte que notre action permette de dire bonjour au nom rentré dans l'input.<br/>
Nous allons modifier notre HTML comme ceci:
```html
<div data-controller="example">
  <input data-target="example.name" type="text">
  <button data-action="click->example#greet">Greet</button>
</div>
```

```html
data-target
```
permet de marquer les éléments importants comme *target* pour qu'on puisse faciliment les référencer dans le controller.<br/>
Sa valeur *example.name* est appelée un *__target descriptor__*.<br/>
Ici ce descriptor indique que *example* est le controller et que *name* est le *name* de la *target*.<br/>
On modifie ainsi notre controller:

```javascript
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name" ]

  greet() {
    const element = this.nameTarget
    const name = element.value
    console.log(`Hello, ${name}!`)
  }
}
```
Vous avez pu voir que nous avons ajouté la ligne :
```
static targets = [ "name" ]
```
Pour chaque target présente dans l'array, Stimulus ajoute trois nouvelles propriétés à votre controller:
- *this.nameTarget*: renvoie le premier élément *target* qui correspond au nom présent dans le scope du controller et dont on peut alors récupérer la valeur. S'il n'y en a pas ça renvoie une erreur;
- *this.nameTargets*: renvoie un array d'éléments correspondants dans le scope du controller;
- *this.hasnameTarget*: renvoie true ou false selon s'il y a une target présente ou non

Rechargez la page, cliquez sur le bouton et regardez votre console.

4. Un form complet
Nous allons finir en faisant en sorte que le message apparaisse sur la page et non dans notre console !<br/>
Pour cela on va ajouter une div où on veut que notre nom apparaisse, avec une *data-target* avec comme targe descriptor *example.output*.

```html
<div data-controller="example">
  <input data-target="example.name" type="text">

  <button data-action="click->example#greet">Greet</button>

  <div data-target="example.output"></div>
</div>
```

On va enfin modifier notre controller comme ceci:
```javascript
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name", "output" ]

  greet() {
    const element = this.nameTarget
    const name = element.value
    this.outputTarget.textContent =
      `Hello, ${name}!`
  }
}
```

Rechargez la page et cliquez sur le bouton.

5. Bonus: un peu de refactoring, déjà !

Nous pouvons nettoyer un peu notre méthode *greet()* en extrayant la récupération de la valeur de *name*.  

```js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name", "output" ]

  greet() {
        this.outputTarget.textContent =
      `Hello, ${this.name}!`
  }

  get name() {
    return this.nameTarget.value
  }
}
```


## Pour approfondir 

Quelques liens utiles si vous voulez creuser le sujet: 
- Les [origines](https://stimulusjs.org/handbook/origin) de Stimulus
- [resources](https://stimulusjs.org/reference/controllers): des définitions plus précises des éléments et méthodes
