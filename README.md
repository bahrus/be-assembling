# be-assembling

```html
<template id=my-template>
    <div>This is my template</div>
    <slot name=some-slot></slot>
</template>
<template be-assembling>
    <my-template>
        <div slot=some-slot>This is some slot</div>
    </my-template>
</template>
```


...yields...

```html
<div>This is my template</div>
<div>This is some slot</div>
```

Why?

As one moves from the micro, "primitive" JS-centric web components to the macro, HTML dominated web components, we are faced with dilemmas as far as how to pass in dependencies that go in quite deep.

Parts doesn't support nesting, and other configurations, like columns of a grid, are a pain to have to pass through the system.

So one solution is to design these macro web components using ideas loosely modeled after dependency injection:

Define a flat list of all the templates containing html snippets that is essentially the configuration of the macro web component.

Using templates via slots, allow consumers of the web component to override the default templates provided above.

Now given this flat list of templates (some default, some user overrides), use this component to weave the templates together in order to achieve the true markup.

