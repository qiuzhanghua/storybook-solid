# Storybook for Solid-js example

This repo is the example of adoption storybook for solid-js.

Thanks to guys from this thread: https://github.com/solidjs/solid-docs-next/issues/35

## Instructions

### Storybook v7

Let's assume that you've migrated to storybook v7.

You need to change the following files.

**1. .storybook/main.js**

```js
module.exports = {
  stories: ["../src/**/*.mdx", "../src/**/*.stories.@(js|jsx|ts|tsx)"],
  addons: [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
  ],
  framework: {
    name: "@storybook/html-vite",
    options: {},
  },
  docs: {
    autodocs: "tag",
  },
};
```

**2. .storybook/preview.js**

If you want HMR works for solid you need to add `/* @refresh reload */` to the beginning of this file however it's not the only change.
See the details below.

```js
/* @refresh reload */
/**
 * Don't forget the line above for HMR!
 * 
 * Note: for some reason HMR breaks if you change .stories file,
 * however reloading the page fixes this issue
 */ 

import { render } from "solid-js/web";

let disposeStory;

export const decorators = [
  (Story) => {
    disposeStory?.();

    const solidRoot = document.createElement("div");

    disposeStory = render(Story, solidRoot);

    return solidRoot;
  },
];

/** Autogenerated by Storybook */
export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};

```

**That's it!**

#### HMR

To make HMR work for your component you need to render it as JSX:

```js
// Correct! HMR works!
// Let's assume that this is storybook meta object
export default {
  // ...
  render: (props) => <Counter {...props} />,
  // ...
} as Meta<ComponentProps<typeof Counter>>;
```

If you write the code like this, it won't work:

```js
// Wrong! HMR doesn't work!
// Let's assume that this is storybook meta object
export default {
  // ...
  render: Counter,
  // ...
} as Meta<ComponentProps<typeof Counter>>;
```

Here's an example story for `Counter` component.

```js
import { Counter, CounterProps } from "../Counter";

import type { Meta, StoryObj } from "@storybook/html";
import type { ComponentProps } from "solid-js";

type Story = StoryObj<CounterProps>;

export const Default: Story = {
  args: {
    initialValue: 12,
    theme: "default",
  },
};

export default {
  title: "Example/Counter",
  tags: ["autodocs"],
  /**
   * Here you need to render JSX for HMR work!
   *
   * render: Counter won't trigger HMR updates
   */
  render: (props) => <Counter {...props} />,
  argTypes: {
    initialValue: { control: "number" },
    theme: {
      options: ["default", "red"],
      control: { type: "radio" },
    },
  },
} as Meta<ComponentProps<typeof Counter>>;

```

### Storybook v6

To see the files for the storybook v6 see [THIS](https://github.com/elite174/storybook-solid-js/tree/16466f35def5ebe4b28603211d8d825d690fbe40).

#### 1. Initialize storybook in your repo as [html project](https://storybook.js.org/docs/html/get-started/install):

```
npx storybook init --type html
```

#### 2. Update storybook config files in `.storybook` folder as follows:

*main.js*

Add `vite-plugin-solid` to storybook config. 
```js
const Solid = require("vite-plugin-solid");

module.exports = {
  stories: ["../src/**/*.stories.mdx", "../src/**/*.stories.@(js|jsx|ts|tsx)"],
  addons: [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
  ],
  framework: "@storybook/html",
  core: {
    builder: "@storybook/builder-vite",
  },
  features: {
    storyStoreV7: true,
  },
  
  // Add solid plugin here
  async viteFinal(config, { configType }) {
    config.plugins.unshift(Solid({ hot: false }));

    return config;
  },
};
```

*preview.js*

Customize your `preview.js` file as follows.

```js
import { render } from "solid-js/web";

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};

let disposeStory;

export const decorators = [
  (Story) => {
    if (disposeStory) {
      disposeStory();
    }

    const root = document.getElementById("root");
    const solidRoot = document.createElement("div");

    solidRoot.setAttribute("id", "solid-root");
    root.appendChild(solidRoot);

    disposeStory = render(Story, solidRoot);

    return solidRoot;
    // return createRoot(() => Story()); // do not work correctly https://github.com/solidjs/solid/issues/553
  },
];
```

## Comments

- _.npmrc_ file is necessary because I use npm v8, however storybook doesn't support it, and that's why it requires this file.
