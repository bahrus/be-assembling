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

## Example

Consider this example of markup, that takes a JSON array, and slices the columns into a tree like structure, which then links in with a flat grid display.

The markup for this web component could start out looking something like the following:

```html
    <form be-reformable='{
        "autoSubmit": true,
        "debug": true
    }'
        action="test.json"
        target="[-value]"
    ></form>
    <obj-ml -value></obj-ml>
    <pass-down on="value-changed" vft="value.results.0.members" to="xtal-slice[-list]" m='1'></pass-down>
    <label>
        <input type="search" name="search" placeholder="Search" size="100" />
        <pass-down on="change" vft from="label" to="[-new-slice-path]"></pass-down>
    </label>
    <xtal-slice -list -new-slice-path split-name-by='_'></xtal-slice>
    <pass-down on='tree-view-changed' vft='treeView' to=[-nodes]></pass-down>
    <pass-down on="slice-changed" vft="slice.list" to=.slice-view[-list]></pass-down>
    <xtal-tree -nodes></xtal-tree>

    <xtal-vlist style="height:600px;width:100%;" page-size="10" id="vlist" be-observant='{
        "list": {"observe": "xtal-tree", "vft": "viewableNodes"}
    }' row-transform='{
        "div": [{}, {}, {"data-path": "path", "style": "marginStyle"}],
        "label": "name",
        "expanderParts": [true, {"if": "open"}, ["-"], ["+"]],
        "button": [{}, {}, {"data-children": "hasChildren"}]
    }'
        be-channeling='[
            {
                "eventFilter": "click",
                "toNearestUpMatch": "xtal-tree",
                "prop": "toggledNodePath",
                "vfe": "path.0.parentElement.dataset.path",
                "composedPathMatch": "button"
            },
            {
                "eventFilter": "click",
                "toNearestUpMatch": "xtal-slice",
                "prop": "newSlicePath",
                "vfe": "path.0.parentElement.dataset.path",
                "composedPathMatch": "label"
            }
        ]'
    >>
        <div class=node slot=row itemscope >
            <button class="expander" part=expander>.</button>
            <label></label>
        </div>
        <template slot="style">
            <style>
                button.expander{
                    display:none;
                }
                button[data-children].expander{
                    display:inline;
                }
            </style>
        </template>
    </xtal-vlist>
    <header>
        <span>First Name</span><span>Last Name</span>
    </header>
    <xtal-vlist style="height:600px;width:100%;" class='slice-view' -list row-transform='{
        ".first_name": "first_name",
        ".last_name": "last_name"
    }'>
        <template slot='style'>
            <style>

                span{
                    color:green;
                }
                div[slot="row"]{
                    display:flex;
                    flex-direction: row;
                    justify-content: space-between;
                }
            </style>
        </template>

        <div slot='row'>
            <span class='first_name'></span><span class='last_name'></span>
        </div>
    </xtal-vlist>
```

But now the consumer of the web component may want to add some fields to the form, allowing for filtering / searching of the url to retrieve the JSON data.

The consumer might want to switch the order of the first_name and last_name columns, add additional columns, etc.

Defining a hierarchical structure of the light children is one approach. But this component (together with be-transplanted) supports an alternative:

Make the markup look as follows:

```html
<slot name=form be-transplanted>
    <template>
        <form be-reformable='{
        "autoSubmit": true,
        "debug": true
    }'
        action="test.json"
        target="[-value]"
    ></form>
</slot>

<slot name=search be-transplanted>
    <template>
        <label>
            <input type="search" name="search" placeholder="Search" size="100" />
        </label>
    </template>
</slot>

<slot name=sliced-vlist be-transplanted>
    <template>
        <xtal-vlist style="height:600px;width:100%;" page-size="10" id="vlist" 
            be-observant='{
                "list": {"observe": "xtal-tree", "vft": "viewableNodes"}
            }' 
            row-transform='{
                "div": [{}, {}, {"data-path": "path", "style": "marginStyle"}],
                "label": "name",
                "expanderParts": [true, {"if": "open"}, ["-"], ["+"]],
                "button": [{}, {}, {"data-children": "hasChildren"}]
            }'
            be-channeling='[
                {
                    "eventFilter": "click",
                    "toNearestUpMatch": "xtal-tree",
                    "prop": "toggledNodePath",
                    "vfe": "path.0.parentElement.dataset.path",
                    "composedPathMatch": "button"
                },
                {
                    "eventFilter": "click",
                    "toNearestUpMatch": "xtal-slice",
                    "prop": "newSlicePath",
                    "vfe": "path.0.parentElement.dataset.path",
                    "composedPathMatch": "label"
                }
            ]'
        >
            <slot></slot>
        </xtal-vlist>
    </template>
</slot>

<slot name=sliced-vlist-row-template  be-transplanted>
    <template>
        <div class=node slot=row itemscope >
            <button class="expander" part=expander>.</button>
            <label></label>
        </div>
    </template>
</slot>

<slot name=sliced-vlist-style  be-transplanted>
    <template>
        <style>
            button.expander{
                display:none;
            }
            button[data-children].expander{
                display:inline;
            }
        </style>
    </template>
</slot>

<slot name=main-grid-header be-transplanted>
    <template>
        <header>
            <span>First Name</span><span>Last Name</span>
        </header>
    </template>
</slot>

<slot name=main-grid-vlist be-transplanted>
    <template>
        <xtal-vlist style="height:600px;width:100%;" class='slice-view' -list row-transform='{
            ".first_name": "first_name",
            ".last_name": "last_name"
        }'>
            <slot><slot>
        </xtal-vlist>
    </template>
</slot>

<slot name=main-grid-vlist-style be-transplanted>
    <template>
        <style>
            span{
                color:green;
            }
            div[slot="row"]{
                display:flex;
                flex-direction: row;
                justify-content: space-between;
            }
        </style>
    </template>
</slot>

<slot name=main-grid-vlist-row-template>
    <template>
        <div slot='row'>
            <span class='first_name'></span><span class='last_name'></span>
        </div>
    </template>
</slot>

<template be-assembling>
</template>
```
