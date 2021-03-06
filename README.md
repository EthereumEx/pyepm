PyEPM
=====
[![Build Status](https://travis-ci.org/etherex/pyepm.svg?branch=master)](https://travis-ci.org/etherex/pyepm) [![Join the chat at https://gitter.im/etherex/pyepm](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/etherex/pyepm?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Python-based EPM (Ethereum Package Manager) for Serpent 2 and Solidity contract deployment using YAML for package definitions.

[![asciicast](https://asciinema.org/a/3rhk5uy1055d8neunwx26jw8r.png)](https://asciinema.org/a/3rhk5uy1055d8neunwx26jw8r)

## Installation
`pip install pyepm`

#### Development
```
git clone https://github.com/etherex/pyepm.git
cd pyepm
pip install -e .
```

## Requirements
Ethereum node ([go-ethereum](https://github.com/ethereum/go-ethereum), [cpp-ethereum](https://github.com/ethereum/cpp-ethereum)) with JSON RPC enabled.

## Configuration

First run `pyepm -h` to create a config file in `~/.pyepm/config` on Linux and OSX and `~/AppData/Roaming/PyEPM` on Windows.

Then edit the configuration file, make sure you set the `address` from which to deploy contracts.

You will need a package definition file in YAML format to get started (see example below). You can use your deployed contracts' names as variables (prefixed with `$`) in later `transact` or `call` steps, making contract initialization a lot easier and less dependent on fixed contract addresses.

```
-
# Set some variables.
  set:
    NameReg: "0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2"
-
# Deploy contracts
  deploy:
    NameCoin:
      contract: namecoin.se
      retry: 15
      wait: True
-
  deploy:
    Subcurrency:
      contract: subcurrency.se
      gas: 100000
      endowment: 1000000000000000000
      retry: 30
      wait: True
-
# Make transactions, here we register the previously deployed
# 'Subcurrency' contract with the deployed NameCoin
  transact:
    RegisterSubToNameCoin:
      to: $NameCoin
      sig: register:[int256,int256]:int256
      data:
        - $Subcurrency
        - SubcurrencyName
      gas: 100000
      gas_price: 10000000000000
      value: 0
      retry: 30
      wait: True
-
  transact:
    TestEncoding:
      to: $NameReg
      sig: some_method:[int256,int256,int256]:int256
      data:
        - $Subcurrency
        - 42
        - "\x01\x00"
      gas: 100000
      gas_price: 10000000000000
      value: 0
      wait: False
-
# Contract calls with return values
  call:
    GetNameFromNameCoin:
      to: $NameCoin
      sig: get_name:[int256]:int256
      data:
        - $Subcurrency
-
# Another deploy
  deploy:
    extra:
      contract: short_namecoin.se
      retry: 10
      wait: True
-
# Deploy Solidity contract
  deploy:
    Wallet:
      contract: wallet.sol
      solidity:
        - multiowned
        - daylimit
        - multisig
        - Wallet
      gas: 2500000
      retry: 30
      wait: True
-
# Transact to deployed Solidity contract name
  transact:
    ToWallet:
      to: $Wallet
      sig: kill:[$Subcurrency]:int256
      retry: 15
      wait: True
```

## Usage

`pyepm YourPackageDefinitions.yaml`

```
usage: pyepm [-h] [-v] [-r HOST] [-p PORT] [-a ADDRESS] [-g GAS] [-c CONFIG]
             [-V VERBOSITY] [-L LOGGING]
             filename [filename ...]

positional arguments:
  filename              Package definition filenames in YAML format

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -r HOST, --host HOST  <host> JSONRPC host (default: 127.0.0.1).
  -p PORT, --port PORT  <port> JSONRPC port (default: 8545).
  -a ADDRESS, --address ADDRESS
                        Set the address from which to deploy contracts.
  -g GAS, --gas GAS     Set the default amount of gas for deployment.
  -c CONFIG, --config CONFIG
                        Use another configuration file.
  -V VERBOSITY, --verbose VERBOSITY
                        <0 - 3> Set the log verbosity from 0 to 3 (default: 1)
  -L LOGGING, --logging LOGGING
                        <logger1:LEVEL,logger2:LEVEL> set the console log
                        level for logger1, logger2, etc. Empty loggername
                        means root-logger, e.g. ':DEBUG,:INFO'. Overrides '-V'
```

## TODO
- Support using variables across multiple definition files
- Export addresses of deployed contracts in a json file
- Post-deployment hooks
- Support named values (1 ether)
