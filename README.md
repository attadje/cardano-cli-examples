# cardano-cli-examples
Code examples using cardano-cli to create Transactions, Native Token, NFT from scratch. 

**Table of contents**
- [Prerequisites](#Prerequisites)

## Prerequisites

- cardano-node / cardano-cli set up on local machine (https://docs.cardano.org/projects/cardano-node/en/latest)
- For the examples the jupyter-lab (https://pypi.org/project/jupyterlab/).  
Jupyter Notebook allows to share code and run it in the same user interface.  
It can combine code from different languages (Python, bash, ...), graphics, visualizations and text in shareable notebooks that run in a web browser.

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



