# react-static using yarn workspaces does not get any Head

When using [yarn workspaces](https://yarnpkg.com/en/docs/workspaces) the `react-static` build does not output expected results from `<Head/>` usage.

https://github.com/nozzle/react-static/issues/762

## How to reproduce

Update to latest `react-static` cli

````shell
yarn global add react-static
````

### First check it actually works outside a workspace

_Using this repo? `git checkout tags/step1` and skip to "Replace the" else_

Create a new workspace folder and add a new `react-static` project in that folder

````shell

mkdir workspace && cd workspace

react-static -n client

# enter the generated react-static folder
cd client

````

Replace the `src/containers/404.js` with some `Head`

````jsx
import React from 'react'
import { Head } from 'react-static'

export default () => (
  <div>
    <Head>
      <title>404</title>
    </Head>
    <h1>404 - Oh no's! We couldn't find that page :(</h1>
  </div>
)

````

Then run

    yarn build

Now check the generated file `dist/404.html` you can see that it got `Head` (the `<title/>` is `404`)

````shell
grep "404</tit" dist/404.html
````

### Now create the workspace and check if it still works

````shell
# still in client folder so go back to the workspace
cd ..
````

_Using this repo? `git checkout tags/step2` and skip to "Then run" else_

````shell
yarn init --yes --private
````

Then edit the generated `package.json` and add the following lines

    "workspaces": ["client"],
    "scripts": { "build": "cd client && yarn run build" }

Then run

    yarn

Then run

    yarn run build

Now check the generated file `client/dist/404.html` you can see that it did not get `Head`. There is no `<title/>`

````shell
grep "404</tit" client/dist/404.html
````

#### Expected result

To get some `Head`

````html
<title data-react-helmet="true">404</title>
````

#### Actual result

Did not get any `Head` there is no `<title/>`

## Possible root cause

`react-static` or upstream packages depend on their packages physically available within the _cwd_ `node_modules` ?

### Known yarn workspaces caveats

> The package layout will be different between your workspace and what your users will get (the workspace dependencies will be hoisted higher into the filesystem hierarchy). Making assumptions about this layout was already hazardous since the hoisting process is not standardized, so theoretically nothing new here. If you encounter issues, try using the nohoist option

Source https://yarnpkg.com/en/docs/workspaces#toc-limitations-caveats

## Workaround

Use the yarn workspaces [`nohoist`](https://yarnpkg.com/blog/2018/02/15/nohoist/) option.

First remove all `node_modules` from the workspace

````shell
rm -rf node_modules */node_modules
````

_Using this repo? `git checkout tags/step3` else_

Edit the workspace `package.json` and change the `workspaces` property to

    "workspaces": {
        "packages": ["client"],
        "nohoist": ["**/react-static", "**/react-static/**"],
    },

_Note: The above `nohoist` should just be `["client/react-static", "client/react-static/**"]` though does not seem to work ... ?_

Then run

    yarn

Then run

    yarn run build

Now check the generated file `client/dist/404.html` you can see that it now gets `Head`.

````shell
grep "404</tit" client/dist/404.html
````

wahey.. finally got some head :-O
