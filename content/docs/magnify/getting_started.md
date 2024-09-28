# Getting Started

## Installation

Install and configure the package using the following command :

```sh
node ace add @foadonis/magnify
```

:::disclosure{title="See steps performed by the add command"}

1. Installs the `@foadonis/magnify` package using the detected package manager.

2. Registers the following service provider inside the `adonisrc.ts` file.

    ```ts
    {
      providers: [
        // ...other providers
        () => import('@foadonis/magnify/providers/magnify')
      ]
    }
    ```

3. Registers the following commands inside the `adonisrc.ts` file.

    ```ts
    {
      providers: [
        // ...other providers
        () => import('@foadonis/magnify/commands')
      ]
    }
    ```

4. Creates the `config/magnify.ts` file.

5. Defines the environment variables and their validation for the selected engine.

6. Install required dependencies.

:::

## Configure your Model

Magnify tries to be as easy as possible to configure. To accomplish that, it brings a `Searchable` [mixin](https://www.typescriptlang.org/docs/handbook/mixins.html) to make any Lucid model `Searchable`.

A `Searchable` model will be automatically synchronized with your Search engine.

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

## Configure your Search Engine

Depending on the engine you selected during the configuration of `@foadonis/magnify` you must configure the credentials and the indexes.

### Algolia

[Algolia](https://www.algolia.com/) automatically configure your indexes for you, you do not need any extra configuration.

### Meilisearch

[Meilisearch](https://www.meilisearch.com/) requires each index to be configured. To accomplish that, head over to your `config/magnify.ts` file.

:::tip

For more informations about the available settings, head over the [Official Meilisearch documentation](https://www.meilisearch.com/docs/learn/getting_started/indexes#index-settings).

:::

```ts
// title: config/magnify.ts
import env from '#start/env'
import { defineConfig, engines } from '@foadonis/magnify'

const magnifyConfig = defineConfig({
  default: 'meilisearch',
  engines: {
    meilisearch: engines.meilisearch({
      host: env.get('MEILISEARCH_HOST'),
      apiKey: env.get('MEILISEARCH_API_KEY'),
      indexSettings: {
        posts: {
          filterableAttributes: ['isPublished'],
          sortableAttributes: ['createdAt'],
        }
      },
    }),
  },
})

export default magnifyConfig
```

After configuring the indexes, run the command `node ace magnify:sync-index-settings` to synchronize the new configuration with Meilisearch.

:::tip

In future versions of Magnify, the indexes configuration will be automatically infered from your model.

:::

### Typesense

[Typesense](https://typesense.org/) requires each collection (index) to be configured. To accomplish that, head over to your `config/magnify.ts` file.

:::tip

For more informations about the available settings, head over the [Official Typesense documentation](https://typesense.org/docs/27.1/api/collections.html#schema-parameters).

:::

```ts
// title: config/magnify.ts
import env from '#start/env'
import { defineConfig, engines } from '@foadonis/magnify'

const magnifyConfig = defineConfig({
  default: 'typesense',
  engines: {
    typesense: engines.typesense({
      apiKey: env.get('TYPESENSE_API_KEY'),
      nodes: [
        {
          url: env.get('TYPESENSE_NODE_URL'),
        },
      ],
      collectionSettings: {
        posts: {
          queryBy: ['title'],
          fields: [
            {
              name: 'title',
              type: 'string',
            },
            {
              name: 'isPublished',
              type: 'bool',
              optional: true,
            },
            {
              name: 'updatedAt',
              type: 'string',
            },
            {
              name: 'createdAt',
              type: 'string',
            },
          ],
        }
      },
    }),
  },
})

export default magnifyConfig
```

:::tip

In future versions of Magnify, the indexes configuration will be automatically inferred from your model.

:::

## Import the Data

Magnify only synchronize your models when they get created, updated or removed. Meaning that any existing data created before the installation of Magnify will not appear directly in your search engine. This is the purpose of the `magnify:import` command.

```sh
node ace magnify:import '#models/user'

node ace magnify:import user

node ace magnify:import ./app/models/user.js
```

## Start Searching

Once everything is configured, you can start searching over your Model:

```ts
const posts = await Post.search('Magnify').take(20).get()
```

You can find more informations in the [Advanced Page](./advanced.md).
