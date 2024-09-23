# Getting Started

## Introduction

This `@foadmin/secrets` library allows you to safely store your environment variable files in your repository by bringing two commands `env:encrypt` and `env:decrypt`. It is heavily inspired by the popular framework [Laravel](https://laravel.com/).

## Installation

Install and configure the package using the following command :

```sh
node ace add @foadmin/shopkeeper
```

:::disclosure{title="See steps performed by the add command"}

1. Installs the `@foadmin/secrets` package using the detected package manager.

2. Registers the commands inside the `adonisrc.ts` file.

    ```ts
    {
      commands: [
        // ...other commands
        () => import('@foadmin/secrets/commands')
      ]
    }
    ```

:::

## Encrypting

You can encrypt your local environment file (`.env`) by using the `env:encrypt` command.
It will create a new file `.env.encrypted`

```sh
node ace env:encrypt
```

## Decrypting

You can decrypt your local environment file (`.env`) by using the `env:decrypt` command.
It will create a new file `.env.encrypted`

```sh
node ace env:decrypt
```

## Specific Environment

When encrypting or decrypting, it uses the `NODE_ENV` variable to select which file to work with.
You can use this same variable to select the environment variable file you want to encrypt:

```sh
APP_ENV=production node ace env:encrypt
APP_ENV=production node ace env:decrypt
```

## Specific Encryption Key

When encrypting or decrypting, it uses the `APP_KEY` variable for the encryption process.

```sh
APP_KEY=mysupersecurekey node ace env:encrypt
APP_KEY=mysupersecurekey node ace env:decrypt
```

:::tip

Under the hood, your files are encrypted using the `APP_KEY` environment variable. When decrypting a file the variable might not be available (as it might be inside the encrypted file). When it is the case, you have to provide it manually.

:::
