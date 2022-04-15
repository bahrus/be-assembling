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