# Truffle Package Deployment Guide

Let's say you have written a package with Solidity using this Truffle box. This guide covers handling a scenario in which this Truffle project has to play nice with:

* Git source control
* Being published on NPM in an easily consumable format
* Private configurations, in particular keeping information about the deploying wallet private

We will cover a scenario in which the deploying account is part of a hierarchical deterministic wallet derived from a 12-word [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic.

## Specify Target Networks

You may choose any networks you'd like for the deployment. For illustrative purposes, let's suppose that you are going with the [Rinkeby](https://www.rinkeby.io/#stats) and [Kovan](https://kovan-testnet.github.io/website/) testnets. You will need to modify `truffle.js` to declare your intention to deploy on these networks:

```js
// Modify the base configuration as follows:

const config = {
  networks: {
    rinkeby: {
      network_id: 4
    },
    kovan: {
      network_id: 42
    }
  }
};

// (rest of config...)
```

Be sure to declare the correct network IDs for the respective networks you decide to name in your Truffle configuration. [Here is a list of these network IDs.](https://ethereum.stackexchange.com/a/17101)

We do these declarations so that we can take advantage of the behavior of `truffle networks --clean`. See [these docs](http://truffleframework.com/docs/advanced/commands#networks) for more details.

## Make an HD Wallet for the Deployment

[`truffle-hdwallet-provider`](https://github.com/trufflesuite/truffle-hdwallet-provider) can be used to furnish a provider which is backed by a BIP39 HD wallet. Let's install this locally in this repository as a development dependency:

    npm i -D truffle-hdwallet-provider

This allows one to use a 12-word BIP39 mnemonic to derive a keypair and its associated Ethereum address for use on the network.

Need one on the fly? One trick to get one is to run [Ganache CLI](https://github.com/trufflesuite/ganache-cli) for a moment and quit it with Ctrl+C:

```
$ ganache-cli
Ganache CLI v6.0.3 (ganache-core: 2.0.2)

Available Accounts
==================
(0) 0xae2e104a271b96bee7fe31a7b687ddf48d120356
(1) 0x0e9d59b93a926c69e27c1077922c5e7209ad394b
(2) 0xf7f92f008fc87904187848f974bf0098c12c47fe
(3) 0x3214ad4c80c811b70bc1b353629db600af9fb2d4
(4) 0xb88c372207ee6c9327172af708ab39a9245a8039
(5) 0xbb647223aeca95fe1136fc9f48d49f71362fe448
(6) 0xe4052566e33bdbc4ec064849c7f0e0261256af9f
(7) 0x8c21c0d69a6abfcea2622808fc531cdba35055cc
(8) 0xd37b370fd6d587e8e55291b3ad10356e0e65a68e
(9) 0x16bb5d6be20591cd195aa8f2a095ec0487097dcc

Private Keys
==================
(0) 9aa38d11d23c7b940a7b2e0c58062a523790544b7cfb9263dff29afa019b92cd
(1) 6e88319718d77b2f68bfd5f1c188e46376b570b183c38e6768404b92d8b996eb
(2) 93b082f3270e81f135b8e61a8a38d0a6169e3a03131ebe01f4f81ef65b3a2aa7
(3) deae1e8979887b63695e859314512b6ce5860fe6711ea336f0fe5d78daa2e6be
(4) df86e8b58835d5cf89c63b0a9a53b2daa2a7935b40aacaddf460741e62214948
(5) 1ee2d31adad0e339897fe8d7cd4a40114897d23695e9ad9bd524f7838bc79318
(6) eb354ca869dcbc18bfffe07a2ed2ed0729037f1fd6f70e13e5d304b499fe9215
(7) 462d24f7e3bba9bd0359de079d520acd60e292b3d79003294b90dc7322b82157
(8) f9bac61df108a401dc43962c2b4bc57cfab78967e75715f1730662c425c9f08d
(9) 1e46ba6f2a2f59a588546de65372c9de89771934ec20927b26d47027c6089e4f

HD Wallet
==================
Mnemonic:      romance spirit scissors guard buddy rough cabin paddle cricket cactus clock buddy
Base HD Path:  m/44'/60'/0'/0/{account_index}

Listening on localhost:8545
```

Take note of the list of addresses and the mnemonic, which in this instance is `romance spirit scissors guard buddy rough cabin paddle cricket cactus clock buddy`. Don't use the same mnemonic! This is supposed to be a secret.

Go ahead and put the following in `truffle-local.js`:

```js
const HDWalletProvider = require("truffle-hdwallet-provider");

const mnemonic =
  "romance spirit scissors guard buddy rough cabin paddle cricket cactus clock buddy";
const infuraAccessKey = "";
const accountIndex = 7;

module.exports = {
  networks: {
    rinkeby: {
      provider: new HDWalletProvider(
        mnemonic,
        `https://rinkeby.infura.io/${infuraAccessKey}`,
        accountIndex
      )
    },
    kovan: {
      provider: new HDWalletProvider(
        mnemonic,
        `https://kovan.infura.io/${infuraAccessKey}`,
        accountIndex
      )
    }
  }
};
```

The account used for the deployment will be associated with the `mnemonic` and the `accountIndex` specified above (note that this index is 7 in this example). Therefore, the address which does the deployment in this example will be `0x8c21c0d69a6abfcea2622808fc531cdba35055cc`.

Also note that we will, for the purposes of this guide, be relying on the infrastructure provided by [INFURA](https://infura.io). You may sign up and get an INFURA access key for free if you'd like.

## Sending Deployment Transactions via Truffle

Run `npm run truffle compile` to check that there aren't any basic scripting errors that have been introduced while writing `truffle-local.js`.

Then let's try to deploy our contracts on Rinkeby. Run `npm run truffle migrate -- --network rinkeby` to deploy the contracts using the scripts in the `migrations` folder.

Assuming you haven't funded your account with test ether yet, you will probably get an error like the following:

```text
> truffle migrate "--network" "rinkeby"

Using network 'rinkeby'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
Error encountered, bailing. Network state unknown. Review successful transactions manually.
insufficient funds for gas * price + value
```

If that is the case, go ahead and [get some Rinkeby test ether](https://www.rinkeby.io/#faucet) for your account. Then try running `npm run truffle migrate -- --network rinkeby` again. This will probably take a couple of minutes.

If you see something like the following in your terminal (except with your own contracts):

```text
> truffle "migrate" "--network" "rinkeby"

Keccak bindings are not compiled. Pure JS implementation will be used.
Using network 'rinkeby'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0x0cf2524f634278b11d0ce41f5c2e157492029c642d721af8343e76586eb4a06e
  Migrations: 0xf0681a06da32b8276b0a7b685019056c2bcfbf13
Saving successful migration to network...
  ... 0x60bae6840a643b1078320a2b8305694ee8544d293892984c751fb212176dbe87
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying BigToken...
  ... 0x3f78c1aae7e5940590d903f6f32f588bb904f6973bf03d18c7e85ca2c1299693
  BigToken: 0x0152b7ed5a169e0292525fb2bf67ef1274010c74
  Deploying AddressRegistry...
  ... 0xaa10a3d8ba2a08ae277eaadd5b876753ac118ede542ae89c25c882eda3766c53
  AddressRegistry: 0xd3515609e3231d6c5b049a28d0d09d038b4cfaed
Saving successful migration to network...
  ... 0x7b8d3fe644e224607551488e3323acfc856b5b9e8f2bb929e9508e534faf4fd5
Saving artifacts...
```

then everything went well, and your contracts are now deployed on Rinkeby!

Go get some [Kovan ether](https://github.com/kovan-testnet/faucet) for the same address, and deploy it on Kovan with `npm run truffle migrate -- --network kovan`

## Inspecting and Preparing Deployed Addresses

After running the migrations, Truffle places the deployed addresses inside of the build artifacts located in `build/contracts`. These build artifacts are JSON files which, when converted to a plain JS object and passed as an argument to [`truffle-contract`](https://github.com/trufflesuite/truffle-contract), yields a Truffle contract abstraction which may be used in decentralized application clients.

The deployment execution in the last section resulted in a unique set of addresses which must be distributed to clients. You may run `npm run truffle networks` to see a list of the most recent deployment locations:

```text
$ npm run truffle networks

> big-token@2.0.0 truffle /home/alan/src/github.com/cag/big-token
> truffle "networks"


Network: UNKNOWN (id: 4447)
  AddressRegistry: 0xf25186b5081ff5ce73482ad761db0eb0d25abfbf
  BigToken: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
  Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0

Network: kovan (id: 42)
  AddressRegistry: 0xd3515609e3231d6c5b049a28d0d09d038b4cfaed
  BigToken: 0x0152b7ed5a169e0292525fb2bf67ef1274010c74
  Migrations: 0xf0681a06da32b8276b0a7b685019056c2bcfbf13

Network: rinkeby (id: 4)
  AddressRegistry: 0xd3515609e3231d6c5b049a28d0d09d038b4cfaed
  BigToken: 0x0152b7ed5a169e0292525fb2bf67ef1274010c74
  Migrations: 0xf0681a06da32b8276b0a7b685019056c2bcfbf13
```

You may also verify that the `networks` key in the build artifacts contains (and in fact is the source of) this information. You may also see deployment information for some `UNKNOWN` networks. Those are typically associated with previous runs of the migration scripts on development networks, such as ones created by `ganache-cli` or `truffle develop`. We can use `npm run truffle networks -- --clean` to remove the information associated with those networks from the build artifacts.

## Saving Deployed Addresses

Information about deployment addresses *should* be preserved for source control. In order to achieve this, we can use a script included with `lil-box` to extract this information from the build artifacts into `networks.json`:

```sh
npm run extractnetinfo
```

Running this command will create or overwrite `networks.json` with all of the network information contained in the build artifacts. Once we have this, build artifacts containing the deployment information for the Rinkeby network can be restored from plain build artifacts:

```sh
npm run injectnetinfo
```

Furthermore, the following command will remove `UNKNOWN` network information before injecting the network information into the build artifacts:

```sh
npm run resetnetinfo
```

Note that `resetnetinfo` is invoked as part of `prepack` to ensure that build artifacts published on NPM contain just the canonical deployment information required for dapp clients.
