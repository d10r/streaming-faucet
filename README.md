# About

This is an on-chain faucet (no off-chain components needed) which distributes a deploy-time configured [SuperToken](https://docs.superfluid.finance/superfluid/docs/super-tokens) via [streams](https://docs.superfluid.finance/superfluid/docs/constant-flow-agreement) to anybody asking for it.  
Asking for it is done by either invoking `StreamingFaucet.startStreamToSender()` or by sending 0 native coins to the contract address. The latter method works from any wallet, however the sender needs to make sure the wallet sets a high enough gas limit (currently ~300000) - the often default 21000 gas will lead to a failing transaction.

There's no unit tests included, the contract is simple enough to be tested interactively.  
It will work only on chains/networks with the Superfluid Framework deployed. A list with current deployments can be found [here](https://docs.superfluid.finance/superfluid/networks/networks).  
# Build and deploy

Developed and tested with nodejs v12. May or may not work with other versions. 

```
npm ci
npx truffle compile
```
Next create a file `.env` with the env vars needed for deployment.  
I then used the truffle console for deployment. E.g. for kovan:
```
npx truffle console --network kovan
```
Then in the truffle console:
```
sfHostAddr = "0xF0d7d1D47109bA426B9D8A3Cde1941327af1eea3"
superTokenAddr = "0xE3817a5e3e436a9b114A829C9FCc02355193cf41"
flowRate = 1E10
faucet = await StreamingFaucet.new(sfHostAddr, superTokenAddr, flowRate)
faucet.address
```
If it succeeded, the last command should print the address of the newly deployed contract.  
The `sfHostAddr` of this example is specific for kovan.  
The `superTokenAddr` of this example is of a SuperToken on kovan I deployed myself for testing purposes.

As a last step to get the faucet functional, you need to fund it with enough of the configured tokens. This can be done with any account and with any ERC-20 capable wallet (the `transfer` method of SuperTokens invokes the ERC-777 callback of a receiver contract if implemented - as is the case here).  
_Enough tokens_ here means: in order to allow opening streams, the Superfluid framework requires a small deposit. The deposit amount is proportional to the flowrate and depends on the time window for liquidations which is network specific, but usually in the range of a few hours.  
The first successful funding action sets the sender account as the `sponsor` of the contract. From then on, only that sponsor can provide additional funds and/or withdraw all tokens.

Note that on testnet deployments SuperTokens may not be monitored by liquidation agents and unfunded streams may not get closed as expected.
