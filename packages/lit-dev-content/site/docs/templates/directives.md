---
title: Built-in directives
eleventyNavigation:
  key: Built-in directives
  parent: Templates
  order: 5
---

Directives are functions that can extend Lit by customizing the way an expression renders.
Lit includes a number of built-in directives to help with a variety of rendering needs:

**Styling:**
*   [`classMap`](#classmap) - sets a list of classes to an element based on an object
*   [`styleMap`](#stylemap) - sets a list of style properties to an element based on an object

**Rendering special values:**
*   [`repeat`](#repeat) - renders values from an iterable into the DOM, with optional keying
*   [`templateContent`](#templatecontent) - renders the content of a `<template>` element
*   [`unsafeHTML`](#unsafehtml) - renders a string as HTML rather than text
*   [`unsafeSVG`](#unsafesvg) - renders a string as SVG rather than text

**Conditional rendering:**
*   [`cache`](#cache) - caches rendered DOM when changing templates rather than discarding the DOM
*   [`guard`](#guard) - only re-evaluates the template when one of its dependencies changes
*   [`ifDefined`](#ifdefined) - sets an attribute if the value is defined and removes the attribute if undefined
*   [`live`](#live) - sets an attribute or property if it differs from the live DOM value rather than the last-rendered value

**Referencing the rendered DOM:**
*   [`ref`](#ref) - gets a reference to an element rendered in the template

**Asynchronous rendering:**
*   [`until`](#until) - renders placeholder content until one or more promises resolve
*   [`asyncAppend`](#asyncappend) - appends values from an `AsyncIterable` into the DOM as they are yielded
*   [`asyncReplace`](#asyncreplace) - renders the latest value from an `AsyncIterable` into the DOM as it is yielded

You can also build your own directives. For more information, see [Custom directives](/docs/templates/custom-directives/).

## classMap

Sets a list of classes to an element based on an object.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {classMap} from 'lit/directives/class-map.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
classMap(classInfo: {[name: string]: string | boolean | number})
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

`class` attribute expression (must be the only expression in the `class` attribute)

</td>
</tr>
</tbody>
</table>

The `classMap` directive uses the `element.classList` API to efficiently add and
remove classes to an element based on an object passed by the user. Each key in
the object is treated as a class name, and if the value associated with the key
is truthy, that class is added to the element. On subsequent renders, any
previously set classes that are falsy or no longer in the object are removed.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property({type: Boolean})
  enabled = false;

  render() {
    const classes = { enabled: this.enabled, hidden: false };
    return html`<div class=${classMap(classes)}>Classy text</div>`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    enabled: {type: Boolean},
  };

  constructor() {
    super();
    this.enabled = false;
  }

  render() {
    const classes = { enabled: this.enabled, hidden: false };
    return html`<div class=${classMap(classes)}>Classy text</div>`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

The `classMap` must be the only expression in the `class` attribute, but it can
be combined with static values:

```ts
html`<div class="my-widget ${classMap(dynamicClasses)}">Static and dynamic</div>`;
```

Explore `classMap` more in the [playground](/playground/#sample=examples/directive-class-map).

## styleMap

Sets a list of style properties to an element based on an object.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {styleMap} from 'lit/directives/style-map.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
styleMap(styleInfo: {[name: string]: string | undefined | null})
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

`style` attribute expression (must be the only expression in the `style` attribute)

</td>
</tr>
</tbody>
</table>

The `styleMap` directive uses the `element.style` API to efficiently add and
remove inline styles to an element based on an object passed by the user. Each
key in the object is treated as a style property name, the value is treated as
the value for that property. On subsequent renders, any previously set style
properties that are undefined or `null` are removed (set to `null`).

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property({type: Boolean})
  enabled = false;

  render() {
    const styles = { backgroundColor: this.enabled ? 'blue' : 'gray', color: 'white' };
    return html`<p style=${styleMap(styles)}>Hello style!</p>`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    enabled: {type: Boolean},
  };

  constructor() {
    super();
    this.enabled = false;
  }

  render() {
    const styles = { backgroundColor: this.enabled ? 'blue' : 'gray', color: 'white' };
    return html`<p style=${styleMap(styles)}>Hello style!</p>`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

For CSS properties that contain dashes, you can either use the camel-case equivalent, or put the property name in quotes. For example, you can write the the CSS property `font-family` as either `fontFamily` or `'font-family'`:

```js
{ fontFamily: 'roboto' }
{ 'font-family': 'roboto' }
```

Refer to CSS custom properties such as `--custom-color`, by placing the whole property name in quotes:

```js
{ '--custom-color': 'steelblue' }
```


The `styleMap` must be the only expression in the `style` attribute, but it can
be combined with static values:

```js
html`<p style="color: white; ${styleMap(moreStyles)}">More styles!</p>`;
```

Explore `styleMap` more in the [playground](/playground/#sample=examples/directive-style-map).

## repeat

Renders values from an iterable into the DOM, with optional keying.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {repeat} from 'lit/directives/repeat.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
repeat(items: Iterable<T>, keyfn: KeyFn<T>, template: ItemTemplate<T>)
repeat(items: Iterable<T>, template: ItemTemplate<T>)
type KeyFn<T> = (item: T, index: number) => unknown;
type ItemTemplate<T> = (item: T, index: number) => unknown;
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

Repeats a series of values (usually `TemplateResults`) generated from an
iterable, and updates those items efficiently when the iterable changes. When
the `keyFn` is provided, key-to-DOM association is maintained between updates by
moving DOM when required, and is generally the most efficient way to use
`repeat` since it performs minimum unnecessary work for insertions and removals.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property()
  items: Array<{id: number, name: string}> = [];

  render() {
    return html`
      <ul>
        ${repeat(this.items, (item) => item.id, (item, index) => html`
          <li>${index}: ${item.name}</li>`)}
      </ul>
    `;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    items: {},
  };

  constructor() {
    super();
    this.items = [];
  }

  render() {
    return html`
      <ul>
        ${repeat(this.items, (item) => item.id, (item, index) => html`
          <li>${index}: ${item.name}</li>`)}
      </ul>
    `;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

If no `keyFn` is provided, `repeat` will perform similar to a simple map of
items to values, and DOM will be reused against potentially different items.

See [When to use map or repeat](/docs/templates/lists/#when-to-use-map-or-repeat) for a discussion
of when to use `repeat` and when to use standard JavaScript flow control.

Explore `repeat` more in the [playground](/playground/#sample=examples/directive-repeat).

## templateContent

Renders the content of a `<template>` element.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {templateContent} from 'lit/directives/template-content.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
templateContent(templateElement: HTMLTemplateElement)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

Lit templates are encoded in Javascript, so that they can embed Javascript
expressions that make them dynamic. If you have a static HTML `<template>` that
you need to include in your Lit template, you can use the `templateContent`
directive to clone the template content and include it in your Lit template. As
long as the template element reference does not change between renders,
subsequent renders will no-op.

<div class="alert alert-warning">

Note, the template content should be developer-controlled and must not be
created using an untrusted string. Examples of untrusted content include query
string parameters and values from user inputs. Untrusted templates rendered with
this directive could lead to [cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) vulnerabilities.

</div>

{% switchable-sample %}

```ts
const templateEl = document.querySelector('template#myContent') as HTMLTemplateElement;

@customElement('my-element')
class MyElement extends LitElement {

  render() {
    return  html`
      Here's some content from a template element:
      ${templateContent(templateEl)}`;
  }
}
```

```js
const templateEl = document.querySelector('template#myContent');

class MyElement extends LitElement {

  render() {
    return  html`
      Here's some content from a template element:
      ${templateContent(templateEl)}`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `templateContent` more in the [playground](/playground/#sample=examples/directive-template-content).

## unsafeHTML

Renders a string as HTML rather than text.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {unsafeHTML} from 'lit/directives/unsafe-html.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
unsafeHTML(value: string | typeof nothing | typeof noChange)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

A key feature of Lit's templating syntax is that only strings originating in
template literals are parsed as HTML. Because template literals can only be
authored in trusted script files, this acts as a natural safeguard against XSS
attacks injecting untrusted HTML. However, there may be cases when HTML not
originating in script files needs to be rendered in a Lit template, for example
trusted HTML content fetched from a database. The `unsafeHTML` directive will
parse such a string as HTML and render it in a Lit template.

<div class="alert alert-warning">

Note, the string passed to `unsafeHTML` must be developer-controlled and not
include untrusted content. Examples of untrusted content include query string
parameters and values from user inputs. Untrusted content rendered with this
directive could lead to [cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) vulnerabilities.

</div>

{% switchable-sample %}

```ts
const markup = '<h3>Some HTML to render.</h3>';

@customElement('my-element')
class MyElement extends LitElement {

  render() {
    return html`
      Look out, potentially unsafe HTML ahead:
      ${unsafeHTML(markup)}
    `;
  }
}
```

```js
const markup = '<h3>Some HTML to render.</h3>';

class MyElement extends LitElement {

  render() {
    return html`
      Look out, potentially unsafe HTML ahead:
      ${unsafeHTML(markup)}
    `;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}


Explore `unsafeHTML` more in the [playground](/playground/#sample=examples/directive-unsafe-html).

## unsafeSVG

Renders a string as SVG rather than text.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {unsafeSVG} from 'lit/directives/unsafe-svg.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
unsafeSVG(value: string | typeof nothing | typeof noChange)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

Similar to with [`unsafeHTML`](#unsafeHTML), there may be cases when SVG content
not originating in script files needs to be rendered in a Lit template, for
example trusted SVG content fetched from a database. The `unsafeSVG` directive
will parse such a string as SVG and render it in a Lit template.

<div class="alert alert-warning">

Note, the string passed to `unsafeSVG` must be developer-controlled and not
include untrusted content. Examples of untrusted content include query string
parameters and values from user inputs. Untrusted content rendered with this
directive could lead to [cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) vulnerabilities.

</div>

{% switchable-sample %}

```ts
const svg = '<circle cx="50" cy="50" r="40" fill="red" />';

@customElement('my-element')
class MyElement extends LitElement {

  render() {
    return html`
      Look out, potentially unsafe SVG ahead:
      <svg width="40" height="40" viewBox="0 0 100 100"
        xmlns="http://www.w3.org/2000/svg" version="1.1">
        ${unsafeSVG(svg)}
      </svg> `;
  }
}
```

```js
const svg = '<circle cx="50" cy="50" r="40" fill="red" />';

class MyElement extends LitElement {

  render() {
    return html`
      Look out, potentially unsafe SVG ahead:
      <svg width="40" height="40" viewBox="0 0 100 100"
        xmlns="http://www.w3.org/2000/svg" version="1.1">
        ${unsafeSVG(svg)}
      </svg> `;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `unsafeSVG` more in the [playground](/playground/#sample=examples/directive-unsafe-svg).

## cache

Caches rendered DOM when changing templates rather than discarding the DOM. You
can use this directive to optimize rendering performance when frequently
switching between large templates.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {cache} from 'lit/directives/cache.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
cache(value: TemplateResult|unknown)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

When the value passed to `cache` changes between one or more `TemplateResult`s,
the rendered DOM nodes for a given template are cached when they're not in use.
When the template changes, the directive caches the _current_ DOM nodes before
switching to the new value, and restores them from the cache when switching back
to a previously-rendered value, rather than creating the DOM nodes anew.

{% switchable-sample %}

```ts
const detailView = (data) => html`<div>...</div>`;
const summaryView = (data) => html`<div>...</div>`;

@customElement('my-element')
class MyElement extends LitElement {

  @property()
  data = {showDetails: true, /*...*/ };

  render() {
    return html`${cache(this.data.showDetails
      ? detailView(this.data)
      : summaryView(this.data)
    )}`;
  }
}
```

```js
const detailView = (data) => html`<div>...</div>`;
const summaryView = (data) => html`<div>...</div>`;

class MyElement extends LitElement {
  static properties = {
    data: {},
  };

  constructor() {
    super();
    this.data = {showDetails: true, /*...*/ };
  }

  render() {
    return html`${cache(this.data.showDetails
      ? detailView(this.data)
      : summaryView(this.data)
    )}`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

When Lit re-renders a template, it only updates the modified portions: it doesn't create or remove any more DOM than needed. But when you switch from one template to another, Lit removes the old DOM and renders a new DOM tree.

The `cache` directive caches the generated DOM for a given expression and input template. In the example above, it caches the DOM for both the `summaryView` and `detailView` templates. When you switch from one view to another, Lit swaps in the cached version of the new view and updates it with the latest data. This can improve rendering performance when these views are frequently switched.

Explore `cache` more in the [playground](/playground/#sample=examples/directive-cache).

## guard

Only re-evaluates the template when one of its dependencies changes, to optimize
rendering performance by preventing unnecessary work.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {guard} from 'lit/directives/guard.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
guard(dependencies: unknown[], valueFn: () => unknown)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Any expression

</td>
</tr>
</tbody>
</table>

Renders the value returned by `valueFn`, and only re-evaluates `valueFn` when one of the
dependencies changes identity.

Where:

-   `dependencies` is an array of values to monitor for changes.
-   `valueFn` is a function that returns a renderable value.

`guard` is useful with immutable data patterns, by preventing expensive work
until data updates.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property()
  value: string = '';

  render() {
    return html`
      <div>
        ${guard([this.value], () => calculateSHA(this.value))}
      </div>`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    value: {},
  };

  constructor() {
    super();
    this.value = '';
  }

  render() {
    return html`
      <div>
        ${guard([this.value], () => calculateSHA(this.value))}
      </div>`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

In this case, the expensive `calculateSHA` function is only run when the `value` property changes.

Explore `guard` more in the [playground](/playground/#sample=examples/directive-guard).

## live

Sets an attribute or property if it differs from the live DOM value rather than the last-rendered value.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {live} from 'lit/directives/live.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
live(value: unknown)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Attribute or property expression

</td>
</tr>
</tbody>
</table>

When determining whether to update the value, checks the expression value
against the _live_ DOM value, instead of Lit's default behavior of checking
against the last set value.

This is useful for cases where the DOM value may change from outside of Lit. For
example, when using an expression to set an `<input>` element's `value`
property, a content editable element's text, or to a custom element that changes
its own properties or attributes.

In these cases if the DOM value changes, but the value set through Lit
expression hasn't, Lit won't know to update the DOM value and will leave it
alone. If this is not what you want—if you want to overwrite the DOM value with
the bound value no matter what—use the `live()` directive.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property()
  data = {value: 'test'};

  render() {
    return html`<input .value=${live(this.data.value)}>`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    data: {},
  };

  constructor() {
    super();
    this.data = {value: 'test'};
  }

  render() {
    return html`<input .value=${live(this.data.value)}>`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

`live()` performs a strict equality check agains the live DOM value, and if
the new value is equal to the live value, does nothing. This means that
`live()` should not be used when the expression will cause a type conversion. If
you use `live()` with an attribute expression, make sure that only strings are
passed in, or the expression will update every render.

Explore `live` more in the [playground](/playground/#sample=examples/directive-live).

## ifDefined

Sets an attribute if the value is defined and removes the attribute if undefined.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {ifDefined} from 'lit/directives/if-defined.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
ifDefined(value: unknown)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Attribute expression

</td>
</tr>
</tbody>
</table>

For AttributeParts, sets the attribute if the value is defined and removes the attribute if the value is undefined (`undefined` or `null`). For other part types, this directive is a no-op.

When more than one expression exists in a single attribute value, the attribute will be removed if _any_ expression uses `ifDefined` and evaluates to `undefined`/`null`. This is especially useful for setting URL attributes, when the attribute should not be set if required parts of the URL are not defined, to prevent 404's.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @property()
  filename: string | undefined = undefined;

  @property()
  size: string | undefined = undefined;

  render() {
    // src attribute not rendered if either size or filename are undefined
    return html`<img src="/images/${ifDefined(this.size)}/${ifDefined(this.filename)}">`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    filename: {},
    size: {},
  };

  constructor() {
    super();
    this.filename = undefined;
    this.size = undefined;
  }

  render() {
    // src attribute not rendered if either size or filename are undefined
    return html`<img src="/images/${ifDefined(this.size)}/${ifDefined(this.filename)}">`;
  }
}
customElements.define('my-element', MyEleent);
```

{% endswitchable-sample %}

Explore `ifDefined` more in the [playground](/playground/#sample=examples/directive-if-defined).

## ref

Retrieves a reference to an element rendered into the DOM.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {ref} from 'lit/directives/ref.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
ref(refOrCallback: RefOrCallback)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Element expression

</td>
</tr>
</tbody>
</table>

Although most DOM manipulation in Lit can be achieved declaratively using
templates, advanced situations may required getting a reference to an element
rendered in the template and manipulating it imperatively. Common examples of
when this may be useful include focusing a form control or calling an imperative
DOM manipulation library on a container element.

When placed on an element in the template, the `ref` directive will retrieve a
reference to that element once rendered. The element reference may be retrieved
in one of two ways: either by passing a `Ref` object or by passing a callback.

A `Ref` object acts as a container for a reference to the element, and can be
created using the `createRef` helper method found in the `ref` module. After
rendering, the `Ref`'s `value` property will be set to the element, where it
can be accessed in post-render lifecycle like `updated`.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  inputRef: Ref<HTMLInputElement> = createRef();

  render() {
    // Passing ref directive a Ref object that will hold the element in .value
    return html`<input ${ref(this.inputRef)}>`;
  }

  firstUpdated() {
    const input = this.inputRef.value!;
    input.focus();
  }
}
```

```js
class MyElement extends LitElement {

  inputRef = createRef();

  render() {
    // Passing ref directive a Ref object that will hold the element in .value
    return html`<input ${ref(this.inputRef)}>`;
  }

  firstUpdated() {
    const input = this.inputRef.value!;
    input.focus();
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

A ref callback can also be passed to the `ref` directive. The callback will be
called each time the referenced element changes.  If a ref callback is
rendered to a different element position or is removed in a subsequent render,
it will first be called with `undefined`, followed by another call with the new
element it was rendered to (if any). Note that in a `LitElement`, the callback
will be called bound to the host element automatically.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  render() {
    // Passing ref directive a change callback
    return html`<input ${ref(this.inputChanged)}>`;
  }

  inputChanged(input?: HTMLInputElement) {
    input?.focus();
  }
}
```

```js
class MyElement extends LitElement {

  render() {
    // Passing ref directive a change callback
    return html`<input ${ref(this.inputChanged)}>`;
  }

  inputChanged(input) {
    input?.focus();
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `ref` more in the [playground](/playground/#sample=examples/directive-ref).


## until

Renders placeholder content until one or more promises resolve.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {until} from 'lit/directives/until.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
until(...values: unknown[])
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Any expression

</td>
</tr>
</tbody>
</table>

Takes a series of values, including Promises. Values are rendered in priority order,
 with the first argument having the highest priority and the last argument having the
 lowest priority. If a value is a Promise, a lower-priority value will be rendered until it resolves.

The priority of values can be used to create placeholder content for async
data. For example, a Promise with pending content can be the first
(highest-priority) argument, and a non-promise loading indicator template can
be used as the second (lower-priority) argument. The loading indicator
renders immediately, and the primary content will render when the Promise
resolves.

{% switchable-sample %}

```ts
@customElement('my-element')
class MyElement extends LitElement {

  @state()
  private content = fetch('./content.txt').then(r => r.text());

  render() {
    return html`${until(this.content, html`<span>Loading...</span>`)}`;
  }
}
```

```js
class MyElement extends LitElement {
  static properties = {
    content: {state: true},
  };

  constructor() {
    super();
    this.content = fetch('./content.txt').then(r => r.text());
  }

  render() {
    return html`${until(this.content, html`<span>Loading...</span>`)}`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `until` more in the [playground](/playground/#sample=examples/directive-until).

## asyncAppend

Appends values from an `AsyncIterable` into the DOM as they are yielded.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {asyncAppend} from 'lit/directives/async-append.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
asyncAppend(iterable: AsyncIterable)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

`asyncAppend` renders the values of an [async iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of), appending each new value after the previous. Note that async generators also implement the async iterable protocol, and thus can be consumed by `asyncAppend`.

{% switchable-sample %}

```ts
async function *tossCoins(count: number) {
  for (let i=0; i<count; i++) {
    yield Math.random() > 0.5 ? 'Heads' : 'Tails';
    await new Promise((r) => setTimeout(r, 1000));
  }
}

@customElement('my-element')
class MyElement extends LitElement {

  @state()
  private tosses = tossCoins(10);

  render() {
    return html`
      <ul>${asyncAppend(this.tosses, (v: string) => html`<li>${v}</li>`)}</ul>`;
  }
}
```

```js
async function *tossCoins(count) {
  for (let i=0; i<count; i++) {
    yield Math.random() > 0.5 ? 'Heads' : 'Tails';
    await new Promise((r) => setTimeout(r, 1000));
  }
}

class MyElement extends LitElement {
  static properties = {
    tosses: {state: true},
  };

  constructor() {
    super();
    this.tosses = tossCoins(10);
  }

  render() {
    return html`
      <ul>${asyncAppend(this.tosses, (v) => html`<li>${v}</li>`)}</ul>`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `asyncAppend` more in the [playground](/playground/#sample=examples/directive-async-append).

## asyncReplace

Renders the latest value from an `AsyncIterable` into the DOM as it is yielded.

<table>
<thead><tr><th></th><th></th></tr></thead>
<tbody>
<tr>
<td class="no-wrap-cell vcenter-cell">Import</td>
<td class="wide-cell">

```js
import {asyncReplace} from 'lit/directives/async-replace.js';
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Signature</td>
<td class="wide-cell">

```ts
asyncReplace(iterable: AsyncIterable)
```

</td>
</tr>
<tr>
<td class="no-wrap-cell vcenter-cell">Usable location</td>
<td class="wide-cell">

Child expression

</td>
</tr>
</tbody>
</table>

Similar to [`asyncAppend`](#asyncappend), `asyncReplace` renders the values of an [async iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of), replacing the previous value with each new value.

{% switchable-sample %}

```ts
async function *countDown(count: number) {
  while (count > 0) {
    yield count--;
    await new Promise((r) => setTimeout(r, 1000));
  }
}

@customElement('my-element')
class MyElement extends LitElement {

  @state()
  private timer = countDown(10);

  render() {
    return html`Timer: <span>${asyncReplace(this.timer)}</span>.`;
  }
}
```

```js
async function *countDown(count) {
  while (count > 0) {
    yield count--;
    await new Promise((r) => setTimeout(r, 1000));
  }
}

class MyElement extends LitElement {
  static properties = {
    timer: {state: true},
  };

  constructor() {
    super();
    this.timer = countDown(10);
  }

  render() {
    return html`Timer: <span>${asyncReplace(this.timer)}</span>.`;
  }
}
customElements.define('my-element', MyElement);
```

{% endswitchable-sample %}

Explore `asyncReplace` more in the [playground](/playground/#sample=examples/directive-async-replace).

