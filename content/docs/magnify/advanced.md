# Advanced

## Model Configuration

### Index Name

By default, Magnify uses the table name of your model as the index name. You can change this behavior by overriding the `$indexName` getter function.

```ts
// title: app/models/post.ts
import { compose } from '@adonisjs/core/helpers'
import { BaseModel, column } from '@adonisjs/lucid/orm'
import { Searchable } from '@foadonis/magnify'

export default class Post extends compose(BaseModel, Searchable) {
  @column({ isPrimary: true })
  declare id: string

  @column()
  declare title: string

  static get $indexName(): string {
    return 'different_index'
  }
}
```

:::warning

Depending on your Search engine, you might as well have to change your index configuration in `config/magnify.ts` to match your new index name.

:::

### Model Searchable Data

By default, Magnify uses `model.serialize()` to store your model in the index. You can change this behavior by overriding the `toSearchableObject` method.

```ts
// title: app/models/post.ts
import { compose } from '@adonisjs/core/helpers'
import { BaseModel, column } from '@adonisjs/lucid/orm'
import { Searchable } from '@foadonis/magnify'

export default class Post extends compose(BaseModel, Searchable) {
  @column({ isPrimary: true })
  declare id: string

  @column()
  declare title: string

  toSearchableObject() {
    return {
      id: this.id,
      title: this.title,
      tags: ['Magnify', 'Adonis'],
    }
  }
}
```

### Model Search Engine

If you have configured multiple engine, you can define the use Search engine per model.

```ts
import { compose } from '@adonisjs/core/helpers'
import { BaseModel, column } from '@adonisjs/lucid/orm'
import { Searchable } from '@foadonis/magnify'
import magnify from '@foadonis/magnify/services/magnify'

export default class Post extends compose(BaseModel, Searchable) {
  @column({ isPrimary: true })
  declare id: string

  @column()
  declare title: string

  get $searchEngine() {
    return magnify.engine('typesense')
  }
}
```

## Commands

### `magnify:import`

If you are installing Magnify into an existing project, you may already have database records you need to import into your indexes. Magnify provides a magnify:import Ace command that you may use to import all of your existing records into your search indexes:

```sh
node ace magnify:import '#models/user'

node ace magnify:import user

node ace magnify:import ./app/models/user.js
```

### `magnify:flush`

Flush is the opposite of the `magnify:import` command, it will delete all the documents of the index.

```sh
node ace magnify:flush '#models/user'

node ace magnify:flush user

node ace magnify:flush ./app/models/user.js
```

### `magnify:sync-index-settings`

For the Search engines that require prior index configuration, you will need to use the `magnify:sync-index-settings` command.

```sh
node ace magnify:sync-index-settings
```

## Search

You may begin searching a model using the `search` method. The search method accepts a single string that will be used to search your models. You should then chain the `get` method onto the search query to retrieve the Lucid models that match the given search query:

```ts
const posts = await Post.search('Magnify').get()
```

### Where Clauses

Magnify allows you to add simple "where" clauses to your search queries. Currently, these clauses only support basic numeric equality checks and are primarily useful for scoping search queries by an owner ID:

```ts
const posts = await Post
  .search('Magnify')
  .where('isPublished', true)
  .get()
```

In addition, the `whereIn` method may be used to verify that a given column's value is contained within the given array:

```ts
const posts = await Post
  .search('Magnify')
  .whereIn('status', ['published', 'draft'])
  .get()
```

:::warning

If your application is using Meilisearch, you must configure your application's filterable attributes before utilizing Magnify's "where" clauses.

:::

### Pagination

In addition to retrieving a collection of models, you may paginate your search results using the paginate method. This method will return a `SimplePaginator` instance just as if you had paginated a traditional Lucid query:

```ts
const posts = await Post
  .search('Magnify')
  .whereIn('status', ['published', 'draft'])
  .paginate(15, 1)
```

Once you have retrieved the results, you may display the results and render the page links using Edge just as if you had paginated a traditional Lucid query:

```ts
// title: app/controllers/posts_controller.ts
import { HttpContext } from '@adonisjs/core/http'
import db from '@adonisjs/lucid/services/db'

class PostsController {
  async index({ request, view }: HttpContextContract) {
    const page = request.input('page', 1)
    const limit = 10

    const posts = await Post
      .search('Magnify')
      .whereIn('status', ['published', 'draft'])
      .paginate(limit, page)

    posts.baseUrl('/posts')

    return view.render('posts/index', { posts })
  }
}
```

```html
// title: resources/views/posts/index.edge
<div>
  @each(post in posts)
    <h1>{{ post.title }}</h1>
    <p> {{ excerpt(post.body, 200) }} </p>
  @endeach
</div>

<hr>

<div>
  @each(anchor in posts.getUrlsForRange(1, posts.lastPage))
    <a href="{{ anchor.url }}">
      {{ anchor.page }}
    </a>
  @endeach
</div>
```

:::tip

You can find more informations about pagination on the [Official Lucid Documentation](https://lucid.adonisjs.com/docs/pagination)

:::
