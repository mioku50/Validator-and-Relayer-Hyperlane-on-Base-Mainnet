---
cover: .gitbook/assets/unnamed (14).jpg
coverY: 0
---

# Guide

**I also have a guide (in Russian) with screenshots in my Medium** - https://bit.ly/3XvBdru



_**Validators**_

Hyperlane validators are the offchain agents responsible for security: they monitor messages in a chain mailbox and, if necessary, sign the merkle root to confirm the current state of the mailbox.

This signature is stored and made publicly available , which is then used by the off-chain relay and the inter-chain security modules in the network. Validators are not networked and do not need consensus; nor do they perform regular transactions on the onchain.

Following this guide, you should be able to run a Hyperlane validator on any of the existing chains on which the protocol runs. Hyperlane validators are run on the original chains.

In this guide, we will run the validator on the Base Mainnet chain, but you can run it on any other chain.

_**Relayer**_

Each Hyperlane message requires two transactions to be delivered: one in the sender chain to send the message, and one in the receiver chain to receive it. A relay is responsible for sending the second transaction.

Hyperlane relayers are configured to relay messages between one or more origin and destination chains. Relayer does not have special permissions in Hyperlane. If Relayer keys are compromised, only the tokens stored on those keys are at risk.

**Validator installation**

_**Minimum technical specifications**_

_2 CPU, 2GB RAM, 4GB SSD_

_**Server preparation**_

```
sudo apt update
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev libgmp3-dev tar clang bsdmainutils ncdu unzip llvm libudev-dev make protobuf-compiler -y
sudo apt-get install git cargo clang cmake build-essential pkg-config openssl libssl-dev protobuf-compiler
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

_**Install Foundry**_

```
curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup
```

_**Install Screen**_

```
sudo apt screen
```



_**Install Hyperlane CLI**_

```
npm install -g @hyperlane-xyz/cli
```

If there are errors during installation and NPM cusses, install NVM and then install CLI again.

```
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.nvm/nvm.sh
nvm install --lts
nvm use --lts
npm --version
```

_**Next, initialize the multisig ISM configuration**_

`hyperlane core init --advanced`



Select - _messageIdMultisigIsm_

_Enter threshold of validators (number) for message ID multisig ISM - **writing 1**_

_Enter validator addresses (comma separated list) for message ID multisig ISM - **writing your EVM public address**_

_Proxy Admin -_ **Enter Your EVM Address**

_Select default hook type - **select merkleTreeHook**_

Next, select - _aggregationHook **and**_**&#x20;enter 1**

Next, **select protocolFee** and answer the following questions::

_For protocol fee hook, enter owner address — **your EVM public address**_

_Use this same address — **YES**_

_Enter max protocol fee for protocol fee hook — **1**_

_Enter protocol fee for protocol fee hook — **0.0001**_

<figure><img src=".gitbook/assets/Снимок экрана 2024-09-05 095747.png" alt=""><figcaption></figcaption></figure>

_**Deploy Contracts**_

&#x20;_**Your selected network Base Mainnet should have at least**_ _**0.0004 ETH, Linea - 0.002 ETH, Base Sepolia - 0.005 ETH**_

```
hyperlane core deploy
```

HYP\_Key — **enter your Private key EVM (Metamask)**

_Select network type — **select testnet or mainnet**_

_Select chain to connect —_ **I chose** _Base Mainnet_

_Do you want to use an API key to verify on this (base) chain’s block explorer — **N (No) and ENTER**_



**We are now ready to generate an agent configuration using our deployed contracts**

```
hyperlane registry agent-config --chains base

