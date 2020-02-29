# Stimulus JS
Une intro au framework StimulusJS.

## Stimulus JS: qu'est-ce que c'est ?


Stimulus a été créé par [Basecamp](https://basecamp.com/). C'est un framework de front, plus simple à prendre en main qu'Angular, React ou Vue. Il n'a pas conçu pour rendre du HTML, mais pour l'augmenter.<br/>
StimulusJS fonctionne selon le principe de __separation of concerns__: on segmente l'application en plusieurs parties afin que chacune d’entre elles isole et gère un aspect précis. De la même façon qu'on sépare le contenu de la présentation avec le CSS, Stimulus permet de séparer le contenu du comportement. Cet arrangement aide à construire des composants réutilisables.<br/>

## Un premier exemple

Nous allons commencer par un exemple simple d'implémentation :)


### Avant de commencer
Nous allons utiliser **webpack**, qui nécessite une version minimum de Rails 5.1 <br/>
Vous pouvez vérifier votre version de rails en faisant `rails --version`<br/>
Pour installer une version supérieure, faire: `gem install rails -v 5.2.1`<br/>

### Initialisation

`rails new my_app`<br/>
`cd my_app`<br/>

### Installation de yarn

Rendez-vous sur la [page d'installation de yarn](https://classic.yarnpkg.com/fr/docs/install/), choisissez votre version et suivez les instructions.

### Installation de webpacker

##### Si vous utilisez Rails 6 : passez directement à l'étape "Installation de StimulusJS".

Nous allons installer [webpacker](https://github.com/rails/webpacker) qui permet d'ajouter __webpack__ à un projet Rails. Stimulus fonctionne avec webpack pour charger automatiquement les fichiers de controllers dans l'app.<br/>
Ajoutez à votre __Gemfile__:
```
gem 'webpacker'
```
`bundle install`<br/>
`rails webpacker:install`<br/>
Plein de fichiers se sont créés ! Nous allons voir rapidement ce qui s'est ajouté: 
- dans __app__, il y a un nouveau dossier __javascript/packs__. Chaque pack est un module que vous pouvez ajouter dans les views Nous allons voir ça plus bas. 
- __config/webpack__
- dans __config__, webpacker.yml
- babel.config.js, postcss.config.js, yarn.lock, .browserslistrc
- __bin/webpack__, __bin/webpack-dev-server__
  
### Installation de StimulusJS

`rails webpacker:install:stimulus`<br/>
Cette commande va:
- ajouter stimulus à votre fichier __package.json__
- créer un dossier __app/javascript/controllers__ qui va contenir vos controllers
- ajouter l'import des controllers dans votre fichier __app/javascript/application.js__

Ajouter dans le head dans __app/views/layouts/application.html.erb__ la ligne suivante:
```ruby
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload', async: true %>
```

### Hello, world !

Nous allons tester pour la première fois l'implémentation de StimulusJS dans votre application :tada: <br/>
`rails g controller home index`<br/>
Dans __config/routes.rb__, indexez la home page à votre controller home : 
```ruby
root 'home#index'
```

1. Un petit bonjour

Dans __app/views/home/index.html.erb__ nous allons commencer par du HTML simple :
```html
<div>
  <input type="text">
  <button>Greet</button>
</div>
```
Vous allez maintenant créer votre premier controller Stimulus dans le fichier __app/javascript/controllers/hello_controller.js__:
```js
import { Controller } from "stimulus"

export default class extends Controller {
}
```
Le principe même de Stimulus est de connecter automatiquement les événements DOM aux controllers Stimulus qui sont des objets JavaScript. <br/>
Maintenant nous allons placer un identifiant *data-controller* à notre div.
```html
<div data-controller="hello">
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
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```
```html
data-action
```
décrit comment les événements de la page doivent déclencher les méthodes de controllers. <br/>
Sa valeur *click->hello#greet* est appelée un __*action descriptor*__.<br/>
Ici ce descriptor indique que "click" est le nom de de l'événement, "hello" est le nom du controller et "greet" le nom de la méthode à déclencher.<br/>
Rechargez votre page et cliquez sur le bouton, n'oubliez pas de regarder dans votre console !</br>

3. Hello, you <3
Nous allons maintenant faire en sorte que notre action permette de dire bonjour au nom rentré dans l'input.<br/>
Nous allons modifier notre HTML comme ceci:
```html
<div data-controller="hello">
  <input data-target="hello.name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

```html
data-target
```
permet de marquer les éléments importants comme *target* pour qu'on puisse faciliment les référencer dans le controller.<br/>
Sa valeur *hello.name* est appelée un *__target descriptor__*.<br/>
Ici ce descriptor indique que *hello* est le controller et que *name* est le *name* de la *target*.<br/>
Stimulus crée automatiquement une propriété *this.nameTarget* qui renvoie le premier élément *target* qui correspond et dont on peut alors récupérer la valeur.<br/> 
Essayons en modifiant ainsi notre controller:

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

Rechargez la page, cliquez sur le bouton et regardez votre console.

4. Un form complet
Nous allons finir en faisant en sorte que le message apparaisse sur la page et non dans notre console !<br/>
Pour cela on va ajouter une div où on veut que notre nom apparaisse, avec une *data-target* avec comme targe descriptor *hello.output*.

```html
<div data-controller="hello">
  <input data-target="hello.name" type="text">

  <button data-action="click->hello#greet">Greet</button>

  <div data-target="hello.output"></div>
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
