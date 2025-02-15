<p align="center">
  <a href="https://www.npmjs.com/package/@friendsofreactjs/react-css-themr">
    <img alt="npm version" src="https://img.shields.io/npm/v/@friendsofreactjs/react-css-themr.svg?style=flat-square">
  </a>
  <a href="https://www.npmjs.com/package/@friendsofreactjs/react-css-themr">
    <img alt="monthly downloads from npm" src="https://img.shields.io/npm/dm/@friendsofreactjs/react-css-themr.svg?style=flat-square">
  </a>
  <a href="https://travis-ci.org/FriendsOfReactJS/react-css-themr">
    <img alt="Travis CI Build Status" src="https://img.shields.io/travis/FriendsOfReactJS/react-css-themr/master.svg?style=flat-square&label=Travis+CI">
  </a>
  <a href="https://greenkeeper.io">
    <img alt="We use greenkeeper" src="https://badges.greenkeeper.io/FriendsOfReactJS/react-css-themr.svg?style=flat-square">
  </a>
  <a href="https://github.com/prettier/prettier">
    <img alt="code style: prettier" src="https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square">
  </a>
  <a href="https://github.com/semantic-release/semantic-release">
    <img alt="We are using semantic-release" src="https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg?style=flat-square">
  </a>
  <a href="https://twitter.com/friendsofreact">
    <img alt="Follow friends of react on Twitter" src="https://img.shields.io/twitter/follow/friendsofreact.svg?label=follow+friendsofreact&style=flat-square">
  </a>
  <a href="https://spectrum.chat/friends-of-reactjs">
    <img alt="Join the community on Spectrum" src="https://withspectrum.github.io/badge/badge.svg">
  </a>
</p>


# Friends of react: React CSS Themr

Easy theming and composition for CSS Modules.

```
$ npm install --save @friendsofreactjs/react-css-themr
```

**Note: Feedback and contributions on the docs are highly appreciated.**

## Why?

