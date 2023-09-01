---
title: Use JavaScript
description: Learn how to use JavaScript with Laravel Orchid to enhance the functionality and interactivity of your administration-style applications. Get tips on incorporating JavaScript libraries and custom scripts, and find out how to debug and optimize your code.
---

The core of the platform in terms of styling is [Bootstrap](http://getbootstrap.com/), and the browser runs [Hotwired](https://hotwired.dev) code. You can connect other libraries to your liking, but we recommend staying in this ecosystem.

## Turbo

Thanks to [Turbo](https://turbo.hotwire.dev), the admin panel emulates the Single Page Application, loading resources only on the first call and giving the impression of re-rendering content in the browser instead of natural standard transitions between pages.


Since all resources will be loaded on the first call, classic calls like this will not work:

```js
document.addEventListener("load", () => {
    console.log('Page load');
});
```

It will be executed only once and will not be called again during transitions. To avoid this, you need to use Turbo events:

```js
document.addEventListener("turbo:load", () => {
    console.log('Page load');
})
```

You can find more details on the website [turbo.hotwire.dev](https://turbo.hotwire.dev).


## Stimulus

[Stimulus](https://stimulus.hotwired.dev/) is a JavaScript framework from the Ruby on Rails developers. It equips frontend development using new approaches to JavaScript, while it does not seek to control all your actions and does not impose a separation of frontend from backend.

Let's build a basic example for a project using **Laravel Mix** (Laravel before versoin 9) that displays the text entered the field for this:

In `resources/js`, create the following structure:

```php
resources
├── js
│   ├── controllers
│   │   └── hello.js
│   └── dashboard.js
├── lang
├── sass
└── views
```

Controller class with the following content:

```php
// hello.js
export default class extends window.Controller {

    static get targets() {
        return [ "name", "output" ]
    }

    greet() {
        this.outputTarget.textContent =
            `Hello, ${this.nameTarget.value}!`
    }
}
```

And the assembly point:

```php
// dashboard.js
import HelloController from "./controllers/hello"

application.register("hello", HelloController);
```

Such a structure will not prevent your application, no matter what kind of front-end to build: Angular/React/Vue, etc.

It remains only to describe the assembly in webpack.mix.js:

```php
let mix = require('laravel-mix');

mix.js('resources/js/dashboard.js', 'public/js')
```

It remains only to connect the received script to the panel in the configuration file or in the service provider using the `registerResource` method. You can do the same with style sheets, which will allow you to effectively build application logic.

```php
// config/platform.php
'resource' => [
    'stylesheets' => [],
    'scripts'     => [
        '/js/dashboard.js'
    ],
],
```

> **Note**. To apply changes to the configuration file, you may need to clear the cache if it was created earlier. It can be done using the artisan command `artisan config:clear`.

An example of a record for a service provider

```php
// app/Providers/AppServiceProvider.php

use Orchid\Platform\Dashboard;

class AppServiceProvider extends ServiceProvider
{
    public function boot(Dashboard $dashboard)
    {
        $dashboard->registerResource('scripts','/js/dashboard.js');
        //$dashboard->registerResource('stylesheets','/css/dashboard.css');
    }
}
```

To achieve the same for a project making use of **Vite** (Laravel ^9.0), the steps differ slightly:

In `public/`, create the following structure:

```php
public
├── js
│   ├── controllers
│   │   └── hello.js
│   └── dashboard.js
├── storage
├── vendor
└── index.php
```

Controller class with the following content:

```php
// hello.js
export default class extends window.Controller {

    static get targets() {
        return [ "name", "output" ]
    }

    greet() {
        this.outputTarget.textContent =
            `Hello, ${this.nameTarget.value}!`
    }
}
```

And the assembly point:

```php
// dashboard.js
import HelloController from "./controllers/hello"

application.register("hello", HelloController);
```

It remains only to connect the received script to the panel in the configuration file. This handles the necessary module injection.

```php
// config/platform.php
'vite' => [
        'public/js/dashboard.js',
   ],
```

> **Note**. To apply changes to the configuration file, you may need to clear the cache if it was created earlier. It can be done using the artisan command `artisan config:clear`.


To display, we will use a template for which you first need to define the `Controller` and `Route` in your application:

```php
// hello.blade.php
<div data-controller="hello">
  <input data-hello-target="name" type="text">

  <button data-action="click->hello#greet">
    Greet
  </button>

  <span data-hello-target="output">
  </span>
</div>
```



## Vue.js Wrapped in a Stimulus


Many developers love the simplicity and power of Vue.js for building interactive and responsive user interfaces.  In this tutorial, we'll show you how to wrap Vue components within a Stimulus controller so you can easily integrate them.


Create a Stimulus controller file, for example `hello_controller.js`:

```js
import {createApp} from 'vue';

export default class extends window.Controller {
    connect() {
        this.app = createApp({
            data() {
                return {
                    message: 'Hello, Vue.js!'
                }
            }
        });

        this.app.mount(this.element);
    }

    disconnect() {
        this.app.unmount();
    }
}

```

Register the controller in your blade file:

```html
<div data-controller="hello">
  @{{ message }}
</div>
```

Now, when the page loads, the Vue.js instance will be created and the message will be displayed in the HTML element. You can then use Vue.js as you normally would within the scope of the Stimulus controller.
