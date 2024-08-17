okex
====
[![Go Reference](https://pkg.go.dev/badge/github.com/amir-the-h/okex.svg)](https://pkg.go.dev/github.com/amir-the-h/okex)
[![GitHub go.mod Go version of a Go module](https://img.shields.io/github/go-mod/go-version/amir-the-h/okex.svg)](https://github.com/amir-the-h/okex)
[![GoReportCard example](https://goreportcard.com/badge/github.com/amir-the-h/okex)](https://goreportcard.com/report/github.com/amir-the-h/okex)
[![GitHub license](https://img.shields.io/github/license/amir-the-h/okex.svg)](https://github.com/amir-the-h/okex/blob/main/LICENSE)
[![GitHub release](https://img.shields.io/github/release/amir-the-h/okex.svg)](https://GitHub.com/amir-the-h/okex/releases/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)
[![CI](https://github.com/amir-the-h/okex/actions/workflows/main.yml/badge.svg)](https://github.com/amir-the-h/okex/actions/workflows/main.yml)
[![CodeQL](https://github.com/amir-the-h/okex/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/amir-the-h/okex/actions/workflows/codeql-analysis.yml)
[![AutoRelease](https://github.com/amir-the-h/okex/actions/workflows/release.yml/badge.svg)](https://github.com/amir-the-h/okex/actions/workflows/release.yml)

*NOTICE:*
> PACKAGE IS CURRENTLY UNDER HEAVY DEVELOPMENT AND THERE IS NO GUARANTY FOR STABILITY.

*DISCLAIMER*: 
> This package is provided as-is, without any express or implied warranties. The user assumes all risks associated with the use of this package. The author and contributors to this package shall not be held liable for any damages arising from the use of this package, including but not limited to direct, indirect, incidental, or consequential damages. This package is not intended to be used as a substitute for professional financial advice. Users are responsible for verifying the accuracy and reliability of the data generated by this package. Use of this package constitutes acceptance of these terms and conditions.

Okex V5 Golang API

A complete golang wrapper for [OKX](https://www.okx.com) V5 API. Pretty simple and easy to use. For more info about
OKX V5 API [read here](https://www.okx.com/docs-v5/en).

Installation
-----------------

```bash
go get github.com/amir-the-h/okex@v1.1.4-alpha
```

Usage
-----------

```go
package main

import (
  "context"
  "github.com/amir-the-h/okex"
  "github.com/amir-the-h/okex/api"
  "github.com/amir-the-h/okex/events"
  "github.com/amir-the-h/okex/events/public"
  ws_public_requests "github.com/amir-the-h/okex/requests/ws/public"
  "log"
)

func main() {
  apiKey := "YOUR-API-KEY"
  secretKey := "YOUR-SECRET-KEY"
  passphrase := "YOUR-PASS-PHRASE"
  dest := okex.NormalServer // The main API server
  ctx := context.Background()
  client, err := api.NewClient(ctx, apiKey, secretKey, passphrase, &dest)
  if err != nil {
    log.Fatalln(err)
  }


  log.Println("Starting")
  errChan := make(chan *events.Error)
  subChan := make(chan *events.Subscribe)
  uSubChan := make(chan *events.Unsubscribe)
  logChan := make(chan *events.Login)
  sucChan := make(chan *events.Success)
  client.Ws.SetChannels(errChan, subChan, uSubChan, logChan, sucChan)

  obCh := make(chan *public.OrderBook)
  err = client.Ws.Public.OrderBook(ws_public_requests.OrderBook{
    InstID: "BTC-USD-SWAP",
    Channel: "books",
  }, obCh)
  if err != nil {
    log.Fatalln(err)
  }

  for {
    select {
    case <-logChan:
      log.Print("[Authorized]")
    case success := <-sucChan:
      log.Printf("[SUCCESS]\t%+v", success)
    case sub := <-subChan:
      channel, _ := sub.Arg.Get("channel")
      log.Printf("[Subscribed]\t%s", channel)
    case uSub := <-uSubChan:
      channel, _ := uSub.Arg.Get("channel")
      log.Printf("[Unsubscribed]\t%s", channel)
    case err := <-client.Ws.ErrChan:
      log.Printf("[Error]\t%+v", err)
      for _, datum := range err.Data {
        log.Printf("[Error]\t\t%+v", datum)
      }
    case i := <-obCh:
      ch, _ := i.Arg.Get("channel")
      log.Printf("[Event]\t%s", ch)
      for _, p := range i.Books {
        for i := len(p.Asks) - 1; i >= 0; i-- {
          log.Printf("\t\tAsk\t%+v", p.Asks[i])
        }
        for _, bid := range p.Bids {
          log.Printf("\t\tBid\t%+v", bid)
        }
      }
    case b := <-client.Ws.DoneChan:
      log.Printf("[End]:\t%v", b)
      return
    }
  }
}
```

Supporting APIs
---------------

* [Rest](https://www.okex.com/docs-v5/en/#rest-api)
  * [Trade](https://www.okex.com/docs-v5/en/#rest-api-trade) (except demo special trading endpoints)
  * [Funding](https://www.okex.com/docs-v5/en/#rest-api-funding)
  * [Account](https://www.okex.com/docs-v5/en/#rest-api-account)
  * [SubAccount](https://www.okex.com/docs-v5/en/#rest-api-subaccount)
  * [Market Data](https://www.okex.com/docs-v5/en/#rest-api-market-data)
  * [Public Data](https://www.okex.com/docs-v5/en/#rest-api-public-data)
  * [Trading Data](https://www.okex.com/docs-v5/en/#rest-api-trading-data)

[comment]: <> (    * [Status]&#40;https://www.okex.com/docs-v5/en/#rest-api-status&#41;)

* [Ws](https://www.okex.com/docs-v5/en/#websocket-api)
  * [Private Channel](https://www.okex.com/docs-v5/en/#websocket-api-private-channel) (except demo special trading
    endpoints)
  * [Public Channel](https://www.okex.com/docs-v5/en/#websocket-api-public-channels)
  * [Trade](https://www.okex.com/docs-v5/en/#websocket-api-trade)

Features
--------

* All [requests](/requests), [responses](/responses), and [events](events) are well typed and will convert into the
  language built-in types instead of using API's strings. *Note that zero values will be replaced with non-existing
  data.*
* Fully automated authorization steps for both [REST](/api/rest) and [WS](/api/ws)
* To receive websocket events you can choose [RawEventChan](/api/ws/client.go#L25)
  , [StructuredEventChan](/api/ws/client.go#L28), or provide your own
  channels. [More info](https://github.com/amir-the-h/okex/wiki/Handling-WS-events) 
