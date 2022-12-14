title: Enforce coding conventions with Git Hooks and Husky
featured_image: files/images/featured/git.jpg
summary: >-
  Examples of how to format code, run tests, and automate the rest of the coding
  conventions checks.
tags: []
categories: []
date: '2022-12-12T14:25:43.000Z'
---
Coding conventions are rules set in place to help us maintain a level of readability and consistency in the codebase.
These rules may involve programming style, architectural best practices, non-functional requirements, etc. 
Git hooks are scripts that run before or after [git events](https://git-scm.com/docs/githooks#_hooks) such as commit, push, checkout, etc. If any of these scripts reports an error, the git action will fail.
Husky is a popular JavaScript package for working with git hooks.
To add husky to your project, install the dependency and run the [init command](https://typicode.github.io/husky/#/?id=automatic-recommended):
```shell
$ npx husky-init && npm install
```
The syntax for updating or adding hooks is `husky add <file> [command]`.
For example, the following command creates a hook that prints **hello from husky pre-commit hook** in the console before each commit.
```bash
$ npx husky add .husky/pre-commit "echo \"hello from husky pre-commit hook \""
```
We can test it by running:
```bash
$ git commit --allow-empty -m "Testing husky..."
```
With the hello world illustration out of the way, we can move into more practical examples.
## Prettify code with Lint-staged and Rome
A standardized set of formatting rules will help us avoid unnecessary conflicts that might appear in git diffs. 
For this, we can use [lint-staged](https://www.npmjs.com/package/lint-staged), a library for running scripts against a set of staged git files and [Rome](https://rome.tools/) as a formatter. 
Start by installing the dependencies:
```bash
$ npm install --save-dev lint-staged rome
```
Add the lint-staged script and configuration to the package.json file. In my example, I will format only .js, .ts, .jsx and .tsx files.
```json
{
  "name": "Playground",
  "version": "1.0.0",
  "scripts": {
    "prepare": "husky install",
    "prettify-staged": "lint-staged"
  },
  "lint-staged": {
    "*.{js,ts,tsx,jsx}": [
      "rome format --write"
    ]
  },
  "devDependencies": {
    "husky": "^8.0.2",
    "lint-staged": "^13.1.0",
    "rome": "^11.0.0"
  }
}

```
Don't forget to update the pre-commit hook!
```bash
$ npx husky add .husky/pre-commit "npm run prettify-staged"
```
That's it! 🥳 From now on, on every commit, the staged JavaScript & TypeScript files will be formatted by Rome.
## Lint code and run tests before pushing to remote
Start by adding a lint task into package.json:
```json
  "scripts": {
    "lint": "rome check ./"
  }
```
and update the pre-push task by running:
```bash
$ npx husky add .husky/pre-push "npm run lint"
$ npx husky add .husky/pre-push "npm run test"
```
From now on, on every push, the codebase should satisfy the linter requirements, and tests should pass.
## Bypassing hooks
While these hooks are helpful in our everyday life, it's easy to imagine a scenario where avoiding these hooks is beneficial. To skip the execution of a git hook, append `--no-verify` at the end of the command. 
For example:
```bash
$ git push --no-verify
```
command will skip the execution of our pre-push checks.

## Next steps
If your project doesn't have git hooks, now is the perfect time to introduce them and gain extra points in the upcoming performance reviews. 
The alternative is to go crazy, think big and share your new hook with the development community. Look at projects like [lolcommits](https://lolcommits.github.io/), [commit-colors](https://github.com/sparkbox/commit-colors) and [podmena](https://github.com/bmwant/podmena) for inspiration.