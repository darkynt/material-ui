# CSS in JS

Material-UI aims to provide strong foundations for building dynamic UIs.
For the sake of simplicity **we expose our internal styling solution to users**.
You can use it, but you don't have to. This styling solution is interoperable with another other one, like [PostCSS](https://github.com/postcss/postcss), [CSS modules](https://github.com/css-modules), or [styled-components](https://github.com/styled-components/styled-components).

## Our styling solution

In the previous versions, Material-UI was using LESS, then a custom inline-style solution to write the style of the components.
These approaches have proven to be limited.
Finally, we have [moved toward](https://github.com/oliviertassinari/a-journey-toward-better-style) a *CSS-in-JS* solution. We think that it's the future:
- [A Unified Styling Language](https://medium.com/seek-blog/a-unified-styling-language-d0c208de2660)
- [The future of component-based styling](https://medium.freecodecamp.com/css-in-javascript-the-future-of-component-based-styling-70b161a79a32)

So, you may have noticed in the demos how that *CSS-in-JS* looks.
We use the `createStyleSheet` function and `withStyles` Higher-order Component.
Here is an example:

{{demo='pages/customization/CssInJs.js'}}

## JSS

The styling solution of Material-UI is using [JSS](https://github.com/cssinjs/jss) at its core.
It's a [high performance](https://github.com/cssinjs/jss/blob/master/docs/performance.md) JS to CSS compiler which works at runtime and server-side.
It is about 5KB (minified and gzipped) and is extensible via a [plugins](https://github.com/cssinjs/jss/blob/master/docs/plugins.md) API.

If you end up using that styling solution in your codebase, you're going to need to *learn the API*.
The best place to start is by looking at the features each [plugin](http://cssinjs.org/plugins) is providing. Material-UI is using the [`jss-preset-default`](http://cssinjs.org/jss-preset-default) module. You can always add new plugins if needed with the [`JssProvider` helper](https://github.com/cssinjs/react-jss#custom-setup).

## Sheet registry

When rendering on the server, you will need to get all rendered styles as a CSS string.
`SheetsRegistry` class allows you to manually aggregate and stringify them.
Read more about [Server Rendering here](/guides/server-rendering).

{{demo='pages/customization/JssRegistry.js'}}

## Class names

You may have noticed, the class names generated by our styling solution are **non-deterministic**.
You can't rely on their name, the following CSS won't work.
```css
.MuiAppBar-root-a2 {
  opacity: 0.6
}
```

Instead, you have to use the `classes` property.
On the other hand, thanks to the non-deterministic nature of our class names, we
can implement optimization for development and production.
They are easy to debug in development and as short as possible in production:

- development: `.MuiAppBar-root-a2`
- production: `.ca2`

## API

### `createStyleSheet(styles) => styleSheet`

Generate a new style sheet that represents the style you want to inject in the DOM.

#### Arguments

If the value isn't provided, we will try to fallback to the name of the component.
1. `styles` (*Function | Object*): A function generating the styles or an object.

Use the function if you need to have access to the theme. It's provided as the first argument.

#### Returns

`styleSheet`: The newly created style sheet.

#### Examples

```js
import { createStyleSheet } from 'material-ui/styles';

const styleSheet = createStyleSheet((theme) => ({
  root: {
    color: 'inherit',
    textDecoration: 'inherit',
    '&:hover': {
      textDecoration: 'underline',
    },
  },
  primary: {
    color: theme.palette.primary[500],
  },
}));
```

### `withStyles(styleSheet, [options]) => Higher-order Component`

Link a style sheet with a component.
It does not modify the component passed to it; instead, it returns a new component, with a `classes` property.
This `classes` object contains the name of the class names injected in the DOM.

Some implementation details that might be interesting to being aware of:
- It's adding a `classes` property so you can override the injected class names from the outside.
- It's adding a `innerRef` property so you can get a reference to the wrapped component.
- It's forwarding *non react static* properties so this HOC is more "transparent".
For instance, it can be used to defined a `getInitialProps()` static method (next.js).

#### Arguments

1. `styleSheet` (*Object*): The style sheet to link to the component. This object can be created with `createStyleSheet`.
2. `options` (*Object* [optional]):
  - `options.withTheme` (Boolean [optional]): Provide the `theme` object to the component as a property.
  - `options.name` (*String* [optional]): The name of the style sheet. Useful for debugging.

#### Returns

`Higher-order Component`: Should be used to wrap a component.

#### Examples

```js
import { withStyles } from 'material-ui/styles';

class MyComponent extends Component {
  render () {
    return <div />;
  }
}

export default withStyles(styleSheet)(MyComponent);
```

Also, you can use as [decorators](https://babeljs.io/docs/plugins/transform-decorators/) like so:

```js
import { withStyles } from 'material-ui/styles';

@withStyles(styleSheet)
class MyComponent extends Component {
  render () {
    return <div />;
  }
}

export default MyComponent
```