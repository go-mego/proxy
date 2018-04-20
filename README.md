# Reverse Proxy

Reverse Proxy 套件能夠將某個請求代理到另一個後端伺服器，這在開發時避免跨域請求十分有用。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
    * [負載平衡](#負載平衡)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/rproxy
```

# 使用方式

透過 `rproxy.New` 初始化一個反向代理設置，並在之後透過 `ServeHTTP` 將客戶端資訊反向代理至指定伺服器或埠口。

```go
package main

import (
	"github.com/go-mego/mego"
	"github.com/go-mego/rproxy"
)

func main() {
	m := mego.New()
	// 將所有 `/api/v1/` 的任何請求都反向代理至 `:1234` 埠口。
	m.Get("/api/v1/*", func(c *mego.Context) {
		p := rproxy.New(rproxy.Config{
			Schema: "HTTP",
			Host:   "localhost:1234",
			// 反向代理時可以增加額外標頭。
			Header: rproxy.Headers{
				"X-Is-Proxy": "Yes",
			},
		})
		p.ServeHTTP(c.Writer, c.Request)
	})
	m.Run()
}
```

## 負載平衡

反向代理同時也能與 Mego 的負載平衡套件一同使用，還請參閱 [Service Discovery](https://github.com/go-mego/sd) 套件。