# Migrating away from `--config` in Expo CLI

The `--config` flag for commands like `expo start` and `expo publish` was added to provide developers with a mechanism to switch between different `app.json` configuration files to support use cases like staging environments and white labeling.

At the time the flag was introduced, dynamic configuration with `app.config.js` was not possible. Now that it is, we are deprecating the `--config` flag in favor of using `app.config.js`. The `--config` flag will continue to work for existing use cases, but it won't be supported in new scenarios, such as on EAS Build or when embedding app config when building native projects locally.

The migration process itself is quick, here's how you can do it.

### Migrating from using multiple `app.json` files with `--config` to `app.config.js`

Imagine you have three config files: `app.json`, `app.staging.json`, and `app.production.json`. When you run your app locally for development, you run `expo start`. When you build and publish for staging, you run `expo build:[ios|android] --config app.staging.json` and `expo publish --config app.staging.json`. You'd use a similar sort of thing for `app.production.json`.

Here's an example `app.json` for development, if you're reading this you probably already understand what you would change for staging and produciton.

```
{
  "expo": {
    "name": "MyApp (Development)",
    "slug": "myapp",
    "icon": "./assets/icon.png",
    "splash": "./assets/splash.png",
    "extra": {
      "apiUrl": "https://localhost:3000/api"
    }
  }
}
```

We can migrate to `app.config.js` and switch the configuration that we use depending on an environment variable. We can specify the environment variable at the same time as we run a command with Expo CLI, for example: `APP_ENV=production expo build:android`. On Windows this will be slightly different and it depends on your shell, but you can use `npx cross-env APP_ENV=production expo build:android` if you're not sure what to do.

While there is essentially unlimited flexibility in how you structure your project, here are two possible ways you may do this: move all of the config to `app.config.js`, or keep the config files separate and load the appropriate file from `app.config.js`.

1. **Move all config to `app.config.js`**

Create `app.config.js`, and copy and paste your config into one file.

```js
const commonConfig = {
  slug: "myapp",
  icon: "./assets/icon.png",
  splash: "./assets/splash.png",
};

export default () => {
  if (process.env.APP_ENV === "production") {
    return {
      ...commonConfig,
      name: "MyApp",
      extra: {
        apiUrl: "https://production.com/api",
      },
    };
  } else if (process.env.APP_ENV === "staging") {
    return {
      ...commonConfig,
      name: "MyApp (Staging)",
      extra: {
        apiUrl: "https://staging.com/api",
      },
    };
  } else {
    return {
      ...commonConfig,
      name: "MyApp (Development)",
      extra: {
        apiUrl: "https://localhost:3000/api",
      },
    };
  }
};
```

2. **Keep config in separate files, select the config in `app.config.js`**

Rename your `app.json` to `app.development.json` and create `app.config.js` with the following contents:

```js
export default () => {
  if (process.env.APP_ENV === "production") {
    return require("./app.production.json");
  } else if (process.env.APP_ENV === "staging") {
    return require("./app.staging.json");
  } else {
    return require("./app.development.json");
  }
};
```