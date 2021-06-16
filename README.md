# React Start New App Explanation

## Description

So you need to start a new react project from scratch. And of course you want to make your work convenient for you. You want to reduce the number of mistakes made. You want to work in team with your collegues and make it be convenient also. It means you have to have the same code style, the same logic in the project and more. When the project will be in production you will need to support it. And you would also like it to be convenient.
The good thing is that there are lot's of tools in frontend world that help it all happen.
So let's see what we can do.

We will cover in that lesson: 
- start new project
- add Typescript support
- add storybooks to the project to have ui documentation and to make it possible to develop ui separately
- add typescipt support to storybooks
- adding aliases to the project and storybooks to get rid of import paths like `../../../../components/Button`
- add linters
- add precommit hooks to force all the team use:
  - linters
  - tests
  - storybooks

## Start a new React app named preparation with typescript support

```npx create-react-app preparation --typescript```

> You can read more on [create-react-app]

[create-react-app]: https://facebook.github.io/create-react-app/

As you see we start the project with Typescipt support. You can remove `--typescript` and get the project without ts. You can still follow the next instructions. But development of a project without ts is strongly not recommended as it leads to a plenty of bugs and reduces code readability and makes it difficult to support the project in the future.


So now we have the project, and it works! We can see the files of the project if we `cd` to the project's folder

```cd preparation```

Here we can see all the files of the project

 We can test it running 

```npm start```


## Make the project **configurable**

Well you can do nothing more and rely on `create-react-app` updates in future. 
But the config provided with `create-react-app` is not enough for convenient development.
I would like to show you how to make it more convenient.
So we need to see webpack config and other configs. But there are no such files in the project.
Well that is because they are hidden from us.
To see and configure that files run


```npm run eject```

Ok, now we have `config` and `scripts` folders in the root of the project, where all the configuration files can be found.

We will use that files while configuring the project next.


## Add storybook

So what now?
It is going to be a good idea to develop ui components isolated from architecture and from the need to always run `npm start` and place all the components to layout. Even if there are not many components yet. And it would defenetely be a good idea to have some documentation on how to use your components.

Here is what we can do about it:

```npx -p @storybook/cli sb init --type react```

> You can read more on [storybook]

[storybook]: https://storybook.js.org

We got two new folders in the root of out project: **stories** and **.storybook**.
By default **.storybook** folder contains all the storybook configurations.
In the **stories** folder you will find examples of storybook usage.
And you have to place your stories into that folder by default.

If you look into `package.json` you will find some new scripts in the `scripts` section.

To run storybooks:

`npm run storybook`

### Add typescript support to storybooks

Cool. Now let's create out own component and a story for the component.

We have to create a stories file in the `stories` folder as you remember.

Let's create our TestComponent in  `src/TestComponent.tsx`:

```ts
import React from 'react'

export const TestComponent = () => <div>Hello, world!</div>

```

And add a story for it in out  `stories` folder `stories/TestComponent.stories.js`

```js
import React from 'react';

import { storiesOf } from '@storybook/react';

import {TestComponent} from '../src/TestComponent'

storiesOf('Welcome', module).add('TestComponent', () => <TestComponent />);

```

Then run 

`npm run storybook`

and you will see that storybooks can not create story for out component.
That is because by default storybook configured to work with JavaScript only, not with TypeScript.
Let's see how we can add ts support to storybooks.

So in ./.storybook/config.js we need to change to be like that:

```js
import { configure } from '@storybook/react';

// automatically import all files ending in *.stories.js
const req = require.context('../src', true, /\.stories\.(js|ts|tsx)$/);
function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);

```

Pay attention to `const req` - the context is set to be the src folder and files can end with js or ts or tsx

So now storybook will look for files that end with .stories.(js|ts|tsx) in out `src` folder. I suppose that the story of a component should be places near that component as it works as a documentation and sometimes when you change your component you will have to change it's story as well - very convenient when they are placed near.

So move the file `stories/TestComponent.stories.js` to `src/TestComponent.stories.tsx'
Pay attection to **.tsx**

Run again 

`npm run storybook`

And still we can not see the story.

Now the files found, but storybook can not show the stories as it can not parse ts yet.


Well, let's correct that.

Now create a `webpack.config.js` in ./.storybook with the next content: 

```js
module.exports = ({ config, mode }) => {
  config.module.rules.push({
    test: /\.(ts|tsx)$/,
    loader: require.resolve('babel-loader'),
    options: {
      presets: [['react-app', { flow: false, typescript: true }]],
    },
  });
  config.resolve.extensions.push('.ts', '.tsx');
  return config;
};
```

Now add some changes to `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "sourceMap": true,
    "jsx": "react",
     "baseUrl": "src",
     "rootDir": "src"
  },
  "include": [
    "src"
  ],
   "exclude": ["node_modules", "build", "scripts"]
}
```

Well, now we can see the story of out componet.
But still there is a problem.
If you open `src/TestComponent.stories.tsx` you will see that `@storybook/react` has some error. It is because we did not install types for storybook.
Let's do it:

```
npm i @types/storybook__react --save-dev
```

Now it works correct. 

## Add jest support to storybooks

So now we have storybooks working. But sometimes while developing component you can broken some other component. Or sometimes you change some logic in your helper functions and run tests. Tests pass and you think it ok. I will commit my changes. 
And you do not even know that some story is now broken. Then other developer will be very surprised.

That is why it is a good idea to make `jest` check if storybooks work fine.

Let's see how to do it:

`npm i --save-dev @storybook/addon-storyshots @types/storybook__addon-storyshots`

> You can read more on [@storybook/addon-storyshots]

[@storybook/addon-storyshots]:  https://www.npmjs.com/package/@storybook/addon-storyshots

Now we have to add one more package:

`npm i --save-dev require-context.macro`

and make some changes to `./.storybook/config.js`

```js
import { configure } from '@storybook/react';
import requireContext from 'require-context.macro';
 
