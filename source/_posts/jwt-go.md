---
title: jwt-go的使用
date: 2019-07-26 12:58:39
tags:
- jwt
- golang
categories:
- golang
preview: 100
---


# go jwt

## 什么是jwt
JSON Web Token通常用于 Oauth 2中的`Bearer` tokens

一个token分成三个部分，用`.`连接，前两个部分是用base64url编码的json对象，最后一部分是签名，使用相同的编码方式编码。


1. 第一部分叫header，包含JWT和签名算法比如HMAC, SHA256, RSA
```Go
{
"typ": "JWT",
"alg": "HS256"
}
```
2. 第二部分叫Playload(Claims)，存储数据
```Go
{
"userId": "23581935-afsakngh12i-asdfaf"
}
```
JWT规定了7个官方字段
```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

3. Signature
signature是对前两部分的签名，首先需要指定一个算法，按照下面的公式产生
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)

## jwt使用方式
客户端收到服务器的JWT后，可以放在cookie里和localstorage里，但是这样不能跨域，更好的方法是放在请求头信息。
以后每次客户端与服务器通信都要带上jwt
`Authorization: Bearer <token>`
另一种做法是跨域的时候，jwt放在post数据体中

## 对称加密方法
比如HSA使用单个密钥，所以任意`[]byte`类型都可以当作一个合法的密钥。对称加密在双方都被信任的情况下最好用。因为签名和验证使用相同的算法，所以不能简单地分发key来验证。

## 非对称加密方法
非对称加密方法，比如RSA使用不同的key来签名和验证token，这就可以用private key产生token，再允许使用者用public key来验证。

每种签名方法使用不同的类型来签名。
```
The HMAC signing method (HS256,HS384,HS512) expect []byte values for signing and validation

The RSA signing method (RS256,RS384,RS512) expect *rsa.PrivateKey for signing and *rsa.PublicKey for validation

The ECDSA signing method (ES256,ES384,ES512) expect *ecdsa.PrivateKey for signing and *ecdsa.PublicKey for validation
```

### HMAC方法演示
```Go
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go"
	"time"
)

func GenerateToken(mc *jwt.MapClaims, secret []byte) string {
// 使用HS256算法(输入HMAC,是对称加密算法)
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, mc)
	// 签名并且获得完整的字符串
	tokenString, err := token.SignedString(secret)
	if err != nil {
		fmt.Printf("GenerateToken err: %v", err)
	}
	return tokenString
}

func ParseToken(tokenString string, secret []byte) (jwt.MapClaims ,error)  {
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		// 验证加密算法
		if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
			return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
		}
		return secret, nil
	})
	// 类型
	if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
		return claims, nil
	} else {
		return nil, err
	}
}

func main() {
	secret := []byte("secret_key")
	mc := jwt.MapClaims{
		"foo": "bar",
		"issat": time.Now().Unix(),
	}
	// 打印token
	tokenString := GenerateToken(&mc, secret)
	fmt.Println(tokenString)

	// 验证token
	claims, err := ParseToken(tokenString, secret)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%v", claims)

}
```

### rsa加密方法演示
```Go
package main

import (
	"crypto/rsa"
	"fmt"
	"io/ioutil"
	"path/filepath"

	"github.com/dgrijalva/jwt-go"
)

// GenerateToken receive MapClaims and return a tokenString
func GenerateToken(mc jwt.MapClaims, privateKey *rsa.PrivateKey) (tokenString string, err error) {
	token := jwt.NewWithClaims(jwt.SigningMethodRS256, mc)
	tokenString, err = token.SignedString(privateKey)
	return
}

// ParseToken parse the token and return a MapClaims
func ParseToken(tokenString string, publicKey *rsa.PublicKey) (claims jwt.MapClaims, err error) {
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		// 验证加密方法是否符合
		if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
			return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
		}
		return publicKey, nil
	})
	if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
		return claims, nil
	}
	return
}

func main() {
	path, err := filepath.Abs("./demo.rsa")
	privateByte, err := ioutil.ReadFile(path)
	if err != nil {
		panic(err)
	}

	path, err = filepath.Abs("./demo.rsa.pub")
	publicByte, err := ioutil.ReadFile(path)
	if err != nil {
		panic(err)
	}
	privateKey, _ := jwt.ParseRSAPrivateKeyFromPEM(privateByte)
	publicKey, _ := jwt.ParseRSAPublicKeyFromPEM(publicByte)

	mc := jwt.MapClaims{
		"name": "baichen",
	}

	tokenString, err := GenerateToken(mc, privateKey)

	if err != nil {
		fmt.Printf("GenerateToken error: %v", err)
		return
	}

	claims, err := ParseToken(tokenString, publicKey)
	if err != nil {
		fmt.Printf("ParseToken error: %v", err)
	}

	fmt.Printf("%v", claims)
}

```

## 参考资料
[JWT](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
[公钥密钥生成](http://web.chacuo.net/netrsakeypair)
[go读取文本内容](https://blog.csdn.net/skh2015java/article/details/78954293)