# Wakanda Angular App (without the Studio)
# TodoMVC example

---

## Yeoman installation 

```bash
npm install -g yo
```

## Wakanda Project Generator installation

```bash
npm install -g generator-wakanda-project
yo wakanda-project Demo
```

Give the proper path to your wakanda server.

More details here: https://github.com/AMorgaut/generator-wakanda-project

In this example, the project is called "Demo" and the following files are generated:

```
MyApp
  |- Demo
     |- ...
  |- Demo Solution
     |- ...
  |- Gruntfile.js
  |- node_modules
  |- package.json
```

### Git configuration

Add a `MyApp\.gitignore`:

```
node_modules/

# Wakanda logs
*.waLog
*/Logs
studio_log*

# Wakanda User and Group cache
*.cacheUAG
```

And create a new repository:
```
git init
git add -i
git commit -m "Init commit"
```

### Model edition

Open `Demo/Mode.waModel` and add for example the following "Todo" application model:

```json
"dataClasses": [
{
  "name": "Todo",
  "className": "Todo",
  "collectionName": "TodoCollection",
  "scope": "public",
  "attributes": [
  {
    "name": "ID",
    "kind": "storage",
    "scope": "public",
    "unique": true,
    "autosequence": true,
    "type": "long",
    "primKey": true
    },
    {
    "name": "title",
    "kind": "storage",
    "scope": "public",
    "type": "string"
    },
    {
    "name": "completed",
    "kind": "storage",
    "scope": "public",
    "type": "bool"
  }
  ]
}
]
```

## Wakanda-Angular Application

Install Ruby, for example with brew, then install compass:

```bash
brew install ruby
sudo gem install compass
```

Generate your angular-wakanda app in `MyApp\Demo` project folder:

```bash
yo angular-wakanda
```

The following files are generated:
```
MyApp
 |- Demo
    |- Gruntfile.js
    |- angularApp
       |- ...
    |- package.json
    |- parsingExceptions.json
```

Note that it will update `MyApp\Demo\.gitignore` to ignore `dist` files.

### Git configuration

Add a git repository in `MyApp\Demo\`

```bash
git init
```

To ignore your angularApp files from the root git repository, update `MyApp\.gitignore` file:

```
# Angular App
Demo/Gruntfile.js
Demo/angularApp/
Demo/package.json
Demo/parsingExceptions.json
```


## Starting the angular app

To try this app, make sure to start first the Wakanda server.

From `MyApp/`:

```bash
grunt serve
```

Then start the angular app (configured to connect to Wakanda server default port)

From `MyApp/Demo/angularapp/`:

```bash
grunt serve
```

## Creating a TodoMVC with Wakanda & Angular

For example, add the following JS code in `MyApp\Demo\angularApp\app\scripts\controllers\main.js`:

```js
angular.module('Demo')
  .controller('MainCtrl', function ($scope, $wakanda, filterFilter) {
    var ds = $wakanda.getDatastore();
    var todos = $scope.todos = ds.Todo.$find();

    $scope.$watch('todos', function () {
      $scope.remainingCount = filterFilter(todos, { completed: false }).length;
      $scope.doneCount = todos.length - $scope.remainingCount;
      $scope.allChecked = !$scope.remainingCount;
    }, true);

    $scope.addTodo = function () {
      var newTodoEntity;
      var newTodo = $scope.newTodo.trim();
      if (!newTodo.length) {
        return;
      }

      newTodoEntity = $wakanda.$ds.Todo.$create({
        title: newTodo,
        completed: false
      });
      todos.push(newTodoEntity);
      newTodoEntity.$save();

      $scope.newTodo = '';
    };

    $scope.doneEditing = function (todo) {
      $scope.editedTodo = null;
      todo.title = todo.title.trim();

      if (!todo.title) {
        $scope.removeTodo(todo);
      }
    };


    $scope.removeTodo = function (todo) {
      todo.$remove();
      todos.splice(todos.indexOf(todo), 1);
    };


    $scope.clearDoneTodos = function () {
      $scope.todos = todos = todos.filter(function (val) {
        return !val.completed;
      });
    };


    $scope.markAll = function (done) {
      todos.forEach(function (todo) {
        todo.completed = done;
      });
    };

});
```
And update your router to initialize a proxy to the Wakanda datastore. In `MyApp\Demo\angularApp\app\app.js`

```
angular
  .module('Demo', [
    'ngResource',
    'ngRoute',
    'wakanda'
  ])
  .config(function ($routeProvider) {

    var routeResolver = {
      app: ['$wakanda', function($wakanda) {
        return $wakanda.init();
      }]
    };

    $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controller: 'MainCtrl',
        resolve: routeResolver
      })
      .when('/about', {
        templateUrl: 'views/about.html',
        controller: 'AboutCtrl'
      })
      .otherwise({
        redirectTo: '/'
      });
  });

```

You can add `"todomvc-common": "^0.3.1"` in your bower.json to fetch TodoMVC common resources.
Finally, update your main view `MyApp\Demo\angularApp\app\view\main.html`:

```
Tutorial to be continued...
```

### Test your angular app from a Wakanda server

WARNING: it will replace your `MyApp/Demo/WebFolder` content with your angular application. Make sure you don't have any previous work inside.

From `MyApp/Demo/`:

```bash
grunt wakCopy
```

It copies your angular app inside the `MyApp/Demo/WebFolder` directory.

You can now test your app from the Wakanda server.

### Cloud Deployment

First, build your angular application.

From `MyApp/Demo/angularApp`:

```bash
grunt build
```

It builds the app in the `MyApp/Demo/angularApp/dist` folder.

Then, copy the built files back in your Wakanda Project (ie. inside the `MyApp/Demo/WebFolder` directory):

From `MyApp/Demo`:

```bash
grunt wakCopybuild
```

Then you can push your Wakanda Solution on the cloud.
For example, if you have a `mycloud` remote referencing your cloud repository:

From `MyApp/`:

```
git push mycloud master:production
```

