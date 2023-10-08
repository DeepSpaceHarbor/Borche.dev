title: Type safety for CSS classes
featured_image: files/images/featured/typed-css.jpg
summary: >-
  Did you know that class names in your CSS/SCSS/LESS files can become types?
  Now you do. It takes less than 15 minutes to implement and they will prevent
  all of the embarrassing mistakes on your next refactor!
tags: []
categories: []
date: 2023-10-08 00:00:00
---
Catching issues as early as possible is essential for the developer experience. But browsers and many popular tools will happily ignore invalid properties, usage of non-existing classes, etc.  Let's explore how you can improve that and create a better developer experience for your project.

#### tl;dr?
 - [CSS-in-JS](#CSS-in-JS)
 - [CSS modules](#CSS-modules)
 - [Sass/SCSS](#Sass-SCSS)
 - [LESS](#LESS)
---
## CSS-in-JS
The CSS-in-JS approach tends to be typesafe by default. 
```tsx
// This is type-safe CSS. It's a CSS object that you can use in your components.
const helloWorldStyle: React.CSSProperties = {
  color: "purple",
};

export function HelloWorld() {
  return <div style={helloWorldStyle}> Hello world! </div>;
}
```

```tsx
export function HelloWorld() {
  return (
    <div
      style={{
        // This is also type-safe CSS.
        color: "purple",
      }}>
      Hello world!
    </div>
  );
}
```

## CSS modules
CSS modules are regular CSS files with limited scope. However, they lack types when used in a component. It becomes a problem when classes are renamed or removed.
You could have a CSS module like this:
```css
/* 📂/HelloWorld.module.css */
.helloWorld {
    color: purple
}
```
and use it (the wrong way) in a component like this:
```tsx
// 📂/HelloWorld.tsx
import styles from "./HelloWorld.module.css";
export function HelloWorld() {
  return <div className={styles.helloWorld123}>Hello world!</div>;
}
```
The example uses wrong classname, but our IDE, the browser and all other tooling stays silent. One of the ways to get a validation for the class names is to generate types with the help of [typed-css-modules](https://github.com/Quramy/typed-css-modules) and get autocomplete in our IDE with [typescript-plugin-css-modules](https://github.com/mrmckeb/typescript-plugin-css-modules). Because these are CLI tools, I like to use [npm-run-all](https://www.npmjs.com/package/npm-run-all) and run them at development time.
Start by installing the dependencies:
```bash
$ npm install -D typed-css-modules typescript-plugin-css-modules npm-run-all
```
Next, update the scripts in your package.json file to start the development server and css modules watcher at the same time.
```json
 "scripts": {
    "dev:start": "vite",
    "dev:css": "tcm src --watch",
    "dev": "run-p dev:**",
  }
```
TCM will run in the background and automatically generate types when it detects changes in the CSS module files. This is the generated types file from my CSS modules example:
```TypeScript
declare const styles: {
  readonly "helloWorld": string;
};
export = styles;
```
To enable autocomplete in your IDE add css modules plugin to `tsconfig.json`.
```json
{
  "compilerOptions": {
    "plugins": [{ "name": "typescript-plugin-css-modules" }]
  }
}
```
## Sass/SCSS
For SCSS we can use [typed-scss-modules](https://github.com/skovy/typed-scss-modules) for type generation and [npm-run-all](https://www.npmjs.com/package/npm-run-all) to generate them during development time.
Start by installing the dependencies:
```bash
$ npm install -D typed-scss-modules npm-run-all
```
Next, update the scripts in your package.json file to start the development server and scss files watcher at the same time.
```json
 "scripts": {
    "dev:start": "vite",
    "dev:scss": "typed-scss-modules src --watch --exportType default",
    "dev": "run-p dev:**",
  }
```

This is my sample scss file:
```scss
// 📂/HelloWorld.scss
.helloWorld {
    color: purple;
  }
```
and these are the generated types:
```TypeScript
export type Styles = {
  'helloWorld': string;
};

export type ClassNames = keyof Styles;

declare const styles: Styles;

export default styles;
```

When used in a component, you will get autocomplete for free.
```tsx
// 📂/HelloWorld.tsx
import styles from "./HelloWorld.scss";
export function HelloWorld() {
  return <div className={styles.helloWorld}>Hello world!</div>;
}
```
## LESS
When it comes to LESS we can use [typed-less-modules](https://github.com/qiniu/typed-less-modules) to generate the types and [npm-run-all](https://www.npmjs.com/package/npm-run-all) to keep the file watcher running during development time.
Start by installing the dependencies:
```bash
$ npm install -D typed-less-modules npm-run-all
```
Next, update the scripts in your package.json file to start the development server and LESS files watcher at the same time.
```json
 "scripts": {
    "dev:start": "vite",
     "dev:less": "tlm src --watch --exportType default",
    "dev": "run-p dev:**",
  }
```
This is my sample LESS file:
```LESS
// 📂/HelloWorld.less
.helloWorld {
    color: purple
}
```
and these are the generated types:
```TypeScript
export interface Styles {
  'helloWorld': string;
}

export type ClassNames = keyof Styles;

declare const styles: Styles;

export default styles;
```
When used in a component, you will get autocomplete for free.
```tsx
// 📂/HelloWorld.tsx
import styles from "./HelloWorld.less";
export function HelloWorld() {
  return <div className={styles.helloWorld}>Hello world!</div>;
}
```
## Next steps
If your project doesn't have types for the CSS classes, now is the time to add them! It takes less than 15 minutes to implement and they go a long way in preventing all of the silly mistakes in the next code refactor.