```

_**Prescribing dependencies**_

`export CONFIG_FILES=$HOME/configs/agent-config.json`&#x20;

`mkdir r -p tmp/hyperlane-validator-signatures-base`

`export VALIDATOR_SIGNATURES_DIR=/tmp/hyperlane-validator-signatures-base`&#x20;

`mkdir -p $VALIDATOR_SIGNATURES_DIR`



**Starting the Validator**

_**Clone Repository**_

```
git clone git@github.com:hyperlane-xyz/hyperlane-monorepo.git
```

We will be asked for permissions in the form of an SSH Key and a password to it If you don't have an SSH Key, we'll make one.

```
ssh-keygen -t rsa -b 4096 -C your Email Github
cat ~/.ssh/id_rsa.pub
```

Copy the key that the team gave us and go to your Github page. Click “Setting” in the right corner and get to the profile settings.

Next, click SSH and GPG Keys and in the SSH key tab enter our previously copied value and clone the repository again

<figure><img src=".gitbook/assets/Снимок экрана 2024-09-05 103615.png" alt=""><figcaption></figcaption></figure>

Once the repository has been successfully cloned, open a new tab and **write the following**

```
screen -S hypval
cd hyperlane-monorepo
cd rust
cd main
```

**Run Validator**

```
cargo run --release --bin validator -- \
    --db ./hyperlane_db_validator_base \
    --originChainName base \
    --checkpointSyncer.type localStorage \
    --checkpointSyncer.path $VALIDATOR_SIGNATURES_DIR \
    --validator.key <your_validator_key
```

\<your\_validator\_key>  - Your **EVM private key (Metamask)** Be sure to **insert 0x in front of our private key !!!!** We wait for “Build” from half an hour to several hours and when it is finished we should see "Green" logs

<figure><img src=".gitbook/assets/Снимок экрана 2024-12-02 121203.png" alt=""><figcaption></figcaption></figure>

That's it! The validator has been successfully installed

We minimize our session (tab) by pressing **CTRL+A+D** and open a **new window.**



**You can install multiple validator nodes on the same Vps Server**

_**Let's install a second validator on this server example (Optimism)**_

Validators expose metrics on the port number specified by the argument `--metrics`. Port `9090` is the default, though any valid port can be chosen.

This means that we write the same validator start command as above but add an additional metric in the form of a different port than 9090. For example 9190

Open a new tab and create a new sessio&#x6E;_**`Screen -S hypvall`**_&#x20;

**Add dependencies**&#x20;

```
export CONFIG_FILES=$HOME/configs/agent-config.json
mkdir r -p tmp/hyperlane-validator-signatures-optimism
export VALIDATOR_SIGNATURES_DIR=/tmp/hyperlane-validator-signatures-optimism
mkdir -p $VALIDATOR_SIGNATURES_DIR
```



```
cargo run --release --bin validator -- \
    --db ./hyperlane_db_validator_optimism \
    --originChainName optimism \
    --checkpointSyncer.type localStorage \
    --checkpointSyncer.path $VALIDATOR_SIGNATURES_DIR \
    --validator.key <your_validator_key
    --metrics-port 9190
```

That's it. The next validator can be started on port 9290, for example.

**Validator number 3 (Arbitrum)**



**Screen -S hypvalll**

**Add dependencies**&#x20;

```
export CONFIG_FILES=$HOME/configs/agent-config.json
mkdir r -p tmp/hyperlane-validator-signatures-arbitrum
export VALIDATOR_SIGNATURES_DIR=/tmp/hyperlane-validator-signatures-arbitrum
mkdir -p $VALIDATOR_SIGNATURES_DIR
```

```
cargo run --release --bin validator -- \
    --db ./hyperlane_db_validator_arbitrum \
    --originChainName arbitrum \
    --checkpointSyncer.type localStorage \
    --checkpointSyncer.path $VALIDATOR_SIGNATURES_DIR \
    --validator.key <your_validator_key
    --metrics-port 9290
