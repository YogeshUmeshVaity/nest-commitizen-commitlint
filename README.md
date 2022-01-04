<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo_text.svg" width="320" alt="Nest Logo" /></a>
</p>
<p align="center">
<a href="https://github.com/commitizen/cz-cli" target="_blank"><img src="https://img.shields.io/badge/commitizen-friendly-brightgreen.svg" alt="commitizen friendly" /></a>
<a href="https://github.com/conventional-changelog/commitlint" target="_blank"><img src="https://img.shields.io/badge/commit-lint-brightgreen" alt="commitlint" /></a>
<a href="https://conventionalcommits.org/" target="_blank"><img src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg?style=flat-square" alt="conventional-commits" /></a>
<a href="https://typicode.github.io/husky/#/" target="_blank"><img src="https://img.shields.io/badge/husky-hooks-brightgreen" alt="husky" /></a>
<a href="https://github.com/okonet/lint-staged" target="_blank"><img src="https://img.shields.io/badge/lint-staged-brightgreen" alt="lint-staged" /></a>
</p>

<p align="center">A sample NestJS repository to demonstrate how to setup Commitizen, commitlint, Husky and Lint-staged for useful commit messages.</p>

## Create the NestJS Project

If you have an already existing NestJS project, skip this step.

Use the Nest CLI to create the project. This makes sure that the Prettier comes pre-configured with ESLint:

```
nest new your-project-name
```

## Install Husky

It’s better to install Husky independent of lint-staged like this to make sure we get the latest version:

```
npx husky-init && npm install
```

Disable running tests in pre-commit hook by commenting the line in the file .husky/pre-commit. We’ll run tests in pre-push hook instead:

```
# npm test
```

## Install Lint-staged

The following command installs the Lint-staged:

```
npx mrm@2 lint-staged
```

Update the lint-staged option in package.json of your project:

```
"lint-staged": {
   "{src,apps,libs,test}/**/*.ts": [
     "eslint --fix"
   ]
 }
```

The snippet above also covers the prettier formatting, if the prettier is configured with ESLint. NestJS comes with a built-in configuration of Prettier with ESLint. So we don’t need to add the ``npm run format`` or ``prettier`` command to this. We don’t need to add the ``"git add"`` either, because from the Lint-staged version 10 onwards, it is added automatically.

### Optional: Fail on warnings

If you want the commit to fail, if there are warnings like unused variables or unused imports, add ``--max-warnings=0`` flag in the lint script of package.json. It would be better to just create a separate script for it. This can be used on the pre-push hook, if we want to allow the warnings during the feature development:

```
"lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix --max-warnings=0"
```

Not all issues are automatically fixed by ESLint. The List of auto fixable issues can be seen [here](https://eslint.org/docs/rules/).

## Install Commitizen globally

Installing commitizen globally is optional, but recommended if you want to use the shorter command `git cz` for committing.

Install the package globally. You may want to add `sudo` before the command if this doesn't work:

```
npm install -g commitizen
```

Install your preferred commitizen adapter globally (for example[ cz-conventional-changelog](https://www.npmjs.com/package/cz-conventional-changelog)):

```
npm install -g cz-conventional-changelog
```

The following command creates a `.czrc` file in your home directory, with path referring to the preferred, globally-installed, commitizen adapter:

```
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

Now you can `cd` into any git repository and use `git cz` instead of `git commit`. This will run the commitizen flow to make a commit. The `git commit` will still run normally whenever you want to use it.

You can use all the `git commit` options with `git cz`. For example: `git cz -a`.

## Install Commitizen in your project

Installing and running Commitizen in your project allows you to make sure that developers are running the exact same version of Commitizen on every machine.

Install Commitizen in the project:

```
npm install --save-dev commitizen
```

Initialize the conventional changelog adapter:

```
npx commitizen init cz-conventional-changelog --save-dev --save-exact --force
```

Add the ``commit`` script to the ``scripts`` section of the ``package.json``:

```
"commit": "cz"
```

Now you can use the command ``npm run commit`` to make a commit using the Commitizen flow.

## Install commitlint

With Commitizen, If the contributors write the git message in the traditional style, for example:

``# git commit -m “Add commitlint.”``

Now the Commitizen flow will start. But if the user aborts the commitizen flow by pressing Ctrl + C, the commit will be done successfully with the above message. That’s against the conventional commits. So, we need the commitlint for aborting the process, if the commit doesn’t meet the standards.

Install commitlint in the project:

```
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

The following command creates the config file .commitlintrc and adds the conventional config:

```
echo "{\"extends\": [\"@commitlint/config-conventional\"]}" > .commitlintrc
```

The following command creates a Husky hook for commitlint and copies the script to it:

```
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

## Run unit tests and lint warnings as errors on ``git push``

This is for catching unused imports and variables in your code. Add the following new script to the `scripts` option of package.json:

```
"lint:warnings": "eslint \"{src,apps,libs,test}/**/*.ts\" --max-warnings=0"
```

Notice that we don’t have the ``--fix`` flag in this script. We want to fix manually and then commit and then push.

To create ``pre-push`` Husky hook and add the script to it, run the following command:

```
npx husky add .husky/pre-push 'npm run lint:warnings'
```

To add the test script to the already created ``pre-push`` husky hook, run the following command:

```
npx husky add .husky/pre-push 'npm test'
```

That's it!

### Optional: Enforce Commitizen flow to run on ``git commit``

Currently, we don't enforce the Commitizen flow on `git commit` command because of an [issue](https://github.com/commitizen/cz-cli/issues/844#issue-971205114) of commitizen flow running twice when you run the `git cz` command.

Not forcing it allows the contributors that can follow the conventional commits without the commitizen flow to use `git commit`. The commitlint takes care if they don't follow the conventional standard, anyway.

When the issue is fixed, we can create the Husky hook for it and add the script to start Commitizen flow:

``npx husky add .husky/prepare-commit-msg 'exec < /dev/tty && git cz --hook || true'``

## License

[MIT licensed](LICENSE).
