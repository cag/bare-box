# lil-box

[![Build Status](https://travis-ci.org/gnosis/lil-box.svg?branch=master)](https://travis-ci.org/gnosis/lil-box)

A little [Truffle](http://truffleframework.com/) box.

* No frontend. Choose your own frontend adventure, or forgo it altogether!
* Designed to work with NPM and Git out of the box.
* Comes with its own Truffle.
* Gives you a `truffle-local.js` to hide things in.
* Helps you handle deployments.

Sounds good? Get it with:

    truffle unbox gnosis/lil-box

## Extras

### Deployment Tools

This box comes with a couple of scripts for recording deployment information on your networks. Check this [deployment guide](deployment-guide.md) for more details.

### Linting and Formatting

This box also comes with source code linting provided by [ESLint](https://eslint.org/):

    npm run lint

and formatting by [Prettier](https://prettier.io/):

    npm run fmtsrc

### Ignored Files/Directories for Different Systems

Both `.gitignore` and `.npmignore` are specified in this box.

`truffle-local.js` is ignored by both Git and NPM, and can contain configuration local to the machine.

The `build` folder is ignored by Git since build artifacts can be almost entirely reproduced by the local Truffle instance and the contract source code, with the exception of deployment information.

However, the `build` folder is *not* ignored by NPM. This means `lil-box` developers can publish packages on NPM which also contain deployment information. Subsequent users of those packages then have direct access to those deployments.

Also, a `.prettierignore` instructs Prettier to ignore the outputs of build processes.

### NPM `prepack` Hook

For preparing NPM packages, a `prepack` hook is included which will lint and format the source, as well as compile Truffle build artifacts and reset the deployment information on these artifacts using the [deployment tools](deployment-guide.md).
