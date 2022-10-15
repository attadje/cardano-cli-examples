# cardano-cli-examples
Code examples using cardano-cli to create Transactions, Native Token, NFT from scratch. 

**Table of contents**
- [Prerequisites](#Prerequisites)
- [Verify everything is set up properly](#Verify-everything-is-set-up-properly)
- [How to create an address](https://github.com/attadje/cardano-cli-examples/blob/main/1-address/how-to-create-an-address.md)
- [How to create a transaction](https://github.com/attadje/cardano-cli-examples/blob/main/2-simple-transaction/how-to-create-a-simple-tx.md)
- [How to create a multi-witness transaction](https://github.com/attadje/cardano-cli-examples/blob/main/3-multi-witness-tx/how-to-create-a-multi-witness-tx.md)
- [How to create a multi-sign transaction](https://github.com/attadje/cardano-cli-examples/blob/main/4-multi-sign-tx/how-to-create-a-multi-sign-tx.md)

## Prerequisites

- cardano-node / cardano-cli set up on local machine (https://docs.cardano.org/projects/cardano-node/en/latest)
- In the examples i use some bash command.
- The example was created using jupyter lab.
Jupyter Notebook allows to share code and run it in the same user interface.  
It can combine code from different languages (Python, bash, ...), graphics, visualizations and text in shareable notebooks that run in a web browser.  
If you have jupyter lab installed or want to install it, the notebook with the code is saved in the same emplacement that the markdown file.
```
pip install jupyterlab
```

## Verify everything is set up properly

cardano-cli

```
cardano-cli version
```

output should be similar:

```
cardano-cli 1.35.3 - darwin-aarch64 - ghc-8.10
git rev 950c4e222086fed5ca53564e642434ce9307b0b9
```

cardano-node

```
cardano-node version
```

output should be similar:

```
cardano-node 1.35.3 - darwin-aarch64 - ghc-8.10
git rev 950c4e222086fed5ca53564e642434ce9307b0b9
```

jupyter-lab
```
jupyter lab --version
```

output should be similar:
```
3.1.18
```



