---
summary: ""
---

# Introduction

Magnify provides a simple, driver based solution for adding full-text search to your Lucid models. Using Lucid hooks, Magnify automatically keep your search indexes in sync with your database.

Currently, Magnify ships with [Algolia](https://algolia.com), [Meilisearch](https://www.meilisearch.com/) and [Typesense](https://typesense.org/) engines. Furthermore, writing custom engines is simple and you are free to extend Magnify with your own search implementations.

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
}
```

```ts
import Post from '#models/post'

const posts = await Post.search('Adonis').take(10).get()
```
