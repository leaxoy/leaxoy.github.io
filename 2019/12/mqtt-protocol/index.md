# Mqtt协议解析

> 这篇属于IM三剑客中的第二篇，前面一篇主要讲解了通用IM的一些架构的知识，这边主要讲解`MQTT`协议的细节，最后一篇将会着重介绍了MQTT Broker的Go语言实现。
> - [通用IM架构](https://leaxoy.github.io/2019/12/im-architecture/)
> - [Go实现Mqtt broker](https://leaxoy.github.io/2019/12/mqtt-implement-in-go/)

&emsp;&emsp;MQTT协议以其效率高，语义完善而著名，非常适合使用在移动设备中，可以大幅度的减少耗电量。相对于TCP，语义更加丰富，额外的overload小，最少只需要2byte；相对于其他的应用层协议Websocket等，头部简单，包体积更小。


## 相关术语
### Client(客户端)
### Session
### Broker
### Subscription
### Topic
### Topic Filter
### Publisher
### Subscriber

## 数据解包
&emsp;&emsp;`MQTT`数据包共分为三部分，`FixedHeader`, `Variable Header` 和 `Payload`, 其中只有`FixedHeader`是必须的，`Variable Header`和`Payload`部分是可选的。这部分着重介绍`FixedHeader`，可选部分将在下面的详细介绍中涉及。

&emsp;&emsp;固定头部的长度为`2...5 byte`，说到这里，可能有人迷糊了，既然说的是固定头部，怎么长度还有不同的选项呢。其实固定头部的意思并不是说长度固定，而是说每个数据包必须包含的意思，哪怕一个数据包什么内容都没有，也要有一个长度为2的头部数据。那么长度不固定的固定头部又该如何解释呢，因为不同的场景数据包的大小也不近相同，为了能让服务端知晓如何解析，解析到何处，就必须要把数据包的大小给编码到数据包中，数据包有大有小，如果用一个比较小的数字的话，就无法发送大于该数字的数据了，如果数据包比较小，但是编码用的数字又比较大的话，就会造成浪费的现象，所以针对不同大小的数据，需要用不同的方式对长度进行编码，这就是可变长度的由来。
>在`mqtt`中，数据包的大小由第2个byte到第5个byte来决定，最小为1个字节，最大为4个字节，加上头部固定的一个字节，正好是`2...5 byte`.

下面为`FixedHeader`的结构：

|        |  Bit 7  |  Bit 6  |  Bit 5  |  Bit 4  | Bit 3 | Bit 2 | Bit 1 |  Bit 0   |
| :----: | :-----: | :-----: | :-----: | :-----: | :---: | :---: | :---: | :------: |
| Byte 1 | MsgType | MsgType | MsgType | MsgType |  Dup  |  Qos  |  Qos  | Retained |
| Byte 2 | Has 3?  |         |         |         |       |       |       |          |
| Byte 3 | Has 4?  |         |         |         |       |       |       |          |
| Byte 4 | Has 5?  |         |         |         |       |       |       |          |
| Byte 5 |         |         |         |         |       |       |       |          |

### 第一个字节
&emsp;&emsp;第一个字节主要用做消息类型的控制，以及对应消息类型的额外的属性。按照bit分割，共有8bit，前四个bit用来表示数据包的类型，是connect还是publish，除了0和15外（用作保留类型），其他的14中均有不同的定义，下方分别给出了14中类型的详细的解释。

#### 剩下的4bit
- 第一个bit，是`Duplicated`的意思，用在`publish`中`qos`为`1/2`的情况，用来表明该消息是否为一条重复的消息。
- 第2/3个bit，声明了`qos`的级别，可选的有`(0b00, 0b01, 0b10)`，分别代表了`(QOS0, QOS1, QOS2)`，不能有3，如果是3的话，会被认为成一个非法的数据包。
- 第四个bit，`Retained`，告知broker是否要将消息持久化，以供后来的订阅者消费。
### 剩下的字节
&emsp;&emsp;这个部分用来表示剩下的数据的长度，前面说了，这个部分长度是可变的，范围为`1-4`，那么如果来确定长度是1还是其他的呢，秘密就藏在字节的最高位(7bit)，这部分的所有数据，最高位bit都表明有没有后续的长度，如果这个值为0的话，表明没有后续的数据了，如果是一的话，表明还需要继续计算长度，因此能表明的数字最小为0，最大为`128 * 128 * 128 * 128 = 268435456 byte = 256 Mb`。
## 连接与认证
### Connect(`1`)
&emsp;&emsp;当客户端建立好与服务器的连接后，客户端发送的第一个报文必须是`Connect`，并且只能发送一次，服务端在收到第二个Connect的时候，会认为客户端异常而断开连接。·

### Connack(`2`)
服务端响应，
## 心跳与保活
### Pingreq(`12`)
客户端发送，无`Variable Header`与`Payload`，

### Pingresp(`13`)

服务端响应，无`Variable Header`与`Payload`，

## 订阅
### Subscribe(`8`)
### Suback(`9`)
## 取消订阅
### Unsubscribe(`10`)
### Unsuback(`11`)
## 发布消息
### Publish(`3`)
### Pubrel(`6`)

用在`QOS2`消息的第二阶段，

## 接收消息
### Puback(`4`)

&emsp;&emsp;用在`QOS1`的消息上，当收到`QOS1`的消息后，马上回复`Puback`的消息，同时设置`MessageId`为`Publish`消息的`MessageId`，

### Pubrec(`5`)

&emsp;&emsp;用在`QOS2`消息的第一阶段

### Pubcomp(`7`)

&emsp;&emsp;用在`QOS2`消息的第二阶段

## 断开连接
### Disconnect(`14`)

&emsp;&emsp;当客户端主动断开连接时，主动发送`Disconnect`消息，该消息无`Varibale Header`与`Payload`。

------

## 服务质量保证
&emsp;&emsp;MQTT发布消息QoS保证不是端到端的，是客户端与服务器之间的。订阅者收到MQTT消息的QoS级别，最终取决于发布消息的QoS和主题订阅的QoS。

| 发布者发布的Qos | 订阅者订阅的Qos | 最终的QOS |
| :-------------: | :-------------: | :-------: |
|        0        |        0        |     0     |
|        0        |        1        |     0     |
|        0        |        2        |     0     |
|        1        |        0        |     0     |
|        1        |        1        |     1     |
|        1        |        2        |     1     |
|        2        |        0        |     0     |
|        2        |        1        |     1     |
|        2        |        2        |     2     |

> &emsp;&emsp;总而言之，言而总之，最终的QOS级别为两者中保证较弱的一方的QOS。


### Qos 0
&emsp;&emsp;Qos 0只会投递一次，没有任何到达的保证，数据从`Publisher`到`Broker`，再从`Broker`到`Subscriber`均为一次发送，**适合一些不那么重要的数据。**
{{<mermaid>}}
sequenceDiagram
    participant Publisher
    participant Broker
    participant Subscriber
    Publisher->>Broker: Publish
    Publisher->>Publisher: Delete(Msg)
    Broker->>Subscriber: Publish
    Broker->>Broker: Delete(Msg)
{{</mermaid>}}

```mermaid
sequenceDiagram
    participant Publisher
    participant Broker
    participant Subscriber
    Publisher->>Broker: Publish
    Publisher->>Publisher: Delete(Msg)
    Broker->>Subscriber: Publish
    Broker->>Broker: Delete(Msg)
```

### Qos 1

{{<mermaid>}}
sequenceDiagram
    participant Publisher
    participant Broker
    participant Subscriber
    Publisher->>Publisher: Store(Msg)
    Publisher->>Broker: Publish (msg_id = x)
    Broker->>Broker: Store(Msg)
    Broker->>Subscriber: Publish (msg_id = y)
    Broker->>Publisher: Puback (msg_id = x)
    Publisher->>Publisher: Delete(Msg)
    Subscriber->>Broker: Puback (msg_id = y)
    Broker->>Broker: Delete(Msg)
{{</mermaid>}}

```mermaid
sequenceDiagram
    participant Publisher
    participant Broker
    participant Subscriber
    Publisher->>Publisher: Store(Msg)
    Publisher->>Broker: Publish (msg_id = x)
    Broker->>Broker: Store(Msg)
    Broker->>Subscriber: Publish (msg_id = y)
    Broker->>Publisher: Puback (msg_id = x)
    Publisher->>Publisher: Delete(Msg)
    Subscriber->>Broker: Puback (msg_id = y)
    Broker->>Broker: Delete(Msg)
```

### Qos 2

{{< mermaid >}}
sequenceDiagram
    participant Publisher
    participant Broker
    participant Subscriber
    Publisher->>Publisher: Store(Msg)
    Publisher->>Broker: Publish
    Broker->>Broker: Store(Msg)
    Broker->>Publisher: Pubrec
    Publisher->>Broker: Pubrel
    Broker->>Subscriber: Publish
    Broker->>Publisher: Pubcomp
    Publisher->>Publisher: Delete(Msg)
    Subscriber->>Subscriber: Store(Msg)
    Subscriber->>Broker: Pubrec
    Broker->>Subscriber: Pubrel
    Subscriber->>Subscriber: Notify(Msg)
    Subscriber->>Broker: Pubcomp
    Broker->>Broker: Delete(Msg)
    Subscriber->>Subscriber: Delete(Msg)
{{< /mermaid >}}
