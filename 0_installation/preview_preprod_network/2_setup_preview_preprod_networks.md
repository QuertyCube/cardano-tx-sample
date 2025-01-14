Once you have built the cardano-node version 1.35.3, please follow the below steps to sync to the preview testnet:

### Download the All_Networks_Config_Files_cardano-config.tar.gz file, extract it and add the config files respectively to below folder structure

### FOLDER ORGANIZATION
- node binaries under $HOME/cardano-node-1.35.3-linux
- testnet folder under $TESTNETPATH/ (refer environment variable below)
- preview network configuration files under $TESTNETPATH/config/preview/
- preprod network configuration files under $TESTNETPATH/config/preprod/
- legacy network configuration files under $TESTNETPATH/config/legacy/
- preview database under $TESTNETPATH/db/preview/
- preprod database under $TESTNETPATH/db/preprod/
- legacy database under $TESTNETPATH/db/legacy/
- node.socket under $TESTNETPATH


UPDATE your .bashrc / .zshrc file accordingly AND RELOAD IT AFTER SAVING IT by either restarting the terminal or running the "source ~/.bashrc" (or .zshrc) file!

```

#export HOME="/home/bharat"
export LEGACYNODE="$HOME/cardano-node-1.35.2-linux"
export CARDANO_NODE="$HOME/cardano-node-1.35.3-linux"

export PATH="$CARDANO_NODE:$LEGACYNODE:$PATH"
export TESTNETPATH="$HOME/testnet"

#magic id for preview network is 2, preprod 1
export TESTNET="--testnet-magic 1"

#legacy magic id
#export TESTNET="--testnet-magic 1097911063"

#socket path remains same, no change required as we're creating this file ourselves
export CARDANO_NODE_SOCKET_PATH="$TESTNETPATH/node.socket"

alias previewnode='$CARDANO_NODE/cardano-node run --topology $TESTNETPATH/config/preview/topology.json --database-path $TESTNETPATH/db/preview/ --socket-path $CARDANO_NODE_SOCKET_PATH --port 3001 --config $TESTNETPATH/config/preview/config.json'
alias preprodnode='$CARDANO_NODE/cardano-node run --topology $TESTNETPATH/config/preprod/topology.json --database-path $TESTNETPATH/db/preprod/ --socket-path $CARDANO_NODE_SOCKET_PATH --port 3001 --config $TESTNETPATH/config/preprod/config.json'
alias legacynode='$LEGACYNODE/cardano-node run --topology $TESTNETPATH/config/legacy/testnet-topology.json --database-path $TESTNETPATH/db/legacy/ --socket-path $CARDANO_NODE_SOCKET_PATH --port 3001 --config $TESTNETPATH/config/legacy/testnet-config.json'
alias ctip='cardano-cli query tip $TESTNET'


#Added some bash-completion scripts here for ease of execution of commands
source <(cardano-cli --bash-completion-script `which cardano-cli`)
source <(cardano-node --bash-completion-script `which cardano-node`)

function utxo() { cardano-cli query utxo $TESTNET --address $1 ; }
function utxof() { cardano-cli query utxo $TESTNET --address $(cat $1) ; }
function submit() { cardano-cli transaction submit --tx-file $1 $TESTNET ;}

```

##Ada faucet (WORKING AS OF NOW)
Use this faucet to get preview / preprod ADA: https://docs.cardano.org/cardano-testnet/tools/faucet


### if after an improper shut-down, you get the below error

```
cardano-node: The db is used by another process. File "/home/bharat/testnet/db/preview/lock" is locked.
```

then either restart the system or kill the cardano-node process (not recommended!)

```
sudo killall cardano-node
```

After this kind of an improper shutdown, you might need to delete all the files in the /testnet/db/preview or /testnet/db/preprod folder and resync the database 

For preview network
```
cd $TESTNET/db/preview
rm -rf ./*
rm -rf ./.*
```
For preprod network
```
cd $TESTNET/db/preprod
rm -rf ./*
rm -rf ./.*
```


Then re-launch the node and it will quickly sync to the latest block (takes only a minute or two right now!)

At the prompt, type the command below and hit ENTER
```
previewnode

OR

preprodnode
```

NOTE: If you use Preprod, **you can only submit alonzo-era transactions**. In Preview, you can submit babbage era txs, but **testnet.cardanoscan.io will not show your transactions**.

### NOTE2: Even if your node is working fine, you might keep getting the following type of error (with some variations):
```
TracePromoteColdFailed 50 36 161.97.170.151:3002 161.126402860273s Network.Socket.connect: <socket: 95>: does not exist (Connection refused)


[2022-08-28 16:15:10.52 UTC] TracePromoteColdFailed 50 36 161.97.170.151:3002 160.254143346317s Network.Socket.connect: <socket: 101>: does not exist (Connection refused)

Followed by a new tip message like below
:cardano.node.ChainDB:Notice:33] [2022-08-28 16:15:19.59 UTC] Chain extended, new tip: aa8f7bd7c37182de01ba8adc9dd2752ceaddfd14a99f48fa11b678a6ac6aaba2 at slot 6020119
[Kalpatar:cardano.node.ChainDB:Notice:33] [2022-08-28 16:15:36.44 UTC] Chain extended, new tip: 140c50f6ae9bd015b7e71aaf949694c9833ce92b6df58ddbfb2d169cde7ee78a at slot 6020136
```

Best way is to check if your tip block is changing after a minute or two, if not changing even after two minutes, then you might actually have some problem. :(