// const req = require.context('../stories', true, /\.stories\.js$/); <-- replaced
const req = requireContext('../src', true, /\.stories\.(js|ts|tsx)$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);

```

**Pay attention to folder path and file extensions!**

Now we will need another package:

`npm i --save-dev react-test-renderer`

Now it is time to add a configuration file for all that stuff.
We have to create it in our src folder as jest looks through that folder.

Create file `src/storyshots.test.ts` with the following content:

```ts
import initStoryshots from '@storybook/addon-storyshots';
 
initStoryshots({
  configPath: './.storybook'
});
```

## Add support for aliases across the project

Now I would like to show you the problem in projects. Sometimes you have import something from a folder that is not near you current file. And you can import like this `../../../src/constants`

I guess it really reduces readability and maintainability of the project.

So let's add aliases to the project to be able to do smth like `import {mytext} from 'constants'`

The main problem here is that aliases must be supported by typescript, storybooks, jest separately.

Let's create `src/context` folder and add files `index.ts` and 'constants.ts` to that folder.

Then we will `export const helloworld = 'helloworld'` from that folder. And  `import {helloworld} from 'context'` in out TestComponent and in TestComponent.stories.tsx.

We need to create `.babelrc` file in the root of the project with the following content:


```json
{
  "presets": ["@babel/preset-env","@babel/preset-react", "react-app"],
  "plugins": [
    [
      "module-resolver",
      {
        "root": [
          "./src"
        ],
        "alias": {
          "context": "./src/context",
        }
      }
    ]
  ]
}
```

Than we have to remove babel section from `package.json` completely.

Now let's install some packages:


`npm i --save-dev babel-plugin-module-resolver @babel/preset-env`

We also have to add in file `tsconfig.json` in section compilerOptions the following code :

```json
    "paths": {
      "src": [
        "src/"
      ],
      "context": [
        "context"
      ],
    },
```

Now let's update out story and our component:

story: 

```tsx
import React from 'react';

import { storiesOf } from '@storybook/react';
import { TestComponent } from './TestComponent';
import { helloworld } from 'context';



storiesOf('TestComponent', module)
  .add('test', () => <TestComponent>{`${helloworld} from storybook`}</TestComponent>)
  .add('test with defaultText', () => <TestComponent/>);

```

component:

```tsx
import { helloworld } from 'context';
import React from 'react';

interface IProps {
  children?: string;
}

export const TestComponent = ({ children }: IProps) => <div>{children ? children : helloworld}</div>
```

Let's create a new test file `first.test.ts` in `src` with the following content:

```ts
import { helloworld } from 'context';

describe('test1', () => {
  it('should pass', () => {
    expect(helloworld).toEqual(helloworld);
  });
});

```

Than run

`npm test`

If you changed something in your story than you will get error that your snapshots do not match.
Let's add new script to **scripts** section of `package.json` that will run tests and update snapshots.

```json
"test:u": "node scripts/test.js --no-watch -u"
```

## Installing packages to use **precommit hooks**

At that moment we have everything working, but there is one last problem.
How can we be sure, that every developer in out team runs tests and lints the code before pushing it to the repo?
Now we can not be sure.
Let's fix it as well.


First of all we need to make this project a git repo running `git init` and install some packages:

```
npm install husky --save-dev
npm i tslint tslint-react ts-loader prettier tslint-config-prettier lint-staged --save-dev

```

Than add correct configurations to `package.json`

```json
  "husky": {
    "hooks": {
      "pre-commit": "npm run precommit"
    }
  },
  "lint-staged": {
    "*.tsx": [
      "npm run lint:fix",
      "git add"
    ],
    "*.ts": [
      "npm run lint:fix",
      "git add"
    ]
  },
  "prettier": {
    "singleQuote": true,
    "arrowParens": "always"
  }

```

Add command to scripts section of `package.json`

```json
  "precommit": "npx lint-staged && npm test",
  "lint": "tsc --noEmit && tslint --project .",
  "test": "node scripts/test.js --no-watch",
  "lint:fix": "npx tslint --project .",
  "prettier:ts": "prettier --write './src/**/*.ts'",
  "prettier:tsx": "prettier --write './src/**/*.tsx'
```

Now let's 

```bash
git add .
git commit -m "Initial commit"
```

You can see that many things are done before commit. 
And if your linters or tests fail - than `git commit` command will fail as well. 
And you will have to fix the problems found berfore proceeding.


# Bonus

## Add styled component to the project

```npm install --save styled-components @types/styled-components```

> You can read more on [styled components]

[styled components]: https://www.styled-components.com/

Than let's add one styled component

```tsx
import { helloworld } from 'context';
import React from 'react';
import styled from 'styled-components';

interface IProps {
  children?: string;
}


const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
`;

export const TestComponent = ({ children }: IProps) => 
  <>
    <div>{children ? children : helloworld}</div>
    <Button>Styled Button</Button>
  </>;

```

and see how it looks in your storybooks

```npm run storybook```

Now update your snapshots 

```npm run test:u```

and commit your changes 

```git add . && git commit -m "add styled components"```

# Conclusion

As you see there are many tools in frontend world to make your work easier and more secure.
Of course there will be updates to the packages I used so look through their docs while setting up a new project.
