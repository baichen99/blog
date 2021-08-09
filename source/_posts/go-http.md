---
title: go net/http使用方法
date: 2019-8-13 22:10:00
tags:
- go
- http
categories:
- golang
---

## get, post

### 基本用法


```go
resp, err := http.Get("http://example.com/")

resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)

resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
	
// 使用完respone后要关闭连接
defer resp.Body.Close()

// 获取响应中的内容
b, _ := ioutil.ReadAll(resp.Body) // b 为[]byte类型 需要类型转换
s := string(b)

```

Post方法参数分别为 url, content-type, data。data要和content-type对应。[具体看这里](https://zhuanlan.zhihu.com/p/129057481)

## client
要管理HTTP客户端的头域、重定向策略和其他设置，创建一个Client：
```go
type Client struct {
	Transport RoundTripper
  CheckRedirect func(req *Request, via []*Request) error
  Jar CookieJar // 给client插入cookie，每次请求都会带上它
  Timeout time.Duration // 说明每次请求的最大时间

}
```

## header

使用 `func NewRequest(method, urlStr string, body io.Reader) (*Request, error)`创建一个request 后，可以管理header
```go
	// 参数：请求方法，url，请求数据
  req, _ := http.NewRequest("POST", "http://blacston.com", nil)
  req.Header.Add("content-type", "application/json")
```

## cookies

```go
    jar := cookiejar.Jar{}
    jar.SetCookies("url", cookies)
```

```go
type Cookie struct {
	Name  string
	Value string

	Path       string    // optional
	Domain     string    // optional
	Expires    time.Time // optional
	RawExpires string    // for reading cookies only

	// MaxAge=0 means no 'Max-Age' attribute specified.
	// MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
	// MaxAge>0 means Max-Age attribute present and given in seconds
	MaxAge   int
	Secure   bool
	HttpOnly bool
	SameSite SameSite
	Raw      string
	Unparsed []string // Raw text of unparsed attribute-value pairs
}

```



## Example

- header

  content-type: application/json

- body:

  ```json
  {
  	"username": "admin",
  	"password": "passowrd"
  }
  ```



```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strings"
)

func main() {
    client := http.DefaultClient
    reqUrl := "http://121.36.63.57:8888/user/login"
    data := make(map[string]string)
    data["username"] = "admin"
    data["password"] = "password"

    jsonData, err := json.Marshal(data)
    if err != nil {
        panic(err)
    }

    req, _ := http.NewRequest(http.MethodPost, reqUrl, strings.NewReader(string(jsonData)))

    req.Header.Set("Content-Type", "application/json")
    resp, _ := client.Do(req)
    b, _ := ioutil.ReadAll(resp.Body)
    fmt.Println(string(b))
}
```





[ShuLogin](https://github.com/AdrianDuan/CCSL/blob/master/utils/password.go)