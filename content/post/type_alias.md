---
title: "类型别名的用途"
date: 2018-01-07T22:04:26+08:00
tags: [
    "golang",
]
---

通常我们使用类型别名只是为了提高代码的可读性，但是今天看到一段有趣的代码，利用类型别名解决了一个递归调用的问题

```golang
package main

import (
	"encoding/json"
	"fmt"
)

type Author struct {
	ID    uint64 `json:"id"`
	Email string `json:"email"`
}

type author Author

func (a *Author) UnmarshalJSON(b []byte) (err error) {
	j, s, n := author{}, "", uint64(0)
	if err = json.Unmarshal(b, &j); err == nil {
		*a = Author(j)
		return
	}
	if err = json.Unmarshal(b, &s); err == nil {
		a.Email = s
		return
	}
	if err = json.Unmarshal(b, &n); err == nil {
		a.ID = n
	}
	return
}

func main() {
	var r Author
	fmt.Println("ERROR:", json.Unmarshal([]byte(`{
	  "author": {
	    "id":    1234567890,
	    "email": "nospam@westartup.eu"
	  }
	}`), &r))
	fmt.Println("RECORDS:", r)
}
```
注意这里的27行，`j`的类型是`author`

如果类型为`Author`，则程序会报错
```
runtime: goroutine stack exceeds 1000000000-byte limit
fatal error: stack overflow
```

这里就是利用的类型别名继承字段，但是不继承方法的特性，来避免了`UnmarshalJSON`方法的递归调用

当然，原文中对`JSON`的处理并不认为有很强的这种需求，但是处理的方法确实比较有意思

另：对`map`做的类型别名定义，可以继承索引操作符，不知何解

### 参考
5. [JSON decoding in Go](https://attilaolah.eu/2013/11/29/json-decoding-in-go/)