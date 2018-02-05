# Frame Component

An ultra light typescript library for writing custom elements.

### Ultra light
Less than 1.5kb (gzipped) when using all decorators in a custom element.

### Simple, yet powerful
This framework utilizes only typescript, and has no dependency on any CLI or specific build tooling.

### A library, not a framework
This library aims to ease the developer experience when writing custom elements, and to fit in to any build system.

## Installing
Install from NPM:
```sh
npm install @framejs/component
```

## Decorators

### @Component({tag: string, style?: string})
The main decorator that holds state provides a renderer (this is needed in order to use the rest of the decorators).

To manually run the renderer use: `this._invalidate();`

To auto-render on `@Attr` and `@Prop` changes set `this._renderOnPropertyChange = true`.
This should only be done with a smart renderer function. it's enabled by default when extending LitElement.

```ts
import { Component } from '@framejs/component';

@Component({
    tag: 'my-element',
    style: ':host { color: blue; }'
})
class MyElement extends HTMLElement {
    render() {
        return `Hello World!`;
    }
}
```

### @Attr() [property]: string | boolean | number
Decorates the element with an attribute setter and getter and updates state/render on change. Updating the property from within the element or externally will update the attribute in the rendered HTML and the other way around.

Providing a default value will set the attribute when the element is ready. If the attribute is already set by the user, the default will be overwritten.

```ts
import { Component, Attr } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Attr() target: string = 'World!'

    render() {
        return `Hello ${this.target}`;
    }
}
```

### @Prop() [property]: any
Decorates the element with a property setter and getter and updates state/render on change.
This value will not be reflected in the rendered HTML as an attribute.

```ts
import { Component, Prop } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Prop() data: string[] = ['Hello', 'world!'];

    render() {
        return `
            ${data.map(word => {
                return word;
            }).join(' ')}
        `;
    }
}
```

### @Watch(property: string) Function(oldValue: any, newValue: any)
The function provided will get triggered when the property changes with the old and new value.

```ts
import { Component, Prop } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Prop() data: string[] = ['Hello', 'world!'];

    @Watch('data')
    dataChangedHandler(oldValue, newValue) {
        // Do something with the new data entry
    }

    render() {
        return `
            ${data.map(word => {
                return word;
            }).join(' ')}
        `;
    }
}
```

### @Event() [property]: EventEmitter
Creates a simple event emitter.

```ts
import { Component, Emit, EventEmitter } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Emit() isReady: EventEmitter;

    connectedCallback() {
        this.isReady.emit('my-element is ready!')
    }
}
```

### @Listen(event: string, target?: window | document) Function
Listens for events and executes the nested logic.

```ts
import { Component, Listen } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Listen('click')
    clickedOnInstanceHandler(event) {
        console.log(event)
    }

    @Listen('resize', window)
    windowResizeHandler(event) {
        console.log(event)
    }
}
```

It's also possible to listen for events from child elements

```ts
import { Component, Listen } from '@framejs/component';
import './my-other-element';

@Component({
    tag: 'my-element'
})
class MyElement extends HTMLElement {
    @Listen('onOtherElementClicked')
    onOtherElementClickedHandler(event) {
        console.log(event)
    }

    render() {
        return `
            // my-other-element emits an customEvent called 'onOtherElementClicked'.
            <my-other-element></my-other-element>
        `;
    }
}
```


## Using lit-html element
`lit-html` is a great templating extension when working with complex components.
Read more about [lit-html](https://github.com/Polymer/lit-html).

Extend `LitElement` instead of `HTMLElement` to get all it offers.

> It's important to use `html` string literal function as it converts the literal to lit-html.

```ts
import { Component, LitElement, html } from '@framejs/component';

@Component({
    tag: 'my-element'
})
class MyElement extends LitElement {
    render() {
        return html`I\m so lit!`;
    }
}
```

## Extendable renderer
The built in renderer is very simple: it receives the returned value, and replaces innerHTML with the new template when updated.

This example shows how `LitElement` is written.

```ts
import { render } from 'lit-html/lib/lit-extended';

export class LitElement extends HTMLElement {
    // Set _renderOnPropertyChange if the renderer
    // should render on every property change.
    public _renderOnPropertyChange = true;

    renderer(template) {
        render(template(), this.shadowRoot);
    }
}
```

Inside your element you can use it like this:

```ts
import { Component, Prop } from '@framejs/component';
import { html } from 'lit-html/lib/lit-exteded';

@Component({
    tag: 'my-element'
})
class MyElement extends LitElement {
    @Prop() message: string = 'it\'s lit!';

    render() {
        return html`${this.message}`;
    }
}
```
