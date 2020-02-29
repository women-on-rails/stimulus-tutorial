# Stimulus JS
Une intro au framework StimulusJS.

## Stimulus JS: qu'est-ce que c'est ?


Stimulus a été créé par [Basecamp](https://basecamp.com/). C'est un framework de front, plus simple à prendre en main qu'Angular, React ou Vue. Il n'a pas conçu pour rendre du HTML, mais pour l'augmenter. 



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
```
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload', async: true %>
```

### Hello, world !

Nous allons tester pour la première fois l'implémentation de StimulusJS dans votre application :tada: <br/>
`rails g controller home index`<br/>
Dans __config/routes.rb__, indexez la home page à votre controller home : 
```
root 'home#index'
```
Dans __app/views/home/index.html.erb__ vous pouvez ajouter ces lignes:
```
<div data-controller="hello">
  <input data-target="hello.name" type="text">

  <button data-action="click->hello#greet">Greet</button>

  <span data-target="hello.output"></span>
</div>
```
Maintenant, créez votre premier controller :
```
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name", "output" ]

  greet() {
    this.outputTarget.textContent =
      `Hello, ${this.nameTarget.value}!`
  }
}
```
C'est le moment de le voir en action !<br/>
`rails s`
Rendez-vous sur `http://localhost:3000/` et testez votre nouvel input :)
