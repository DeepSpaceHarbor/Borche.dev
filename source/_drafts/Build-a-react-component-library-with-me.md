title: Build a react component library with me
featured_image: files/images/featured/ui.jpg
summary: Build a react component library with me
tags: []
categories: []
date: 2023-09-08 05:51:27
---
A component library is a set of components, typically containing parts of the user interface shared across multiple apps. I like them because they allow the team to think about the future and focus on the small details and interactions throughout the different apps. From business perspective, you should probably have a component library when:
- The same design components library is used in multiple apps.
- The user interface is constantly evolving.
- Time to market is priority. A well developed library comes with a docs page, which reduces the discovery process when building new features.
- You find yourself repeating the code for similar but slightly different components.

## Tech stack
tl;dr:
- [Vite](https://vitejs.dev/)
- [TypeScript](https://www.typescriptlang.org/)
- CSS Modules
- [Storybook](https://storybook.js.org/)
- [Vitest](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- A small army of utilities that make my life easier.

## Development environment
Start by creating a new vite project with react and typescript.

```bash
$ npm create vite@latest UIVanguard -- --template react-ts
```
The contents of the public folder and most of the content in the src folder can be removed. The demo for this blog post will be a toy component called Greeting. It should accept a name as prop and display Hello name!
Here's how my code looks like:
```tsx
//📁/src/Greeting/Greeting.tsx
export type GreetingProps = {
  name: string;
};

export function Greeting({ name }: GreetingProps) {
  return <h1>Hello {name}!</h1>;
}
```
Next step is to install storybook. Make sure to answer yes when asked about the installation of relevant plugins such as eslint.
```bash
$ npx storybook@latest init
```
My code for the Greeting component story looks like this:
```tsx
//📁/src/Greeting/Greeting.stories.ts
import type { Meta, StoryObj } from "@storybook/react";
import { Greeting } from "./Greeting";

const meta = {
  title: "Components/Greeting",
  component: Greeting,
  tags: ["autodocs"],
  argTypes: {
    name: { control: "text" },
  },
} satisfies Meta<typeof Greeting>;

export default meta;

type Story = StoryObj<typeof meta>;

export const HelloWorld: Story = {
  args: {
    name: "World",
  },
};
```
## Styling
Vite has [support for CSS modules](https://vitejs.dev/guide/features.html#css-modules). Any CSS file ending with .module.css is considered a CSS modules file. 
For our toy component, I've decided to change the color and the font style.
```CSS
/*
📁/src/Greeting/Greeting.module.css
*/
.hello {
  color: purple;
  font-style: italic;
}
```
```tsx
//📁/src/Greeting/Greeting.tsx
import classes from "./Greeting.module.css";

export type GreetingProps = {
  name: string;
};

export function Greeting({ name }: GreetingProps) {
  return <h1 className={classes.hello}>Hello {name}!</h1>;
}
```
CSS Modules don't come with the type safety that many of us take for granted. The workaround for that is to use [typed-css-modules](https://github.com/Quramy/typed-css-modules) to automatically generate the types and [typescript-plugin-css-modules](https://github.com/mrmckeb/typescript-plugin-css-modules) to bring those types into our editor. Start by installing these 2 packages. I'll show you the configuration when you reach the configure section of this blog post.
```bash
$ npm install -D typed-css-modules typescript-plugin-css-modules
```
## Testing
Start by installing vitest, react testing library and other dependencies needed to make them work together.
```bash
$ npm install -D vitest @vitest/coverage-istanbul jsdom @testing-library/react @testing-library/jest-dom
```
Next, create the tests setup file `./tests/setup.ts`
```TypeScript
//📁/tests/setup.ts
import { afterEach } from "vitest";
import { cleanup } from "@testing-library/react";
import "@testing-library/jest-dom/vitest";

// runs a cleanup after each test case (e.g. clearing jsdom)
afterEach(() => {
  cleanup();
});
```
and update the vite config with the information about unit tests.
```JavaScript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./tests/setup.ts"],
    globals: true,
    coverage: {
      enabled: true,
      provider: "istanbul",
      reporter: ["text", "json", "html"],
    },
  },
});
```
Now you're ready to write your test. 
Here's what I wrote for the Greeting component.
```tsx
//📁/src/Greeting/Greeting.test.tsx
import { render } from "@testing-library/react";
import { Greeting } from "./Greeting";
import { it, describe, expect } from "vitest";

describe("Greeting", () => {
  it("Renders greeting", () => {
    const container = document.createElement("div");
    render(<Greeting name="Borche" />, {
      container: container,
    });
    expect(container).toMatchSnapshot();
  });
});
```
## Lint, format, configure, etc


## Publishing
## Next steps