```

**Running the Validator via Docker (Example Polygon Chain)**

`mkdir -p tmp/hyperlane-validator-signatures-polygon`&#x20;

`chmod -R 777 /tmp/hyperlane-validator-signatures-polygon`



* `<polygon>`: Specify the blockchain on which you want to run your Hyperlane node.
* `<NAME>`: Provide a unique name for your validator.
* `<0xPRIVATE_KEY>`: Enter the private key&#x20;
* \--metrics port _**your port**_

```
docker run -d \
  -it \
  --name hyperlane \
  -e CONFIG_FILES=/configs/agent-config.json \
  --mount type=bind,source=/root/configs/agent-config.json,target=/configs/agent-config.json \
  --mount type=bind,source=/tmp/hyperlane-validator-signatures-polygon,target=/hyperlane_db_polygon \
  gcr.io/abacus-labs-dev/hyperlane-agent:agents-v1.0.0 \
  ./validator \
  --db /hyperlane_db_polygon \
  --originChainName polygon \
  --reorgPeriod 1 \
  --validator.id <name> \
  --checkpointSyncer.type localStorage \
  --checkpointSyncer.folder polygon \
  --checkpointSyncer.path /tmp/hyperlane-validator-signatures-polygon/polygon_checkpoints \
  --validator.key <0xprivatekey> \
  --chains.polygon.signer.key <0xprivatekey> \
  --metrics-port 9190
```

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-20 115037 (1).jpg" alt=""><figcaption></figcaption></figure>

**Logs:**

`docker ps -a`

docker logs \<id container hyperlane>



**Run Relayer**

```
screen -s hyprel
cd hyperlane-monorepo
cd rust
cd main
```

```
cargo run --release --bin relayer -- \
    --db ./hyperlane_db_relayer \
    --relayChains <chain_1_name>,<chain_2_name> \
    --allowLocalCheckpointSyncers true \
    --defaultSigner.key <your_relayer_key> \
    --metrics-port 9091
```

\<chain\_1\_name>,\<chain\_2\_name> _**Example** base,arbitrum_

\<your\_relayer\_key>  - **your private key&#x20;**_**EVM (Metamask)**_

<figure><img src=".gitbook/assets/Снимок экрана 2024-12-02 121455.png" alt=""><figcaption></figcaption></figure>

**Logs**

**Screen -ls**

```
screen -x hypval
screen -x hyprel
```

**Sending tokens using Warp Routers**

After creating our Validator, a unique “Mailbox” address is created and when using Warp Routers on the network where we created the Validator, Hyperlane CLI automatically uses it when deploying your Warp Route.

If after creating a Warp Route in SuperBridge, the same picture is observed, i.e. the bridge does not show us the amount of gas and the “Review Bridge” button is inactive:

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-10 175203.png" alt=""><figcaption></figcaption></figure>

then first make sure that when configuring the Warp Route when asked for _**“Proxy Admin" you press N (no).**_

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-10 175724.png" alt=""><figcaption></figcaption></figure>

If the problem persists, we will perform transactions through HyperlaneCLI

First of all, so that we don't have to constantly enter “Owner Address and ‘Private Key’, let's enter the _**following command:**_

```
HYP_KEY=0xprivatekey
```

_**Next, enter the following**_

```
hyperlane warp send --relay --warp $HOME/.hyperlane/deployments/warp_routes/AIDOGE/arbitrum-base-config.yaml --amount 2
```

/AIDOGE - _**Your token**_

/arbitrum-base-config.yaml - _**change to your networks**_

\--amount 2 - _**number of tokens**_

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-10 180612.png" alt=""><figcaption></figcaption></figure>

The totals shall be as follows:

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-10 180636.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Снимок экрана 2024-11-10 180657.png" alt=""><figcaption></figcaption></figure>





Official Doc — [https://docs.hyperlane.xyz/docs/guides/deploy-hyperlane-local-agents](https://docs.hyperlane.xyz/docs/guides/deploy-hyperlane-local-agents)

_Twitter_ — [https://x.com/hyperlane](https://x.com/hyperlane)

_Discord_ — [https://discord.com/invite/hyperlane](https://discord.com/invite/hyperlane)\
\