When you use [CSS Modules](https://github.com/css-modules/css-modules) to style your components, a classnames object is usually imported from the same component. Since css classes are scoped by default, there is no easy way to make your component customizable for the outside world.

## The approach

Taking ideas from [future-react-ui](https://github.com/nikgraf/future-react-ui) and [react-themeable](https://github.com/markdalgleish/react-themeable), a component should be shipped **without** styles. This means we can consider the styles as an **injectable dependency**. In CSS Modules you can consider the imported classnames object as a **theme** for a component. Therefore, every styled component should define a *classname API* to be used in the rendering function.

The most immediate way of providing a classname object is via *props*. In case you want to import a component with a theme already injected, you have to write a higher order component that does the job. This is ok for your own components, but for ui-kits like [React Toolbox](http://www.react-toolbox.io) or [Belle](http://nikgraf.github.io/belle/), you'd have to write a wrapper for every single component you want to use. In this fancy, you can understand the theme as a **set** of related classname objects for different components. It makes sense to group them together in a single object and move it through the component tree using a context. This way, you can provide a theme either via **context**, **hoc** or **props**.

The approach of @friendsofreactjs/react-css-themr consists of a *provider* and a *decorator*. The provider sets a context theme. The decorator adds to your components the logic to figure out which theme should be used or how should it be composed, depending on configuration, context and props.

## Combining CSS modules

There are three possible sources for your component. Sorted by priority: **context**, **configuration** and **props**. Any of them can be missing. In case multiple themes are present,  you may want to compose the final classnames object in three different ways:

- *Override*: the theme object with the highest priority is the one used.
- *Softly merging*: theme objects are merged but if a key is present in more than one object, the final value corresponds to the theme with highest priority.
- *Deeply merging*: theme objects are merged and if a key is present in more than one object, the values for each objects are concatenated.

You can choose whatever you want. We consider the last one as the most flexible so it's selected *by default*.

## How does it work?

Say you have a `Button` component you want to make themeable. You should pass a unique name identifier that will be used to retrieve its theme from context in case it is present.

```jsx
// Button.js
import React, { Component } from 'react';
import { themr } from '@friendsofreactjs/react-css-themr';

@themr('MyThemedButton')
class Button extends Component {
  render() {
    const { theme, icon, children } = this.props;
    return (
      <button className={theme.button}>
        { icon ? <i className={theme.icon}>{icon}</i> : null}
        <span className={theme.content}>{children}</span>
      </button>
    )
  }
}

export default Button;
```

The component is defining an API for theming that consists of three classnames: button, icon and content. Now, a component can use a button with a success theme like:

```jsx
import Button from './Button';
import successTheme from './SuccessButton.css';

export default (props) => (
  <div {...props}>
    <p>Do you like it?</p>
    <Button theme={successTheme}>Yeah!</Button>
  </div>
);
```

### Default theming

If you use a component with a base theme, you may want to import the component with the theme already injected. Then you can compose its style via props with another theme object. In this case the base css will **always** be bundled:

```jsx
// SuccessButton.js
import React, { Component } from 'react';
import { themr } from '@friendsofreactjs/react-css-themr';
import successTheme from './SuccessButton.css';

@themr('MySuccessButton', successTheme)
class Button extends Component {
  render() {
    const { theme, icon, children } = this.props;
    return (
      <button className={theme.button}>
        { icon ? <i className={theme.icon}>{icon}</i> : null}
        <span className={theme.content}>{children}</span>
      </button>
    )
  }
}

export default Button;
```

Imagine you want to make the success button uppercase for a specific case. You can include the classname mixed with other classnames:

```jsx
import React from 'react';
import SuccessButton from 'SuccessButon';
import style from './Section.css';

export default () => (
  <section className={style.section}>
    <SuccessButton theme={style}>Yai!</SuccessButton>
  </section>
);
```

And being `Section.css` something like:

```scss
.section { border: 1px solid red; }
.button  { text-transform: uppercase; }
```

The final classnames object for the `Button` component would include class values from `SuccessButton.css` and `Section.css` so it would be uppercase!

### Context theming

Although context theming is not limited to ui-kits, it's very useful to avoid declaring hoc for every component. For example, in [react-toolbox](http://www.react-toolbox.io), you can define a context theme like:

```jsx
import React from 'react';
import { render } from 'react-dom';
import { ThemeProvider } from '@friendsofreactjs/react-css-themr';
import App from './app'

const contextTheme = {
  RTButton: require('react-toolbox/lib/button/style.scss'),
  RTDialog: require('react-toolbox/lib/dialog/style.scss')
};

const content = (
  <ThemeProvider theme={contextTheme}>
    <App />
  </ThemeProvider>
);

render(content, document.getElementById('app'));
```

The main idea is to inject classnames objects for each component via context. This way you can have the whole theme in a single place and forget about including styles in every require. Any `Button` or `Dialog` component will use the styles provided in the context.

## API

### `<ThemeProvider theme>`

Makes available a `theme` context to use in styled components. The shape of the theme object consists of an object whose keys are identifiers for styled components provided with the `themr` function with each theme as the corresponding value. Useful for ui-kits.

### `themr(Identifier, [defaultTheme], [options])`

Returns a `function` to wrap a component and make it themeable.

The returned component accepts a `theme`, `composeTheme`, `innerRef` and `mapThemrProps` props apart from the props of the original component. The former two are used to provide a `theme` to the component and to configure the style composition, which can be configured via options too. `innerRef` is used to pass a ref callback to the decorated component and `mapThemrProps` is a function that can be used to map properties to the decorated component. The function arguments are:

- `Identifier` *(String)* used to provide a unique identifier to the component that will be used to get a theme from context.
- `[defaultTheme]` (*Object*) is  classname object resolved from CSS modules. It will be used as the default theme to calculate a new theme that will be passed to the component.
- `[options]` (*Object*) If specified it allows to customize the behavior: 
  - [`composeTheme = 'deeply'`] *(String)* allows to customize the way themes are merged or to disable merging completely. The accepted values are `deeply` to deeply merge themes, `softly` to softly merge themes and `false` to disable theme merging.
  - [`mapThemrProps = (props, theme) => ({ ref, theme })`] *(Function)* allows to customize how properties are passed down to the decorated component. By default, themr extracts all own properties passing down just `innerRef` as `ref` and the generated theme as `theme`. If you are decorating a component that needs to map the reference or any other custom property, this function is called with *all* properties given to the component plus the generated `theme` in the second parameter. It should return the properties you want to pass.

## Contribution

We'd love you to contribute to react-css-themr. First, please read our [Contribution Guide](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md#code-of-conduct).

We try to make it as easy as possible.
We are using semantic-release to have more time to concentrate on important stuff
instead of struggling in the dependency or release hell.

Therefore the first rule is to follow the [eslint commit message guideline](https://github.com/conventional-changelog-archived-repos/conventional-changelog-eslint/blob/master/convention.md).
When you always commit via `yarn commit` this is really easy. Commitizen will guide you.

All PRs will be merged into the develop branch. When we merge the develop into the master
travis and semantic release will build a new release.

So no one can directly push into master!

### Development commands

We have some yarn scripts that should make live easier. The deployment scripts are just for
semantic-release and travis, but the following will help you.

| Command         | Description                    |
| --------------- | ------------------------------ |
| `yarn build` |  Runs the development build. |
| `yarn lint`  | Executes linter, we use prettier with eslint. |
| `yarn test`  | Executes the jest test on all source files. |
| `yarn test:watch`  | Perfect while writing tests is the watcher for test. |
| `yarn commit`  | Executes commitizen to guide you for the correct commit message. |

## About

The project is originally authored by [Javi Velasco](https://twitter.com/javivelasco) as an effort of providing a better customization experience for [React Toolbox](http://react-toolbox.io). Any comments, improvements or feedback is highly appreciated.

We thank Javi Velasco for all his efforts and for creating such a great package. The package `javivelasco/react-css-themr` should not be unmaintained - so the friends of react will continue.

Thanks to [Nik Graf](http://www.twitter.com/nikgraf) and [Mark Dalgleish](http://www.twitter.com/markdalgleish) for their thoughts about theming and customization for React components.

## License

This project is licensed under the terms of the [MIT license](https://github.com/FriendsOfReactJS/@friendsofreactjs/react-css-themr/blob/master/LICENSE).
