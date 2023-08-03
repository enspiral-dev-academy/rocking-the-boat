# About `typed-html`

This post is explores a new way of using `jsx` syntax to do server-side rendering as a replacement to `express-handlebars`.

## TL;DR

- `typed-html` allows the developer to use `jsx` syntax
- An excellent stepping stone for students to familiarise themselves with `jsx` before learning React
- Seamless switching between JS and HTML, for example, using array methods inside HTML
- No need to add middlewares to setup a templating engine

## Server-Side Rendering

Server-side rendering is gaining a lot of popularity and it's been adopted by many modern frameworks such as:

- Sveltekit
- AstroJS
- NextJs
- Nuxt

In the bootcamp, we teach this technique by introducing `express-handlebars` to render pages using its templating language and show dynamic content. Handlebars has it's own basic language to handle conditional rendering and iterating over collections and other things.
Many students shared their learning reflections after the end of the bootcamp and asked why learn handlebars for two weeks then replace it with React. Many of them, unfortunately, missed the point and that is not about learning handlebars but it's about learning Server-side Rendering as technique to build web apps.
However, I agree with the students to some extent. The DX and the syntax is old and we can do better than this. We can use `jsx` syntax when doing server-side rendering and re-use the cool features that comes with it for React, which brings me to `typed-html`

## `typed-html`

This package allows developers to write safe and typed `jsx` syntax in the backend as if you're writing React components.
Here is an example of how to respond with an html using `jsx` in express:

```js
// server.tsx

import express from "express";
import * elements from 'typed-html'

const server = express();

// jsx component
function About(props: Props) {
   return <h1>{props.title}</h1>
}

server.get('/about', (req, res) => {
    res.send(<About title="About Us" />)
})
```

Two things to notice here, the first is that the file name has `tsx` as the extension so that we can use `jsx` syntax. The second thing is that this example uses `res.send` to respond with a string. The cool thing about `typed-html` is that jsx returns a string, this is why we're using `res.send`
No need to configure an engine, middleware, or templates. Everything CAN be written in one file, but if you prefer to separate your components, this is how you would do it:

```js
// About.tsx

function About(props: Props) {
   return <h1>{props.title}</h1>
}
```

```js
import { About } from "./components/About";

server.get("/about", (req, res) => {
  res.send(<About title="About Us" />);
});
```

And this is it :smile:

## More Examples

```js
interface Props {
    fruits: {name: string, calories: number}[]
}

function FruitList(props: Props) {
   return (
     <ul>
         {props.fruits.map((fruit) => <li>{fruit.name (fruit.calories)}</li>)}
     </ul>
  )
}

server.get('/fruits', (req, res) => {
 const fruits = await db.getFruits()
 res.send(<FruitList fruits={fruits} />)
})

```

```js
import * as elements from "typed-html";

function Input(attributes: Attributes, value: string) {
  return <input {...attributes} value={value} type="text" class="bg-red-400" />;
}
```

You can also use the ternary operator to conditionally show and hide stuff just like how it's done it React and I'll leave this as an exercise for you

Check out [dreamfest](https://github.com/dev-academy-challenges/challenges/tree/typed-html/packages/dreamfest-solution) and see a full working solution

## Installation & Configuration

Now that I got you interested, here is how you install it and configure it

```
npm i typed-html
```

Then go to tsconfig.json and add these two lines:

```json
{
  "compilerOptions": {
    "jsx": "react",
    "jsxFactory": "elements.createElement"
  }
}
```

Now you can start using `jsx` for any file `tsx` files by importing `elements`

```js
import * as elements from "typed-html";
```

## Benefits

- Using `typed-html` should prepare the students to learn and get used to `jsx` syntax for two weeks before learning React
- Learn why learning `array` methods is useful when dealing with `jsx` components (loops and conditionals)
- After the students learning React, I expect them to actually see the difference between CSR and SSR
- `jsx` syntax is easy to work with compared to the build-in helpers that come with handlebars such as, `{{#each art}}{{/each}}`
- Thrown errors in handlebars are harder to debug compared to `typed-html`

## Update

Gerard suggested another interesting alternative to `typed-html` and that is to use `react-dom/server`

```diff
-import * as elements from "typed-html";
+import { renderToMarkup } from "react-dom/server";

server.get('/about', (req, res) => {
-    res.send(<About title="About Us" />)
+    res.send(renderToMarkup(<About title="About Us" />))
})
```

So instead of using `typed-html` to render the jsx, we can use `react-dom/server` to do the same thing. 
Very cool stuff, thanks Gerard :smile: