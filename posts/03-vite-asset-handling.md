# Vite Asset Handling?

## Episode TLDR;

- Vite can handle assets (`.png`, `.css`, `.sass`, etc.) by default
- Since Vite can do this, we no longer need `server/public` to store assets
- To use assets, we can use the `import` synax:
  
```jsx
// import an image url
import logo from './assets/logo.png'
// import css
import './assets/style.css'

// ...
function App() {
  return (
    <img src={logo} alt="logo" />
  )
}
```
- Or, we can use `public` as a directory at the root of the project to store assets and use them like so:

```jsx
// public/logo.png is a file
function App() {
  return (
    <img src="/logo.png" alt="logo" />
  )
}
```

Vite automatically recognises this directory as containing static assets and will serve them as such.

## Vite

There are several reasons we're choosing to use Vite over a tool like Webpack, but they boil down to:

- **Vite is a low config build tool**

Webpack is incredibly extensible, but it comes packaged relatively unopinionated... which means that we (staff _and_ students) need to configure it to extend its functionality if we want to do fancy things (SASS, CSS Modules, Tailwind, `.env` variables, `.png`s, `.svg`s, etc.).

Vite comes pre-configured with all of these things, and to extend it further is relatively straightforward with a community of plugins.

In fact, this is all the config students are going to see in Week 4:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()]
})
```

- **Vite is fast**

While it doesn't matter dramatically at this scale of our challenges, Vite is considered much faster than Webpack. This is because Vite uses ES Modules to load files, which means that it doesn't need to bundle all of our files together to serve them to the browser when running in `development`.

- **Vite is gaining traction**

If a student were to pick up another front-end framework, they'd likely be using Vite. It's the build tool of many non-React frameworks: Vue, SvelteKit, Astro, SolidStart, and more.

## How does it work in development?

To talk about some of the benefits of Vite, we need to understand how it works in development.

### Vite's Dev Server

Vite's dev server is a Node server that runs in the background. It's job is to serve our files to the browser, and it does this by using ES Modules.

When running, it does not need to bundle the files together, but it does transpile them using `esbuild`/`Babel`.

Because of the presence of a dedicated dev server, we do not _build_ our code and then serve it via `server/public/index.html`. Instead, we run the dev server and it serves our code directly. The default port for Vite is `5173`, which is L337 for 'Vite' 🤯.

## Vite Asset Handling

Vite comes pre-configured with good defaults, one of them being asset handling. This means that we can import assets like `.png`s, `.css`, `.sass`, etc. and Vite understands those file extensions and can use them without much hassle.

### Where do assets go now?

In `server/public`...?

One of the more confusing mental models of our stack is how asset handling works in general.

I write a component in `client/components/Counter.tsx` and add `className="counter__default"` (or however I write BEM 🤷), but where does the class actually live? Go two folders back, take a left _out_ of the `client` directory, head down until you see `server`, and then, travel on into `public/main.css`. Confusing right?

With Vite, we can write a file `client/components/counter.css`, add our class, and then inside `client/components/Counter.tsx`, write:

```tsx
import './counter.css'
```

And that will import the CSS for me so that I can use my `counter__default` class, and the code for that lives right there.

I can go one step further and decide to start using SASS, change the file name to `counter.scss`, and Vite manages that for me.

I can go another step further and decide that I want to apply styles directly

```css
/* counter.module.scss */
h1 {
  margin-left: 1rem;
}
```

and use CSS Modules. Again, Vite manages that for me.


### Death to `server/public`

Now that we can write, import, and use assets in the client side, we no longer need `server/public`.

**And**, because Vite has its own dedicated Dev Server  our `client` code and `server` code are *decoupled* and can be run separately in development.

In week 4 you'll notice the challenges don't even have a `server` directory. Hopefully, this will simplify the mental model for where code is running and why it needs to run there (React stuff runs and is served in Vite/browser land, Express stuff runs in and is served in Node land).

## Cheatsheet and conventions

### CSS:

**Convention:** For global styling we will write CSS files in `client/styles/main.css` or `client/styles/<your-file-name>.css` and import them in `client/index.tsx`

```tsx
// client/index.tsx
import './styles/main.css'
```

However, it's completely possible and reasonable to co-locate local styles with components:

```css
/* client/components/Counter/index.css */
h1 {
  text-underline: red;
}
```

```tsx
// client/components/Counter/index.tsx
import './index.css'
```

### SASS:

```sh
npm install sass
```

Rename your CSS files to `.scss` and you're good to go and use SASS.

The same conventions apply for SASS as they do for CSS.

### CSS Modules:

Rename your CSS files to `.module.(s)css` (include the `s` for SASS) and now you can import your styles as an object:

CSS Modules are a way to scope CSS to a component. This means that you can write a class name like `counter` and it will only apply to the component you're writing it in.

```css
/* client/components/Counter/index.module.scss */
.counter {
  margin-left: 1rem;

  &:hover {
    color: red;
  }
}
```

```tsx
// client/components/Counter/index.tsx
import styles from './index.module.scss'

function Counter() {
  return (
    <div className={styles.counter}>Counter</div>
  )
}
```

### Tailwind:

Follow their [installation guide](https://tailwindcss.com/docs/guides/vite) (skipping step 1.), it's relatively straightforward (especially compared to Webpack).

### Images:

Images can be co-located with their components, however, it may make more sense to have a `client/assets` directory and import from there. Or to use a `public` directory at the root of the project.

```tsx
// client/components/Counter/index.tsx
import logo from './logo.png'

function Counter() {
  return (
    <img src={logo} alt="logo" />
  )
}
```

### Environment Variables:

Vite comes with a built-in way to use environment variables. You can read more about it [here](https://vitejs.dev/guide/env-and-mode.html#env-files).

Essentially, any environment variable prefixed with `VITE_` will be available in the browser. This could be useful for things like public client keys, root URLs, etc.

```sh
VITE_API_KEY=1234
```

```tsx
// client/components/Counter/index.tsx
function Counter() {
  return (
    <div>{import.meta.env.VITE_API_KEY}</div>
  )
}
```