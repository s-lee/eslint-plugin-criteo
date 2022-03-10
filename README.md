# eslint-plugin-criteo

This ES lint plugin defines custom rules we use at Criteo and exposes a recommended set of rules.

## Installation

```sh
npm install eslint eslint-plugin-criteo --save-dev
```

## Usage

Add `criteo` to the plugins section of your `.eslintrc` configuration file and apply:

- `plugin:criteo/recommended-app` if the project is an application
- `plugin:criteo/recommended-lib` if the project is a library

```json
{
  "plugins": ["criteo"],
  "overrides": [
    {
      "files": ["*.ts"],
      "extends": ["plugin:criteo/recommended-app"], // or recommended-lib
      [...]
    },
    {
      "files": ["*.html"],
      "extends": ["plugin:criteo/recommended-template"],
      [...]
    }
  ]
}
```

Then configure/disable the rules under the rules section.

```json
{
  "rules": {
    "criteo/rule-1": ["error", { "custom-config-key": "custom-config-value" }],
    "criteo/rule-2": "off"
  }
}
```

## Pre-commit Git hook

As a developer, it can be very frustrating to get a change rejected by QA bot because the code does not respect an ES lint rule. Even more if the failing rule has an auto-fix! To avoid it, you can configure your project to run ES lint automatically on pre-commit. Because it will target only staged files, it is quite fast!

1. Define the pre-commit hook script `.git-hook-lint-staged` at the root of the first/main UI project of the repository. It will make it available to all developers since Git hooks cannot be pushed directly.

```sh
#!/bin/sh
cd project1 && npx lint-staged \
  && cd ../project2 && npx lint-staged # If the repository hosts several projects, add each of them
```

2. Also define the `postinstall` NPM script in the `scripts` section of `package.json`:

```
"scripts": {
  "postinstall": "npx shx cp .git-hook-lint-staged ../.git/hooks/lint-staged"
}
```

It will enable the hook automatically after `npm install`.

3. Still in the same project, install `shx` to make the copy command work on all environments: `npm install --save-dev shx`

4. Install `lint-staged` in each UI project of the repository: `npm install --save-dev lint-staged`

5. In each project, also define its configuration by declaring in `package.json`:

```
"lint-staged": {
  "**.{ts,component.html}": "eslint --fix"
}
```

You can also declare `"*.{ts,component.html,json,scss}": "prettier --write"` if you use Prettier.

🙌

## Criteo rules

### ngx-component-display

_Ensure Angular components have a display property set._

Config:

- `ignore`: classes whose names match this regular expression (defined as string) will be ignored (default: `'^.*DialogComponent$'`)
- `propertyName`: name of the display property (default: `'cdsDisplay'`)

Why?

- By default, custom components are displayed as `inline` by the browser which is rarely what we expect. For example, it makes impossible to define a width or margins on them. E.g. `<my-component class="w-100 cds-mb-3"></my-component>` would have no effect.
- The workaround of wrapping them in `<div></div>` should be avoided to not make the DOM and the bundle file heavier.

### ngxs-selector-array-length

_Ensure that when using the @Selector() decorator, the number of selectors passed in matches the number of arguments passed to the selector function._

Why?

It can be tricky to pin down the source of an error when using the @Selector() decorator. While this rule can't make sure you put all the parameters in the right order, it does avoid the most obvious mistakes.

## External rules

- All recommended rules from:
  - `@angular-eslint/template`: https://github.com/angular-eslint/angular-eslint/tree/master/packages/eslint-plugin-template/docs/rules
  - `@typescript-eslint`: https://github.com/typescript-eslint/typescript-eslint/tree/main/packages/eslint-plugin#supported-rules
  - `eslint`: https://eslint.org/docs/rules/
- `@angular-eslint/template/use-track-by-function`: https://github.com/angular-eslint/angular-eslint/blob/master/packages/eslint-plugin-template/docs/rules/use-track-by-function.md
- `no-only-tests/no-only-tests`: https://github.com/levibuzolic/eslint-plugin-no-only-tests#usage
