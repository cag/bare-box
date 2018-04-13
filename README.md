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
