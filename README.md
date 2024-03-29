# GoCryptfire

The official Cryptfire Go client - GoCryptfire allows you to interact with the Cryptfire V1 API.

## Installation

```sh
go get -u github.com/cryptfire/gocryptfire/v1
```

## Usage
Cryptfire uses a personal access token (PAT) to interact/authenticate with the
APIs. Generate an API Key from the [API menu](https://early.cryptfire/settings) 
in the Cryptfire Customer Portal.

To instantiate a GoCryptfire client, invoke `NewClient()`. Most operations require
that you pass a PAT to an `oauth2` library to create the `*http.Client`, which
configures the `Authorization` header with your PAT as the `bearer api-key`. If 
a PAT is not provided, public operations like listing plans or applications
will still work.

The client has three optional parameters:

- BaseUrl: Change the Cryptfire default base URL
- UserAgent: Change the Cryptfire default UserAgent
- RateLimit: Set a delay between calls. Cryptfire limits the rate of back-to-back calls. Use this parameter to avoid rate-limit errors.

### Example Client Setup

```go
package main

import (
  "context"
  "os"

  "github.com/cryptfire/gocryptfire/v1"
  "golang.org/x/oauth2"
)

func main() {
  apiKey := os.Getenv("CryptfireAPIKey")

  config := &oauth2.Config{}
  ctx := context.Background()
  ts := config.TokenSource(ctx, &oauth2.Token{AccessToken: apiKey})
  CryptfireClient := goCryptfire.NewClient(oauth2.NewClient(ctx, ts))

  // Optional changes
  _ = CryptfireClient.SetBaseURL("https://api.cryptfire.io")
  CryptfireClient.SetUserAgent("mycool-app")
  CryptfireClient.SetRateLimit(500)
}
```

Passing `nil` to `NewClient` will work for routes that do not require
authentication.

```go
  ... 

  CryptfireClient := gocryptfire.NewClient(nil)
  ctx := context.Background()
  plans, _, _, err := CryptfireClient.Plan.List(ctx, "", nil)

  ...
```

### Example Usage

Create an Account

```go
accountData := &goCryptfire.AccountCreateReq{
  Email:                "foo@bar.com",
  Phone:                "+19173734363",
  Country:              "US"
  UID:                  "noone"
}

res, err := CryptfireClient.Account.Create(context.Background(), accountData)

if err != nil {
  fmt.Println(err)
}
```

## Pagination

GoCryptfire v2 introduces pagination for all list calls. Each list call returns a
`meta` struct containing the total amount of items in the list and
next/previous links to navigate the paging.

```go
// Meta represents the available pagination information
type Meta struct {
  Total int `json:"total"`
  Links *Links
}

// Links represent the next/previous cursor in your pagination calls
type Links struct {
  Next string `json:"next"`
  Prev string `json:"prev"`
}

```
Pass a `per_page` value to the `list_options` struct to adjust the number of
items returned per call. The default is 100 items per page and max is 500 items
per page.

This example demonstrates how to retrieve all of your instances, with one
instance per page.

```go
listOptions := &gocryptfire.ListOptions{PerPage: 1}
for {
    i, meta, err := client.Instance.List(ctx, listOptions)
    if err != nil {
        return nil, err
    }
    for _, v := range i {
        fmt.Println(v)
    }

    if meta.Links.Next == "" {
        break
    } else {
        listOptions.Cursor = meta.Links.Next
        continue
    }
}
```

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE) file for details.
