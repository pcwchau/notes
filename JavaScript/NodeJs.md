# Index

- [Index](#index)
- [Overview](#overview)
  - [Operation](#operation)
- [NPM](#npm)
  - [Operation](#operation-1)
    - [Install all dependencies](#install-all-dependencies)
    - [Install a single package](#install-a-single-package)
    - [package.json](#packagejson)
      - [Semantic Versioning](#semantic-versioning)
      - [Dependencies vs. devDependencies](#dependencies-vs-devdependencies)
  - [npx](#npx)
- [Yarn](#yarn)

# Overview

- [Introduction to Node.js - Official](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)

Node.js is a runtime environment that allows you to run JavaScript on the backend. It came on the scene because JavaScript used to only work in the web browser. Node.js runs the V8 JavaScript engine, the core of Google Chrome, outside of the browser.

- Node.js provides a set of asynchronous I/O primitives in its standard library that prevent JavaScript code from blocking.
- You can decide which ECMAScript version to use by changing the Node.js version.
- You can also enable specific experimental features by running Node.js with flags.
- Node.js supports both the CommonJS and ES module systems (since Node.js v12).
- Since V22.6.0, Node.js has experimental support for some TypeScript syntax. You can write code that's valid TypeScript directly in Node.js without the need to transpile it first.

## Operation

To check the version of Node.js:

```sh
node -v
```

To run a JavaScript file:

```sh
node hello-world.js
```

# NPM

- [About npm - Official](https://docs.npmjs.com/about-npm)
- [Tutorial - careerfoundry](https://careerfoundry.com/en/blog/web-development/what-is-npm/)

NPM stands for Node Package Manager. It started as a way to download and manage dependencies of Node.js packages, but it has since become a tool used also in frontend JavaScript. Now NPM is the world's largest Software Library Registry. NPM is also a software package manager and installer only for JavaScript (and TypeScript). It can handle package dependency for your project.

**It is pre-installed when you download Node.js on your system.** 

You define all your project’s dependencies inside your `package.json` file. Anytime you or a team member needs to get started with your project, all they have to do is run `npm install`.

## Operation

For the `npm` commands, it is suggested to execute in CMD instead of PowerShell.

To check the version of NPM:

```sh
npm -v
```

To update NPM on your system, run the following command:

```sh
npm install npm@latest -g
```

To create a `package.json` file in the beginning of the project:

```sh
npm init -y
```

### Install all dependencies

If a project has a `package.json` file, by running

```sh
npm install
```

it will install everything the project needs locally, in the `./node_modules` folder, creating it if it's not existing already.

Other options:

- `--production` installs only packages from *dependencies*, but not *devDependencies*.
- `--no-optional` will prevent optional dependencies from being installed.
- `--legacy-peer-deps` can solve conflicts.

### Install a single package

```sh
npm install <package-name>
```

Furthermore, since npm 5, this command adds `<package-name>` to the `package.json` file *dependencies*. Before version 5, you needed to add the flag `--save`.

Often you'll see more flags added to this command:

- `-g` installs the package files in `/usr/local` or wherever node is installed. Use this if you are going to run it on the command line. This is common for command-line tooling packages like live-server.
- `--save-dev` installs and adds the entry to the `package.json` file *devDependencies*.
- `--no-save` installs but does not add the entry to the `package.json` file *dependencies*.
- `--save-optional` installs and adds the entry to the `package.json` file *optionalDependencies*.

### package.json

- [package.json Docs - npm](https://docs.npmjs.com/cli/v9/configuring-npm/package-json)

A `package.json` file is created by npm and exists at the root of a project in JavaScript/Node.

An `npm install` within the context of an npm project will download packages into the project's `node_modules` folder according to `package.json` specifications, upgrading the package version (and in turn regenerating package-lock.json) wherever it can based on `^` and `~` version matching.

In the `package.json` file there is also a scripts property. This can be used to run command line tools that are installed within the project’s local context. Common scripts are things like `npm start`, `npm test`, etc.

#### Semantic Versioning

```json
"dependencies": {
  "cors": "^2.8.5",
  "express": "4.16.4",
},
"devDependencies": {
    "eslint": "~4.19.1",
    "prettier": "^1.19.1",
}
```

The versions above are in the format MAJOR, MINOR and PATCH:
- A MAJOR version involves breaking changes—likely you will need to update the package in your project when a major version has changed.
- A MINOR version change is backwards compatible, meaning it should update without breaking things (well, one can hope)
- A PATCH version change is backwards compatible bug fixes, or other small fixes

You’ll see there’s also some characters before the version in the `package.json` above:
- `^`: latest minor release. For example, a `^1.0.4` specification might install version `1.3.0` if that's the latest minor version in the `1.X.X` major series.
- `~`: latest patch release. In the same way as `^` for minor releases, `~1.0.4` specification might install version `1.0.7` if that's the latest minor version in the `1.0.X` minor series.
- No symbol before the version means the version of the package must match exactly, and should not be updated

#### Dependencies vs. devDependencies

- Dependencies are the list of modules/packages that are required for your project to run. These are installed using npm install to add the package to the dependencies list.
- devDependencies, short for development dependencies, are modules/packages that are NOT required for your project to run. These are often things that help the development process but aren’t part of the project themselves. For example, linters like eslint, testing, etc.

## npx

NPX stands for Node Package eXecute. It is simply an NPM package runner. **It allows developers to execute any Javascript Package available on the NPM registry without even installing it.** When using `npx <package>`, the current stable version of the package will be downloaded and executed at the time the command is run. After the execution is finished, the package will be uninstalled automatically.

**NPX is installed automatically with NPM version 5.2.0 and above.**

Some examples:

```sh
npx react-native@latest init AwesomeProject

npx create-next-app@latest some-project
```

# Yarn

- [Yarn vs. npm - knowledgehut](https://www.knowledgehut.com/blog/web-development/yarn-vs-npm)
- Yarn, or Yet Another Resource Navigator, is a relatively new package manager developed by Facebook.
- It was developed to provide more advanced capabilities that NPM lacked at the time (such as version locking) while also making it safer, more reliable, and more efficient.
- NPM has introduced several important features ever since Yarn was released. Yarn is now more of an alternative to NPM than a replacement in its current version.

Since Yarn doesn’t come pre-installed with Node.js, it needs to be installed explicitly as:

```sh
npm install yarn -g 
```

Install:

```sh
yarn install  # all pacakges
yarn add <package> # specific package

yarn remove <package>
```