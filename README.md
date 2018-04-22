# Reverse Proxy [![GoDoc](https://godoc.org/github.com/go-mego/rproxy?status.svg)](https://godoc.org/github.com/go-mego/rproxy)

Reverse Proxy 套件能夠將某個請求代理到另一個後端伺服器，這在開發時避免跨域請求十分有用。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
	* [服務轉發](#服務轉發)
    * [負載平衡](#負載平衡)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/rproxy
```

# 使用方式

透過 `rproxy.New` 初始化一個反向代理中介軟體，並傳入 Mego 引擎的 `Use` 中來在所有的路由中使用。

```go
package main

import (
	"github.com/go-mego/mego"
	"github.com/go-mego/rproxy"
)

func main() {
	m := mego.New()
	// 將反向代理的中介軟體作為全域中介軟體，就能夠在所有路由中使用。
	m.Use(rproxy.New())
	m.Run()
}
```

反向代理中介軟體也可以只用在單一路由中。

```go
func main() {
	m := mego.New()
	// 反向代理中介軟體也能夠僅用於單路由上。
	m.GET("/api/v1/*", rproxy.New(), func(r *rproxy.Proxy) {
		// ...
	})
	m.Run()
}
```

## 服務轉發

透過 `Serve` 將客戶端請求反向代理至指定伺服器或埠口，並參雜自訂設置。

```go
func main() {
	m := mego.New()
	// 將所有 `/api/v1/` 的任何請求都反向代理至 `:1234` 埠口。
	m.GET("/api/v1/*", rproxy.New(), func(r *rproxy.Proxy) {
		// 透過 `Serve` 來將請求轉發至指定伺服器。
		r.Serve(&rproxy.Options{
			Schema: "HTTP",
			Host:   "localhost:1234",
			// 反向代理時可以增加額外標頭。
			Header: &rproxy.Headers{
				"X-Is-Proxy": "Yes",
			},
		})
	})
	m.Run()
}
```

## 負載平衡

反向代理同時也能與 Mego 的負載平衡套件一同使用，還請參閱 [Service Discovery](https://github.com/go-mego/sd) 套件。

```go
package main

import (
	"github.com/go-mego/mego"
	"github.com/go-mego/rproxy"
	"github.com/go-mego/sd/lb"
)

func main() {
	m := mego.New()
	// 將請求以負載平衡的方式反向代理至三個節點的其中一個。
	m.GET("/api/v1/*", rproxy.New(), lb.New(&lb.Nodes{
		"localhost:1234",
		"localhost:4567",
		"localhost:8910",
	}), func(r *rproxy.Proxy, l *lb.Balancer) {
		r.Serve(&rproxy.Options{
			Schema: "HTTP",
			// 透過負載平衡輪詢取得一個可用的節點作為反向目標主機。
			Host: l.Get().String(),
		})
	})
	m.Run()
}
```