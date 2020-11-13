# Nervos Lumos Sign Message

A function for Nervos' Lumos framework to sign a message using an Secp256k1 private key.

### Installation

In your Node.js project directory type:

```sh
npm i @jm9k/nervos-lumos-sign-message
```

### Usage Example

```javascript
const { createTransactionFromSkeleton, generateAddress, sealTransaction, TransactionSkeleton } = require("@ckb-lumos/helpers");
const { Indexer } = require("@ckb-lumos/indexer");
const { initializeConfig } = require("@ckb-lumos/config-manager");
const { Reader, RPC } = require("ckb-js-toolkit");
const { secp256k1Blake160 } = require("@ckb-lumos/common-scripts");
const { signMessage } = require("@jm9k/nervos-lumos-sign-message");

const nodeUrl = "http://127.0.0.1:8114/";
const privateKey = "0xd00c06bfd800d27397002dca6fb0993d5ba6399b4238b2f29ee9deb97593d2bc";
const script =
{
	code_hash: "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
	hash_type: "type",
	args: "0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7"
};

async function main()
{
	initializeConfig();

	const indexer = new Indexer(nodeUrl, "./indexed-data");
	indexer.startForever();

	const address = generateAddress(script);

	let skeleton = TransactionSkeleton({ cellProvider: indexer });
	skeleton = await secp256k1Blake160.payFee(skeleton, address, 1000000n);
	skeleton = secp256k1Blake160.prepareSigningEntries(skeleton);

	const signingEntries = skeleton.get("signingEntries").toArray();
	const signature = signMessage(privateKey, signingEntries[0].message).serializeJson();
	const tx = sealTransaction(skeleton, [signature]);

	const rpc = new RPC(nodeUrl);
	const res = await rpc.send_transaction(tx);
	console.log(res);
}
main();
```
