# Go json使用的奇技淫巧


## 基本使用
```go
package main

import "encoding/json"

type A struct {
    B int
    C string
}

func main() {
    buf, err := json.Marshal(A{B: 1, C: "Hello"})
    if err != nil {
        panic(err)
    }
    println(string(buf))
}
```
## 修改结果中的key值
```go
package main

type A struct {
    B int `json:"rate"`
    C string `json:"label"`
}
```
结果
```json
{"rate": 12, "label": "request_total"}
```
## 数字转字符串
因为前端js对int64的处理可能会因为溢出导致无法准确处理，因此我们期望可以返回字符串类型。
```go
package main

type A struct {
    B int `json:",string"`
}
```
## 字段的舍弃
### 去除默认值
使用`omitempty`属性用以抛弃掉默认值的字段
```go
package main

type A struct {
    B int `json:"rate,omitempty"`
    C string `json:"label,omitempty"`
}
```
结果
```json
// case 1
{"rate": 12, "label": "request_total"}
// case 2
{}
```
> 当值为默认值时，在最终结果中会被抛弃掉，对于Number类型的`0`，string类型`""`，bool类型`false`，指针类型的`nil`，均属于此类。
### 强制去除某个属性
使用`-`属性强制去除特定的key.
```go
package main
type A struct {
    B int `json:"rate"`
    C string `json:"-"`
}
```

## 定制`Marshaler/Unmarshal`
&emsp;&emsp;在很多case 中，我们需要对结构体进行一些定制，比如枚举类型和string的转换，比如一些属性的预处理，这个时候我们就需要实现自己的`Marshal/Unmarshal`方法了。
举一个很常见的例子，在web开发中，数据的ID可能我们并不希望暴露给外界，因此希望可以做一个转换。通常情况下，我们要定义两个结构体，data model 一个，web model一个，中间通过额外的方法来做转换，有多少个结构对就需要多少个转换函数，实在是令人郁闷。

&emsp;&emsp;那么，对于上面提到的棘手的问题，有没有优雅而又简单的方法解决呢？答案是肯定的，那就是自定义实现`Marshal/Unmarshal`方法的类型。

**废话不多说，直接上代码**
```go
package base

import (
	"encoding/json"
	"strings"

	"github.com/golang/glog"
	"github.com/speps/go-hashids"
	"golang.org/x/xerrors"
)

var seed *hashids.HashID

func init() {
	Init("golang-advanced-json")
}

func Init(salt string) {
	idData := hashids.NewData()
	idData.Salt = salt
	var err error
	seed, err = hashids.NewWithData(idData)
	if err != nil {
		glog.Fatalf("initialize hash id failed, %s", err)
	}
}

type ID int64

func (i ID) MarshalText() (text []byte, err error) {
	if i < 0 {
		return nil, xerrors.Errorf("err: ID(%d) must greater than 0", i)
	}
	str, err := seed.EncodeInt64([]int64{int64(i)})
	if err != nil {
		return nil, err
	}
	return []byte(str), nil
}

func (i *ID) UnmarshalText(text []byte) error {
	is, err := seed.DecodeInt64WithError(string(text))
	if err != nil {
		return xerrors.Errorf("unmarshal id failed, %w", err)
	}
	if len(is) != 1 {
		return xerrors.Errorf("bad unmarshal id length, %d", len(is))
	}
	*i = ID(is[0])
	return nil
}

func (i ID) MarshalJSON() ([]byte, error) {
	text, err := i.MarshalText()
	if err != nil {
		return nil, err
	}
	return json.Marshal(string(text))
}

func (i *ID) UnmarshalJSON(data []byte) error {
	var text string
	err := json.Unmarshal(data, &text)
	if err != nil {
		return err
	}
	return i.UnmarshalText([]byte(text))
}

func ParseIDList(str string) ([]ID, error) {
	items := strings.Split(str, ",")
	ids := make([]ID, 0, len(items))
	for _, item := range items {
		id := new(ID)
		err := id.UnmarshalText([]byte(item))
		if err != nil {
			continue
		}
		ids = append(ids, *id)
	}
	return ids, nil
}

func ParseInt64List(str string) ([]int64, error) {
	ids, err := ParseIDList(str)
	if err != nil {
		return nil, err
	}
	int64s := make([]int64, 0, len(ids))
	for _, id := range ids {
		int64s = append(int64s, int64(id))
	}
	return int64s, nil
}

func FormatIDList(ids []ID) (string, error) {
	items := make([]string, 0, len(ids))
	for _, id := range ids {
		item, err := id.MarshalJSON()
		if err != nil {
			continue
		}
		items = append(items, string(item))
	}
	return strings.Join(items, ","), nil
}
```
那么使用呢
```go
package main

type User struct {
    UserID base.ID `json:"userId"`
    NickName string `json:"nickName"
}
```
## 懒惰解析/编码
有些情况下，我们不能第一时间来判断文档的类型，就无法通过定义结构体来处理文档了，如果使用`map[string]interface{}`,则会损失类型检查，因此我们希望可以通过文档中某些固定的key，来解析不固定的key
```go
package main

type MessageType int

const (
	MessageTypeText MessageType = iota + 1
)

type Message struct {
	MessageType  MessageType     `json:"messageType"`
	MessageId    int64           `json:"messageId"`
	MessageOwner int64           `json:"messageOwner"`
	Payload      json.RawMessage `json:"payload"`
}

type TextMessage struct {
	Text string `json:"text"`
}

type ImageMessage struct {
	Uri string `json:"uri"`
}
```
我们可以先解析到`Message`中，然后根据`MessageType`进行二次解析，这样，既能解析动态文档，也可以保证类型检查不丢失。
## 保留长整型
默认的json实现中，如果一个数据是长整型，并且对应的go中的数据类型是interface的话，就会产生解析成float64的问题，这个是绝对不能接受的。可以通过如下办法来解决：
```go
package main

import "encoding/json"

func main() {
    d := json.NewDecoder([]byte(`{"a": 1234567890987654321}`))
    d.UseNumber()
    res := make(map[string]interface{})
    d.Decode(&res)
}
```
