## ethclient

ethclient is extend [go-ethereum](https://github.com/ethereum/go-ethereum) client. 

- Add `BalanceOf` for query token balance

- Add `CalculatePoolAddressV2` `CalculatePoolAddressV3` for calculate uniswap pool address offline

- Refactor `Call` for call smart contract method

- Add `BuildContractTx` for contract transaction
## install

```
go get github.com/ackermanx/ethereum
```

## usage
Below is an example which shows some common use cases for ethclient.  Check [ethclient_test.go](https://github.com/ackermanx/ethereum/blob/main/client/ethclient_test.go) for more usage.

### get balance

```go
package main

import (
	"context"
	"log"
	"math/big"
	"time"

	"github.com/ackermanx/ethereum/client"
	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
)

var (
	testKey = "33f46d353f191f8067dc7d256e9d9ee7a2a3300649ff7c70fe1cd7e5d5237da5"
)

func main() {
	var binanceMainnet = `https://data-seed-prebsc-1-s1.binance.org:8545`

	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	c, err := client.DialContext(ctx, binanceMainnet)
	cancel()
	if err != nil {
		panic(err)
	}

	// get latest height
	ctx, cancel = context.WithTimeout(context.Background(), time.Second*5)
	blockNumber, err := c.BlockNumber(ctx)
	cancel()
	if err != nil {
		panic(err)
	}
	log.Println("latest block number: ", blockNumber)

	// get busd balance
	busdContractAddress := common.HexToAddress("0xed24fc36d5ee211ea25a80239fb8c4cfd80f12ee")
	address := "0xe96e6b50db659935878f6f3b0491B7F192cf5F59"
	bnbBalance, err := c.BalanceAt(context.Background(), common.HexToAddress(address), nil)
	if err != nil {
		panic(err)
	}
	log.Println("bnbBalance: ", bnbBalance.String())
	balance, err := c.BalanceOf(address, busdContractAddress.String())
	if err != nil {
		panic(err)
	}
	log.Printf("address busd balance: %s\n", balance.String())

	// build contract transfer
	tx, err := c.BuildContractTx(
		testKey, "transfer",
		client.ERC20Abi,
		&busdContractAddress, &bind.TransactOpts{From: common.HexToAddress(address)},
		common.HexToAddress("0x38F32C2473a314d447d681D30e1C0f5D07194371"),
		big.NewInt(100000000000000000),
	)
	if err != nil {
		panic(err)
	}
	err = c.SendTransaction(context.Background(), tx)
	if err != nil {
		panic(err)
	}
	log.Println(tx.Hash().String())
}

```

### generate pool address offline

```go
package main

import (
	"fmt"
	"math/big"

	"github.com/ackermanx/ethereum/uniswap"
)

func main() {
	weth := "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"
	dai := "0x6b175474e89094c44da98b954eedeac495271d0f"
	pair, err := uniswap.CalculatePoolAddressV2(weth, dai)
	if err != nil {
		panic(err)
	}

	fmt.Printf("weth-dai pair address in uniswap v2: %s\n", pair.String())

	fee := big.NewInt(3000)
	poolAddress, err := uniswap.CalculatePoolAddressV3(weth, dai, fee)
	if err != nil {
		panic(err)
	}
	fmt.Printf("weth-dai pool address in uniswap v3: %s\n", poolAddress.String())
}

```

## JetBrains OS licenses

`ethclient` had been being developed with GoLand under the **free JetBrains Open Source license(s)** granted by JetBrains s.r.o., hence I would like to express my thanks here.

<a href="https://www.jetbrains.com/?from=ethclient" target="_blank"><img src="https://raw.githubusercontent.com/ackermanx/ethclient/main/docs/images/jetbrains/jetbrains-variant-3.svg" width="216" align="middle" align="middle"/></a>
