# Contributing Guide

## Prepare the Development Environment

1. Make sure you've got Node (v. ^16) and NPM (v. ^8) installed
2. Fork the repo
3. Clone the fork
4. Install the dependencies: `npm i`
5. Run `npm run env:prep`
6. Introduce some changes (see the section below)
7. Make sure the `npm run build` passes
8. Commit and push the changes
9. Create a PR

_In case of any troubles, refer to the troubleshooting instructions at the bottom of this page._

The step 5 is a shorthand for `npm run build:lib && npm link && npm link @maplibre/maplibre-gl-directions` where the last two commands must be performed in order to have the `@maplibre/maplibre-gl-directions` as a local symlinked dependency, because the Demo project uses not the library sources, but the locally-built `/dist` folder to make sure that the instance being tested is the same instance which is deployed to the end user.

## Project Structure

The source files of the library itself are located under the `/src` folder. The `/src/main.ts` is the main entry point. The `/src/directions` folder contains everything related to the plugin's main purpose. The `/src/control` folder contains all the UI-Control-related source code (is currently WIP).

TypeDoc is responsible for building all the API documentation. The API documentation is built from the library sources and from the files located under the `/doc` folder.

The Demo project's source files are located under the `/demo` folder. See its README for more information.

## Making Changes

_All the processes mentioned in this section are described in more detail in the section below._

Changes to the library sources should be introduced while there's a running `dev:lib` process. You may also want to have a running `dev:doc` process to see how correctly your doc-comments are parsed. Notice however that `dev:doc` does not serve the built documentation static-website somewhere, it just continuously rebuilds it.

The Demo project might be used as a test-stand while working on the library sources. The `dev:demo` process serves the demo project at `localhost:3000` (by default). The Demo project uses the built `/dist` folder to refer to the library (instead of using sources from the `/src` directly), so you may also want to have a running `dev:lib` at the same time.

You may keep any examples you add when creating a PR if you think the changes you were working on are worth a separate example. An additional example by itself (without modifying the library sources) is a good PR candidate as well. The only thing you should remember when introducing any changes to the Demo project is that they are about to be seen by any other person, a potential end-user, so don't go too crazy and provide human-readable explanations of what the example is really meant to show.

## NPM Scripts

- `npm run prepare` - is run automatically when the project is being prepared (namely, when `npm i` is run). Configures the Husky's checks.

- `npm run env:prep` - builds the library and self-links it for the Demo project.

- `npm run dev:lib` - starts a vite-powered development server for the library source files. Continuously rebuilds the contents of the `/dist` folder while you make changes to the `/src` folder. **Does not rebuild types!** Must be restarted in order to rebuild them.

- `npm run dev:doc` - continuously rebuilds the documentation static-website while you make changes to the library source files and puts the results under the `/docs/api`.

- `npm run dev:demo` - starts a vite-powered development server for the Demo project. The Demo project targets the library from the `/dist` folder via a symlinked `@maplibre/maplibre-gl-directions` package.

- `npm run build` - Combines `npm run lint`, `npm run build:lib`, `npm run build:doc` and `npm run build:demo` into a single call.

  1. `npm run build:lib` - builds the library (the `/src` folder contents) and outputs the resulting es-module and its type declarations into the `/dist` folder.

  2. `npm run build:doc` - builds the documentation (the `/doc` folder contents and the source code comments) using the TypeDoc compiler and outputs the resulting static-website into the `/docs/api` folder.

  3. `npm run build:demo` - builds the Demo project (the `/demo` folder contents) and outputs the resulting static-website into the `/docs` folder.

- `npm run format` - formats all the files that are not ignored by the `.prettierignore` and rewrites the files in-place.

- `npm run prelint` - is run automatically each time when `npm run lint` is called.

- `npm run lint` - lints and fixes the contents all the .ts, .js, .cjs and .svelte files and rewrites the files in-place.

- `npm run check:lib` - checks the `/src` folder contents using the `svelte-check`.

- `npm run check:demo` - checks the `/demo` folder contents using the `svelte-check`.

- `npm run check` - combines `npm run lint`, `npm run check:lib` and `npm run check:demo` into a single call. Is run automatically if there were changes to the `/src` or `/demo` folders before you commit the changes and aborts the commit if there are errors that could not be automatically fixed by eslint.

Since the deployment process is configured to be performed automatically, you don't have to make sure that the `/docs` folder is up-to-date (it's actually ignored by Git). You run `npm run build` manually only to make sure that the build doesn't fail.

## CI/CD Setup

We use GitHub Actions for deployment. There are several tasks that get performed independently:

- Build and Deploy the Demo project
- Build and Publish the Library
- Pull Request Checks

### Build and Deploy the Demo project

Deploys the `/docs` folder to GitHub Pages each time the main branch is pushed (building the folder from sources beforehand). The GitHub Pages is configured in such a way so that the deployed site is actually available under the maplibre.org domain. Does not require any special access tokens.

### Build and Publish the Library

Triggered manually whenever the team decides it's the time to release a new version of the library. Builds the sources from the target branch, runs all the checks, prepares the build-artifacts, updates the version in `package.json` (and automatically commits the changes), prepares the release notes and releases the new version to NPM all in one action.

At the time of writing this ignores all the changes to the files made by `eslint --fix` and `prettier --write`, but there's a plan to fix that (#108).

Requires a special access token (a secret) with slightly bumped permissions in order for github actions bot to be able to commit and push the updated `package.json` version of the library. The token is called "GH_TOKEN" and is a personal access token with the following permissions granted: admin:org, admin:org_hook, admin:public_key, admin:repo_hook, repo, workflow, write:packages.

Also requires the "NPM_ORG_TOKEN" token which is an organization secret to be able to release the package under the "@maplibre" namespace.

### Pull Request Checks

Whenever there's a new PR, ensures that:

- The build passes
- The PR has at least one of the following labels: either `release:ignore`, or `semver:patch`, or `semver:minor`, or `semver:major`.

`semver-*` labels are used to indicate the backwards-compatibility of the proposed changes in order for GitHub to be able to automatically build the release notes. The `release:ignore` label should be used in cases when there's no need to mention the PR in release notes.

Please, note that there's no protection from releasing a minor or a patch version when there was a `semver:major` PR merged. The maintainers should track the changes themselves. In general, it's a good idea to release new versions as soon as there are some changes available.

## Troubleshooting

### "Failed to resolve import "@maplibre/maplibre-gl-directions" from "demo/<...>". Does the file exist?

That happens when the package is not self-symlinked. Perhaps you did `npm i` or installed some new dependencies (NPM removes all the symlinked deps after updating the `node_modules).

**Solution**: run `npm run env:prep` once again.

### `npm run build` fails without a meaningful reason

This may happen if you have already had the project on your machine and recently pulled the upstream changes. Due to some package version updates the build may start to fail without any reasonable explanation of why is it so.

**Solution**: run `npm ci` instead of `npm i` and try again. Another solution might be to completely delete the `node_modules` folder and reinstall the dependencies from scratch.
