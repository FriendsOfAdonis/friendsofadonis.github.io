# Components

Components are the building blocks of your Edgewire application. They combine state and behavior to create reusable pieces of UI for your front end. Here, we'll cover the basics of creating and rendering components.

## Creating components

An Edgewire component is simply a class that extends Component. You can create component files by hand or use the following Ace command:

```bash
node ace make:edgewire create-post
```

After running this command, Edgewire will create two new files in your application. The first will be the component's class: app/components/Counter.ts.

```ts
import { Component, view } from 'edgewire'

export default class CreatePost extends Component {
  async render() {
    return view('edgewire/create-post')
  }
}
```

The second will be the component's Edge.js view: resources/views/edgewire/create-post.edge

### Inline components

If your component is fairly small, you may want to create an inline component. Inline components are single-file Edgewire components whose view template is contained directly in the render() method rather than a separate file:

```ts
import { Component, view } from 'edgewire'

export default class CreatePost extends Component {
  async render() {
    return `
        <div>
          <button>Create</button>
        </div>
    `
  }
}
```

### Omitting the render method

To reduce boilerplate in your components, you can omit the render() method entirely and Edgewire will use its own underlying render() method, which returns a view with the conventional name corresponding to your component:

```ts
import { Component, view } from 'edgewire'

export default class CreatePost extends Component {}
```

If the component above is rendered on a page, Edgewire will automatically determine it should be rendered using the edgewire/create-post template.

## Setting properties

Edgewire components have properties that store data and can be easily accessed within the component's class and Edge.js view. This section discusses the basics of adding a property to a component and using it in your application.

To add a property to a Edgewire component, declare a public property in your component class. For example, let's create a `title` property in the `CreatePost` component:

```ts
import { Component, view } from 'edgewire'

export default class CreatePost extends Component {
  title = 'Post title...'

  async render() {
    return view('edgewire/create-post')
  }
}
```

### Accessing properties in the view

Component properties are automatically made available to the component's Edge.js view. You can reference it using standard Edge.js syntax. Here we'll display the value of the `title` property:

```html
<div>
  <h1>Title: {{ title }}</h1>
</div>
```

The rendered output of this component would be:

```html
<div>
  <h1>Title: "Post title..."</h1>
</div>
```

### Sharing additional data with the view

In addition to accessing properties from the view, you can explicitly pass data to the view from the render() method, like you might typically do from a controller. This can be useful when you want to pass additional data without first storing it as a property—because properties have specific performance and security implications.

To pass data to the view in the render() method, you can use the with() method on the view instance. For example, let's say you want to pass the post author's name to the view. In this case, the post's author is the currently authenticated user:

```ts
import { Component, view } from 'edgewire'

export default class CreatePost extends Component {
  title = 'Post title...'

  async render() {
    const user = this.ctx.auth.getUserOrFail()
    return view('edgewire/create-post').with({
      author: user.name,
    })
  }
}
```

Now you may access the `author` property from the component's Edge.js view:

```html
<div>
  <h1>Title: {{ title }}</h1>
  <span>Author: {{ author }}</span>
</div>
```

### Adding wire:key to @each loops

When looping through data in a Edgewire template using `@each`, you must add a unique `wire:key` attribute to the root element rendered by the loop.

Without a `wire:key` attribute present within an Edge.js loop, Edgewire won't be able to properly match old elements to their new positions when the loop changes. This can cause many hard to diagnose issues in your application.

For example, if you are looping through an array of posts, you may set the `wire:key` attribute to the post's ID:

```html
<div>
  @each(post in posts)
    <div wire:key="@{{ post.id }}">
      <h1>{{ post.title }}</h1>
    </div>
  @end
</div>
```

### Binding input to properties

One of Edgewire's most powerful features is "data binding": the ability to automatically keep properties in-sync with form inputs on the page.

Let's bind the $title property from the CreatePost component to a text input using the wire:model directive:

```html
<form>
  <label for="title">Title:</label>

  <input type="text" id="title" wire:model="title" />
</form>
```

Any changes made to the text input will be automatically synchronized with the `title` property in your Edge.js component.

> "Why isn't my component live updating as I type?"
> If you tried this in your browser and are confused why the title isn't automatically updating, it's because Edgewire only updates a component when an "action" is submitted—like pressing a submit button—not when a user types into a field. This cuts down on network requests and improves performance. To enable "live" updating as a user types, you can use wire:model.live instead. Learn more about data binding.

Edgewire properties are extremely powerful and are an important concept to understand. For more information, check out the Edgewire properties documentation.

## Calling actions

Actions are methods within your Edgewire component that handle user interactions or perform specific tasks. They're often useful for responding to button clicks or form submissions on a page.

To learn more about actions, let's add a save action to the CreatePost component:

```ts
import { Component, view } from 'edgewire'
import Post from '#models/post'

export default class CreatePost extends Component {
  title: string

  async render() {
    return view('edgewire/create-post')
  }

  save() {
    Post.create({
      title: this.title,
    })
  }
}
```

Next, let's call the save action from the component's Edge.js view by adding the `wire:submit` directive to the `<form>` element:

```html
<form wire:submit="save">
  <label for="title">Title:</label>

  <input type="text" id="title" wire:model="title" />

  <button type="submit">Save</button>
</form>
```

When the "Save" button is clicked, the save() method in your Edgewire component will be executed and your component will re-render.

To keep learning about Edgewire actions, visit the actions documentation.

## Rendering components

To render a component, you can use the `@edgewire` Edge.js tag. It works the same as the [`@component`](https://edgejs.dev/docs/components/introduction#using-components) tag.

```html
@!component('create-post')
```

### Passing data into components

(WIP) requires mount

To pass outside data into an Edgewire component, you can use attributes on the component tag. This is useful when you want to initialize a component with specific data.

To pass an initial value to the `title` property of the `CreatePost` component, you can use the following syntax:

```html
@!component('create-post', { title: 'Initial title' })
```
