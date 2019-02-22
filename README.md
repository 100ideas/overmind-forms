# overmind-forms

## 21 Feb 2019 patch notes (@100ideas)

this folder is a fork of source code of the overmind-forms node package with small patch.
- original: https://github.com/garth/overmind-forms
- patch: https://github.com/100ideas/overmind-forms
- demo repo: https://github.com/100ideas/overmind-forms-demo/ (see codesandbox_v1 branch)
- [codesandbox demo](https://codesandbox.io/s/github/100ideas/overmind-forms-example/tree/codesandbox_v1)

I couldn't get codesandbox to use my fork of the package in the demo app, so hence this `patch/overmind-forms` folder and `package.json` postinstall script.

Detail of codesandbox error: I wanted codesandbox to install using a particular branch of the github repo of my fork of `overmind-forms`, the `hack` branch.

```json
// in package.json
  "dependencies": {
    "overmind-forms": "100ideas/overmind-forms#70fdc88",  // fail
    "overmind-forms": "100ideas/overmind-forms#hack",     // fail
    "overmind-forms": "100ideas/overmind-forms",          // works - but is master branch
```

I also noticed codesandbox having trouble compiling the typescript source of the package on install (since the demo repo is a js create-react-app template and probably didn't have the right ts deps). So I un-gitignored the built directories (`es/`, `lib/`), added them to my repo, and tried disabling the build directives triggered by the `prepare` script in the packages' `package.json`. that also didn't work with codesandbox.

## 20 Feb 2019 patch notes (@100ideas)

if you want to do local development: 

```bash
# get overmind-forms demo app
git clone https://github.com/100ideas/overmind-forms-example.git)

# get patched overmind-forms (place in parallel directory to demo)
git clone https://github.com/100ideas/overmind-forms.git)
git checkout --track origin/hack

# build and set up local package link
cd overmind-forms
yarn
yarn build
yarn link

# complete link
cd ../overmind-forms-example
yarn link overmind-forms

# install & start
yarn
yarn start
```

`overmind-forms/src/isFormValid.ts` is a derived state function built into `overmind-form` that makes `state.<your-forms-name>.isValid` reactive. 

It is broken in `overmind-forms@0.0.4` + `overmind@15.1.2`. The function should trigger whenever any field element in the `overmind-form` has the property `isValid: false`, i.e. at a path-value like `state.[overmindForm].[formField].isValid: false`.

But location of the isValid boolean is one level deeper, at `state.[overmindForm].[formField].isValid.isValid: false`. This patch fixes the problem by looking one level deeper. A better solution might have been to just move the boolean up one level.

Also, the typescript types are too complicated for me, so currently the compiler complains about a type mismatch using this new path. Someone more familiar with typescript could probably fix that in a couple of minutes.

Lastly, here is what the state tree managed by `overmind-form` looks like (in this case for the form in the demo app):

```json
//state: {
  "loginForm": {
    "isValid": false,
    "email": {
      "value": "",
      "isPristine": true,
      "isValid": {
        "isValid": true  // maybe shouldn't be valid isPristine is true...
      },
      "showError": false
    },
    "password": {
      "password": {
        "value": "aa",
        "isPristine": false,
        "isValid": {
          "isValid": false,
          "failedRule": {
            "name": "minLength",
            "arg": 4
          },
          "errorMessage": null
        },
        "showError": false
      },
      "field3": {
        "value": "[\"foo\", \"bar\", \"baz\", \"mip\"]",
        "isPristine": true,
        "isValid": {
          "isValid": false,
          "failedRule": {
            "name": "isAlphanumeric"
          },
          "errorMessage": null
        },
        "showError": false
      }
    }
//}
```

---

# overmind-forms

A basic forms implementation based on the concepts from cerebral-forms.

## Install

```
yarn add overmind-forms
```

or

```
npm install overmind-forms
```

## Setup

Configure your overmind app for use with overmind-forms, the commented lines to your `app.ts`

```ts
// app.ts
import { Overmind, TApp } from 'overmind'
import { namespaced } from 'overmind/es/config'
import { TConnect, createConnect } from 'overmind-react'

import * as login from './modules/login'
import form from 'overmind-forms/module' // add this line

const config = namespaced({
  login,
  form // and this one too
})

declare module 'overmind' {
  interface IApp extends TApp<typeof config> {}
}

export const app = new Overmind(config)
export type Connect = TConnect<typeof app>
export const connect = createConnect(app)
```

## Example Module

The following example shows creating a form in login module

```ts
// modules/login/state.ts
import form from 'overmind-forms'
import { isEmail } from 'validator'

export let loginForm = form({
  email: { value: '', isValid: isEmail },
  password: { value: '', isValid: value => value.length > 1 }
})
```

## Example Component

```tsx
// components/Login/index.tsx
import React from 'react'
import { connect, Connect } from '../../app'

const Login: React.StatelessComponent<Connect> = ({ app }) => (
  <form
    onSubmit={e => {
      e.preventDefault()
      app.actions.login.formSubmitted()
    }}>
    <input
      className={app.login.email.showError ? 'error' : ''}
      value={app.login.email.value}
      onChange={e =>
        app.form.setField({
          field: app.login.email,
          value: e.target.value
        })
      }
    />
    <input
      className={app.login.password.showError ? 'error' : ''}
      value={app.login.password.value}
      type="password"
      onChange={e =>
        app.actions.form.setField({
          field: app.login.password,
          value: e.target.value
        })
      }
    />
    <button type="submit" disabled={app.login.loginForm.isValid}>
      Submit
    </button>
  </form>
)
```
