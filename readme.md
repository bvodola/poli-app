# Criação de APPs com Node.js e Ionic
## Instalações Iniciais

- Acessar [ionic.io](http://ionic.io) e criar uma conta
- Acessar [heroku.com](http://heroku.com) e criar uma conta
- Acessar [c9.io](http://c9.io) e criar uma conta
    - Criar um novo workspace, clonando o seguinte repositório https://github.com/bvodola/poli-app.git

## Dentro do Workspace c9.io
Instalar cordova e ionic através dos seguintes comandos no terminal (que fica na aba inferior):

```bash
$ npm install -g cordova
$ npm install -g ionic
```

E então prosseguir com os seguintes comandos
```bash
$ npm install
$ node server
```

Depois disso, selecione **Preview > Preview Running Application**.
Verifique que a aplicação está funcionando corretamente, acessando o endereço
https://appname-username.c9.io/sessions

Obs:
- appname ... nome do app que você escolheu no c9.io
- username ... node de usuário que você escolheu no c9.io

Se tudo estiver funcionando corretamente, o navegador irá exibir o conteúdo abaixo, que nada mais são do que alguns dados fictícios usados para testar o APP.
```json
[{"id":0,"title":"Introduction to Ionic","speaker":"CHRISTOPHE COENRAETS","time":"9:40am","room":"Ballroom A","description":"In this session, you'll learn how to build a native-like mobile application using the Ionic Framework, AngularJS, and Cordova."},{"id":1,"title":"AngularJS in 50 Minutes","speaker":"LISA SMITH","time":"10:10am","room":"Ballroom B","description":"In this session, you'll learn everything you need to know to start building next-gen JavaScript applications using AngularJS."},{"id":2,"title":"Contributing to Apache Cordova","speaker":"JOHN SMITH","time":"11:10am","room":"Ballroom A","description":"In this session, John will tell you all you need to know to start contributing to Apache Cordova and become an Open Source Rock Star."},{"id":3,"title":"Mobile Performance Techniques","speaker":"JESSICA WONG","time":"3:10Pm","room":"Ballroom B","description":"In this session, you will learn performance techniques to speed up your mobile application."},{"id":4,"title":"Building Modular Applications","speaker":"LAURA TAYLOR","time":"2:00pm","room":"Ballroom A","description":"Join Laura to learn different approaches to build modular JavaScript applications."}]
```

## Comentário sobre o Banco de Dados

O Banco de dados já foi criado de antemão e está hospedado na nuvem através do serviço [MongoLab](http://www.mongolab.com). A conexão com o banco de dados deste serviço é feita através da seguinte linha de código presente em **server.js**:

```js
// MongoDB
mongoose.connect('mongodb://poli-app:a1b2c3d4@ds049864.mongolab.com:49864/poli-app');
```

## Criação da API

Crie o diretório **models** dentro do diretório raíz. Agora, dentro de **models**, crie um arquivo chamado **course.js**

Abra o arquivo **course.js** e modifique o conteúdo de  para que fique como abaixo:

```js
// Dependencies
var restful = require('node-restful');
var mongoose = require('mongoose');

// Schema
var courseSchema = new mongoose.Schema({
    name: String,
    code: String,
    classroom: String
});

// Return Model
module.exports = restful.model('Courses', courseSchema);
```

Agora, crie um novo arquivo dentro de **routes** com o nome **api.js**. Abra este arquivo e insira o conteúdo abaixo:
```js
// Dependencies
var express = require('express');
var router = express.Router();

// Models
var Course = require('../models/course');

// Routes
Course.methods(['get', 'put', 'post', 'delete']);
Course.register(router, '/courses');

// Return Router
module.exports = router;
```

Finalmente, agora no arquivo **server.js**, abaixo da seguinte linha:
```js
app.get('/sessions/:id', sessions.findById);
```

insira:
```js
app.use('/api', require('./routes/api'));
```

O arquivo *server.js* completo ficará então assim:

```js
var express = require('express'),
    bodyParser      = require('body-parser'),
    methodOverride  = require('method-override'),
    sessions        = require('./routes/sessions'),
    mongoose        = require('mongoose'),
    app = express();

mongoose.connect('mongodb://poli-app:a1b2c3d4@ds049864.mongolab.com:49864/poli-app');

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
    extended: true
}));
app.use(methodOverride());      // simulate DELETE and PUT

// CORS (Cross-Origin Resource Sharing) headers to support Cross-site HTTP requests
app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");
    next();
});

app.get('/sessions', sessions.findAll);
app.get('/sessions/:id', sessions.findById);
app.use('/api', require('./routes/api'));

app.set('port', process.env.PORT || 5000);

app.listen(app.get('port'), function () {
    console.log('Express server listening on port ' + app.get('port'));
});
```

Executando novamente o comando:
```sh
$ node server
```
E selecionando **Preview > Preview Running Application**, podemos agora testar nossa API de Cursos acessando o endereço https://appname-username.c9.io/api/courses. Inicialmente, esse endereço deve retornar um conjunto vazio, representado por `[]`. Isso é esperado, já que ainda não adicionamos nenhum curso. Vamos fazer isso mais adiante.

### Fazendo o Deploy da API
Execute os seguintes comandos:

```sh
$ git add .
$ git commit -m "Added a Procfile."
$ heroku login
Enter your Heroku credentials.
...
$ heroku create
Creating arcane-lowlands-8408... done, stack is cedar
http://arcane-lowlands-8408.herokuapp.com/ | git@heroku.com:arcane-lowlands-8408.git
Git remote heroku added
$ git push heroku master
```

Guarde o endereço do aplicativo gerado pleo Heroku. Você vai usá-lo em seguida.

## Criando o Aplicativo

### Criando o Service
```sh
$ cd ~/workspace
$ ionic start client sidemenu
...
Create an ionic.io account to send Push Notifications and use the Ionic View app?
(Y/n):n
```

Dentro de **client/www/js/*** crie o arquivo *services.js* e insira o seguinte conteúdo

```js
angular.module('starter.services', ['ngResource'])

.factory('Course', function ($resource) {
    return $resource('https://stark-plateau-9821.herokuapp.com/api/courses/:courseId');
});
```

Onde o endereço acima, que dentro de $resource é igual àquele gerado pelo Heroku, que você deve ter guardado.

Agora, abra o arquivo *index.html* que está em **www** e faça as seguintes modificações:

Depois de:
```html
<!-- ionic/angularjs js -->
<script src="lib/ionic/js/ionic.bundle.js"></script>
```

Insira:
```html
<script src="lib/ionic/js/angular/angular-resource.min.js"></script>
```

E depois de:
```html
<!-- your app's js -->
<script src="js/app.js"></script>
```

Insira:
```html
<script src="js/services.js"></script>
```

### Criando os Controllers

Abra o arquivo **client/www/js/controllers.js**

Modifique a primeira linha para que fique assim:

```js
angular.module('starter.controllers', ['starter.services'])
```
Então, depois do trecho:

```js
.controller('PlaylistCtrl', function($scope, $stateParams) {
});
```

Adicione:
```js
.controller('CoursesCtrl', function($scope, Course) {
    $scope.courses = Course.query();
})

.controller('CourseCtrl', function($scope, $stateParams, Course) {
    $scope.course = Course.get({courseId: $stateParams.courseId});
});
```

### Criando os Templates

Em **client/www/templates***, crie o arquivo **courses.html** e insira o seguinte conteúdo:

```html
<ion-view view-title="Courses">
  <ion-content>
    <ion-list>
      <ion-item ng-repeat="course in courses"
                href="#/app/courses/{{course._id}}">{{course.name}}</ion-item>
    </ion-list>
  </ion-content>
</ion-view>
```

Ainda em **client/www/templates***, crie o arquivo **course.html** (repare que dessa vez a palavra course está no singular) e insira o seguinte conteúdo:

```html
<ion-view view-title="Course">
  <ion-content>
    <div class="list card">
      <div class="item">
        <h3>{{course.name}}</h3>
        <h2>{{course.code}}</h2>
        <p>{{course.classroom}}</p>
      </div>
    </div>
  </ion-content>
</ion-view>
```

### Implementando Rotas

Abra o arquivo **app.js** em **client/www/js** e após o trecho:

```js
    .state('app.playlists', {
      url: '/playlists',
      views: {
        'menuContent': {
          templateUrl: 'templates/playlists.html',
          controller: 'PlaylistsCtrl'
        }
      }
    })
```
Adicione:

```js
.state('app.courses', {
  url: "/courses",
  views: {
      'menuContent': {
          templateUrl: "templates/courses.html",
          controller: 'CoursesCtrl'
      }
  }
})

.state('app.course', {
    url: "/courses/:courseId",
    views: {
        'menuContent': {
          templateUrl: "templates/course.html",
          controller: 'CourseCtrl'
      }
    }
});
```

Agora, abra o menu lateral em **client/www/templates/menu.html**. Abaixo de:

```
<ion-item menu-close href="#/app/playlists">
  Playlists
</ion-item>
```

Adicione:
```
<ion-item menu-close href="#/app/courses">
    Courses
</ion-item>
```

### Testando A Aplicação

No terminal, execute o comando:

```sh
$ ionic serve -p $PORT --nolivereload
```

Para ver seu aplicativo rodando no celular, baixe o Ionic View (procure nas lojas de APPS correspondentes ao seu aparelho). Então execute no terminal:

```sh
$ ionic upload
```

