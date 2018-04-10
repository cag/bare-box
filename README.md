# lil-box

[![Build Status](https://travis-ci.org/gnosis/lil-box.svg?branch=master)](https://travis-ci.org/gnosis/lil-box)

A little Truffle box.

* No frontend. Choose your own frontend adventure, or forgo it altogether!
* Designed to work with NPM and Git out of the box.
* Comes with its own Truffle.
* Gives you a `truffle-local.js` to hide things in.
* Has robot maintainers.

Sounds good? Get it with:

    truffle unbox gnosis/lil-box

## Extras

This box comes with a couple of scripts for recording deployment information on your networks.

`npm run extractnetinfo` will take all the network info in your Truffle build artifacts and put them in `networks.json`. This file can be checked into source control in lieu of the `build` artifacts folder.

`npm run injectnetinfo` will correspondingly take the network info contained in `networks.json` and put them back in their respective `build` artifacts. Furthermore, `npm run resetnetinfo` will clean the network info before doing this using [`truffle networks --clean`](http://truffleframework.com/docs/advanced/commands#networks).
