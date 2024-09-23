# Getting Started

## Introduction

Edgewire allows you to build reactive interfaces without leaving your server using Adonis. It is heavily inspired by the PHP library [Livewire](https://livewire.laravel.com/docs/quickstart) and even share some code.

## Installation

Start by installing the `edgewire` library.

```bash
node ace add @foadonis/edgewire
```

:::disclosure{title="See steps performed by the add command"}

1. Installs the `@foadonis/edgewire` package using the detected package manager.

2. Registers the following service provider and commands inside the `adonisrc.ts` file.

    ```ts
    {
      commands: [
        // ...other providers
        () => import('@foadonis/edgewire/commands')
      ],
      providers: [
        // ...other providers
        () => import('@foadonis/edgewire/providers/shopkeeper')
      ]
    }
    ```

3. Creates the `config/edgewire.ts` file.

:::

### Include scripts in your layout

In order to work, Edgewire requires [Livewire](https://livewire.laravel.com/) Javascript library and [Alpinejs](https://alpinejs.dev/).

You can the `@edgewireScripts` Edge tag at the end of your body in your root layout to load the scripts.

```html
// title: resources/views/layouts/app.edge
</head>
<body>
  ...

  @edgewireScripts
</body>
```

## Create your first component

Edgewire provides different commands to quickly generate new components. Run the following command to make a new `Counter` component:

```bash
node ace make:edgewire counter
```

:::disclosure{title="See steps performed by the add command"}

1. Generates an empty component `app/components/counter.ts`

2. Generates an empty template `resources/views/edgewire/counter.edge`

3. Registers the component in `start/view.ts`

:::

Open `app/components/counter.ts` and replace its content with the following:

```ts
// title: app/components/counter.ts
import { Component, view } from 'edgewire'

export default class CounterComponent extends Component {
  count = 1

  increment() {
    this.count++
    console.log('Increment', count)
  }

  decrement() {
    this.count--
    console.log('Decrement', count)
  }

  // This is optional as the template defaults to the component name.
  render() {
    return view('edgewire/counter')
  }
}
```

The above code:

- Declares a property called `count`. As it is public, it will be available in your template.
- Declares two methods `increment` and `decrement`. As they are public, they can be triggered directly from the browser.
- Defines how to render the component using the `render` method.

### Writing the View

Open the template `resources/view/edgewire/counter.edge` and replace its content with the following:

```html
// title: resources/view/edgewire/counter.edge
<div>
  <h1>{{ count }}</h1>

  <button wire:click="increment">+</button>

  <button wire:click="decrement">-</button>
</div>
```

This code will display the value of the `count` property and two buttons that increment and decrement the `count` property, respectively.

### Test it out

You can now display your component using the `@edgewire` tag:

```html
<body>
  @!edgewire("counter")
</body>
```

After clicking one of the buttons, you will notice that the count updates in real time.

Not impressive if we know React. But if you check your server and browser logs, you will discover that all the business logic inside our component is running by the server and not the client!

After clicking one of the buttons, you will notice that the count updates in real time.

Not impressive if we know React. But if you check your server and browser logs, you will discover that all the business logic inside our component is running by the server and not the client!
