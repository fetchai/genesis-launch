
# We have launched!

- ChainID: `fetchhub-1`
- Seed: `9f9774d88bb6ff9f43b395baf0e7a4baba27dec6@connect-fetchhub.m-v2-london-c.fetch-ai.com:36856`
- Block explorer: `https://explore-fetchhub.fetch.ai`
- RPC: `https://rpc-fetchhub.fetch-ai.com`

# Fetch.ai Mainnet

In launching the Fetch.ai, there will be full token interoperability with the FET ERC-20 token. Initially, transfers between the two networks will be mediated by a custodial bridge, and the supply of FET tokens that can be transferred to the mainnet will be restricted to 60 million FET. This will be increased progressively over the first six months of the network's operation until there is free movement of FET between Ethereum and the native chain. This will coincide with the deployment of a non-custodial relayer based on the [Gravity](https://github.com/cosmos/gravity-bridge) bridge. 

The token rewards for the first two months of the network's operation will be more than 4% per month corresponding to an annual rate of around 63%. It is expected that all tokens that are moved to the main-net will be staked, so the reverse token swaps to Ethereum will not be enabled until Wednesday the 21st of April when the first unbonding period will have been completed. 

## Timeline
- **18:00 UTC on Monday the 29th of March, 2021**. 
   - The Ethereum bridge [contract](https://etherscan.io/address/0x947872ad4d95e89e513d7202550a810ac1b626cc) will be opened and will collect FET ERC20-token transfers to the mainnet. The contract UI [here](https://token-bridge.fetch.ai/) will be opened to allow FET token holders specify a destination address on the mainnet.
   - This genesis launch repo will be released publicly. At this point, we will start collecting genesis signatures for the initial validator set for the network launch.
- **12:00 UTC on Wednesday the 31st.** The collection of genesis transactions will be closed and merged to create the genesis block.
- **14:00 UTC on Wednesday the 31st.** Mainnet launch. 

## Token Bridge

The UI will check that the destination address has the correct format, which follows the bech32 standard with a prefix of "fetch". ERC-20 tokens will be held in the bridge [contract](https://etherscan.io/address/0x947872ad4d95e89e513d7202550a810ac1b626cc), which will serve as a hot wallet for transferring funds from the mainnet to Ethereum. A [cold wallet](https://etherscan.io/address/0x5a8de252ea228deCe61638C336fE43ac8166166a) will also be used to secure the ERC-20 token funds with a symmetric hot-cold wallet deployed on the mainnet to manage and secure transfers.  

## Install fetchd and fetchcli

Clone and install fetchd and fetchcli from https://github.com/fetchai/fetchd.
Make sure to checkout the `v0.7.2` release before installing

Once installed, verify you have the correct version with:

```bash
fetchcli version
```

This should output `0.7.2`.

## Create key pair

This will create your validator account, along with the private and public keys. Replace <account_name> with a name of your choice, and run:

```bash
fetchcli keys add <account_name>
```

Make sure to save the mnemonic phrase from the output, and keep it somewhere safe and private!

## Create initial transaction

We'll now retrieve the genesis file from a git repository and generate your initial validator transaction.
You'll have to update the genesis file for the network with your validator information and provide us your initial staking transaction by creating a pull request.

Start by forking this repository . If needed, have a look at the [Github Quickstart - Fork a repo](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) guide.


### Retrieve the current genesis

```bash
git clone <your_genesis_repo_fork_url>
cd genesis-launch/
```

### Create genesis account

```
fetchd --home $(pwd) add-genesis-account <account_name> 1000000000000000000afet
```

The `1000000000000000000afet` amount above defines your initial account balance. It must match the amount of token you have transfered across the bridge.

Running the above command also have several side effects:

- It generates a default `fetchd` configuration in `config/config.toml`. Make sure to review it and adjust its content. You can for example set the `moniker` key that will identify your node on the network, and can't be changed later.
- It generates the keys `node_key.json` and `priv_validator_key.json`. You may want to replace them if you already have some prepared, and **make sure to backup and secure them properly**.

Now your account has been added to the genesis file. To ease inspecting the changes, you will probably need to sort the genesis file, as the json format does not preserve ordering:

```bash
cat <<< $(jq -S . <  ./config/genesis.json) > ./config/genesis.json
git diff ./config/genesis.json
```

This should output something similar to:

```diff
           }
+        },
+        {
+          "type": "cosmos-sdk/Account",
+          "value": {
+            "account_number": "0",
+            "address": "fetch<your_address>",
+            "coins": [
+              {
+                "amount": "1000000000000000000",
+                "denom": "afet"
+              }
+            ],
+            "public_key": null,
+            "sequence": "0"
+          }
         }
       ],
```

Make sure your address and amount are correct, then continue to the next step.

### Create validator

```
fetchd --home $(pwd) gentx  --amount 1000000000000000000afet --name <account_name>
```

This command will create a validator and delegate `1000000000000000000afet` tokens from `<account_name>`. This amount is limited by the amount provisioned on the account in the previous step.

Several other parameters can be adjusted here to configure your validator, such as `--commission-max-change-rate`, `--commission-max-rate`, `--commission-rate`, `--ip`... Use `fetchd gentx -h` for the full list of options.

This command will create the file `./config/gentx/gentx-<your_moniker>.json`, containing a transaction for setting up your validator. **Make sure to review it carefully.**
 
This file is signed, and cannot be modified manually. If you want to change any parameters, delete it and issue the `fetchd gentx` command again, specifying the new parameter.

### Commit and push

Once you're happy with your gentx and genesis account settings, you can commit and push the `genesis.json` and your `gentx` file:

```
git add .
git commit -m "<account_name> gentx"
git push origin main
```

Once pushed, you're ready to create your pull request. If needed, here's the [Github documentation - creating a pull request from a fork](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork).

## Start your validator

We'll soon review and merge your pull request. Once we'll have collected all initial transactions from validators, we'll generate the final genesis and publish it on this repository. We'll notify you, so you can then update your fork by pulling the latest genesis file and start your validator providing it the seed identifier:

```bash
git remote add upstream https://github.com/fetchai/genesis-launch.git
git fetch upstream
git reset --hard upstream/main
fetchd --home $(pwd) start --p2p.seeds "9f9774d88bb6ff9f43b395baf0e7a4baba27dec6@connect-fetchhub.m-v2-london-c.fetch-ai.com:36856"
```

You can also set the `<seed_id>` by updating the `seeds` key in your `./config/config.toml` to persist this change.

In the event your fetchd service isn't starting properly, or if you started it with the wrong seed, or any other problems, you can wipe its runtime database with. This won't affect any configuration made under the `config/` folder.

```bash
fetchd --home $(pwd) unsafe-reset-all
```

## Problems, questions?

Feel free to [open an issue](https://github.com/fetchai/genesis-prelaunch/issues) if you encounter any problems or have a question, we'll do our best to help you!
