# Plugins

Plugins are the building blocks of features in a Docusaurus 2 site. Each plugin handles its own individual feature. Plugins may work and be distributed as part of a bundle via presets.

## Plugins design {#plugins-design}

Docusaurus' implementation of the plugins system provides us with a convenient way to hook into the website's lifecycle to modify what goes on during development/build, which involves (but is not limited to) extending the webpack config, modifying the data loaded, and creating new components to be used in a page.

## Creating plugins {#creating-plugins}

A plugin is a function that takes two parameters: `context` and `options`. It returns a plugin instance object (or a promise). You can create plugins as functions or modules. For more information, refer to the [plugin method references section](./api/plugin-methods/README.md).

### Functional definition {#functional-definition}

You can use a plugin as a function directly included in the Docusaurus config file:

```js title="docusaurus.config.js"
module.exports = {
  // ...
  plugins: [
    // highlight-start
    async function myPlugin(context, options) {
      // ...
      return {
        name: 'my-plugin',
        async loadContent() {
          // ...
        },
        async contentLoaded({content, actions}) {
          // ...
        },
        /* other lifecycle API */
      };
    },
    // highlight-end
  ],
};
```

### Module definition {#module-definition}

You can use a plugin as a module path referencing a separate file or NPM package:

```js title="docusaurus.config.js"
module.exports = {
  // ...
  plugins: [
    // without options:
    './my-plugin',
    // or with options:
    ['./my-plugin', options],
  ],
};
```

Then in the folder `my-plugin`, you can create an `index.js` such as this:

```js title="my-plugin.js"
module.exports = async function myPlugin(context, options) {
  // ...
  return {
    name: 'my-plugin',
    async loadContent() {
      /* ... */
    },
    async contentLoaded({content, actions}) {
      /* ... */
    },
    /* other lifecycle API */
  };
};
```

## Theme design

When plugins have loaded their content, the data is made available to the client side through actions like [`createData` + `addRoute`](../api/plugin-methods/lifecycle-apis.md#addRoute) or [`setGlobalData`](../api/plugin-methods/lifecycle-apis.md#setglobaldatadata-any-void). This data has to be _serialized_ to plain strings, because [plugins and themes run in different environments](./architecture.md). Once the data arrives on the client side, the rest becomes familiar to React developers: data is passed along components, components are bundled with Webpack, and rendered to the window through `ReactDOM.render`...

**Themes provide the set of UI components to render the content.** Most content plugins need to be paired with a theme in order to be actually useful. The UI is a separate layer from the data schema, which makes swapping designs easy.

For example, a Docusaurus blog may consist of a blog plugin and a blog theme.

:::note

This is a contrived example: in practice, `@docusaurus/theme-classic` provides the theme for docs, blog, and layouts.

:::

```js title="docusaurus.config.js"
module.exports = {
  // highlight-next-line
  themes: ['theme-blog'],
  plugins: ['plugin-content-blog'],
};
```

And if you want to use Bootstrap styling, you can swap out the theme with `theme-blog-bootstrap` (another fictitious non-existing theme):

```js title="docusaurus.config.js"
module.exports = {
  // highlight-next-line
  themes: ['theme-blog-bootstrap'],
  plugins: ['plugin-content-blog'],
};
```

Now, although the theme receives the same data from the plugin, how the theme chooses to _render_ the data as UI can be drastically different.

While themes share the exact same lifecycle methods with plugins, themes' implementations can look very different from those of plugins based on themes' designed objectives.

Themes are designed to complete the build of your Docusaurus site and supply the components used by your site, plugins, and the themes themselves. A theme still acts like a plugin and exposes some lifecycle methods, but most likely they would not use [`loadContent`](../api/plugin-methods/lifecycle-apis.md#loadContent), since they only receive data from plugins, but don't generate data themselves; themes are typically also accompanied by an `src/theme` directory full of components, which are made known to the core through the [`getThemePath`](../api/plugin-methods/extend-infrastructure#getThemePath) lifecycle.

To summarize:

- Themes share the same lifecycle methods with Plugins
- Themes are run after all existing Plugins
- Themes add component aliases by providing `getThemePath`. We will talk about theme aliases right next.