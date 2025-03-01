---
title: Google序列化-Protobuf的语法与实际使用
date: 2025-03-01
tags:
  - proto
  - json
  - xml
categories: Tools
description: Protocol Buffers（简称protobuf）是谷歌提供的一种新的序列化方法，比json和xml更快更小；以二进制的传输，并且利用编译器protoc进行编译，使得proto文件和二进制序列一起兼具了可读性和速度；
---
# 基础

## 0.1 序列化
- 序列化
	对象转换为字节序列（字符串）的过程
- 反序列化
	字节序列 恢复为 对象的过程
```cpp
struct User {
	string name;
	int age;
	string sex;
};

User u1 = {'Jack', 25, 'Male'};

// 序列化
string str = "Jack\n18\nMale";
```

当需要将数据存储，保存对象状态；或者网络进行传输数据时，不能传输整个对象；
> Socket编程发送与接收就是序列化与反序列化

实现序列化的手段：
Protobuf,  [[JSON]],  [[XML]]

## 0.2 Protobuf
全称 ： **Protocol Buffers** 是Google的 语言无关、平台无关、可扩展的 二进制数据**序列化协议**
可以用于通信协议、数据存储等场景的 结构化数据 存储传输

proto 比 xml 更小更快更简单
proto 可以定义数据的结构、会自动生成源代码；更新数据结构，不会破坏原有结构编译的已部署程序

## 0.3 使用简述
编写`.proto`文件，定义数据结构和属性；
用`protoc`编译器编译`.proto`，生成接口；
在目标代码中调用接口；

# Proto结构
## 1.1 语法
```protobuf
syntax = "proto3"; // 指定 proto3 语法

package example;  // 包名 (命名空间既视感)

message Person {
	string name = 1; // 字段类型 + 字段名称 + 唯一编号tag 
	int32 id = 2;
	string email = 3;
	repeated string phones = 4;
}
```
一个字段的完整是：
`<规则> <类型> <名> = <编号>;`
## 1.2 类型 与 规则
- 字段类型
```protobuf
int32 string bool bytes enum message
```
- 字段规则
```protobuf
optional // 可选
repeated // 重复  数组
oneof   // 多选一 [[Protobuf#1.4 oneof 多态]]
```
- 字段编号
唯一编号； `1~15`只占用1字节
## 1.3 嵌套
实现复杂结构
```protobuf
message Base {
	string base_field = 1;
}

message Derived {
	Base base = 1;
	string derived_field = 2;
}
```
## 1.4 oneof 多态
可以**包含多个**类型字段，但是**同时只能有一个**被设置
多选一需求，不同字段之间是**互斥的**

```protobuf
message Event {
	oneof event_type {
		ClickEvent click = 1;
		KeyPressEvent keypress = 2;
	}
}

message PersonInfo {
	string name = 1;
	oneof contact {
		string qq = 2;
		string wx = 3; 
	}
}
```
然后在代码中的具体设置
```cpp
PersonInfo person;
person.set_qq("12987");

person.set_wx("wx_23816"); // 这个时候会 覆盖 qq字段 
```

# 编译与使用
## 2.1 编译器 protoc
编写好`.proto`文件后，利用`protoc`编译器生成目标语言代码；
```bash
protoc --cpp_out=. person.proto

protoc --python_out=. person.proto

protoc --go_out=. person.proto
```
编译后会生成对应代码
`python` : `persn_pb2.py`
`go` : `person.pb.go`
`c` : `person.pb.cc` 与 `person.pb.h`

> 这种还挺常见，例如QT的`.ui`会生成`ui_xxxx.h`

## 2.2 使用编译后文件

进行序列化

```cpp
#include "person.pb.h"

// 进行序列化
Person person;
// 构造 赋值
person.set_name( "Jack" );
person.set_id( 65 );
person.set_email( "a@bb.com" );

std::string serialized = person.SerializeToString();
```

反序列化
```cpp
Person parsed_person;
parsed_person.ParseFromString( serialized );
```

## 2.3 提供的一些默认方法
`set_`,`has_`,`clear_`
`mutable_`
`add_`, `_size`
### 值操作
- 设置
```cpp
person.set_name('Jack');
```
- 获取
```cpp
std::string person_name = person.name();
```
- **存在性检查**
```cpp
if ( person.has_email() ) {
	// ...
}
```
- 清除
```cpp
person.clear_name();
```
### 可变指针 mutable
`.proto`文件内容
```protobuf
message Address {
	string city = 1;
}
message Person{
	// ...
	Address address = 2;
}
```
具体使用(C++为例)
```cpp
Address* addr = person.mutable_address();

addr->set_city("Beijing");
```
### Repeated 字段操作
字段定义`.proto`
```protobuf
message Exam {
	// ...
	repeated int32 scores = 3;
}
```
repeated的操作(C++为例)
```cpp
// 添加
exam.add_scores(90);
exam.add_scores(68);

// 获取
int score_0 = exam.scores(0);
int score_2 = exam.scores(2);

// 修改
exam.mutable_scores(0)->push_back(95);

// size
int size = exam.scores_size();
```
### 交换与合并
```cpp
// 交换 swap
Person p1, p2;

p1.Swap(&p2);

// 合并 merge
p1.MergeFrom(p2);
```
合并的话，p1字段内容会被p2覆盖或者追加

# 序列化对比
#proto #xml #json
[[JSON]] [[XML]]

|      | `proto`        | `json`                  | `xml`         |
| ---- | -------------- | ----------------------- | ------------- |
| 格式   | 二进制            | 文本 键值对                  | 文本 标签         |
| 可读性  | 低（需要解析）        | 高                       | 高（感觉不如json）   |
| 传输效率 | 体积小，序列化快       | 体积较大，解析慢                | 体积大，解析慢       |
| 类型支持 | 强类型（预定义）       |                         |               |
| 场景   | 高性能通信<br>大数据传输 | Web API<br>配置文件<br>数据交换 | 兼容旧系统<br>复杂文档 |
## 三个代码例子

### proto
```protobuf
message Order {
  int64 order_id = 1;
  repeated Product products = 2; // 强类型，需要定义 // 数组
  float total_price = 3;
  Status status = 4; // 强类型，需要定义

  enum Status { // 枚举 status
    PENDING = 0;
    SHIPPED = 1;
    DELIVERED = 2;
  }

  message Product { // 嵌套类型
    string name = 1;
    int32 quantity = 2;
    float price = 3;
  }
}
```
### json
```json
"order" : {
	"order_id" : 1,
	"products" : [
		{"name" : "mouse", "quantity" : 2, "price" : 21.5},
		{"name" : "keyboard", "quantity" : 1, "price" : 52.49}
	],
	"total_price" : 95.49,
	"status" : "PENDING"
}
```
### xml
```xml
<Order>
  <OrderID>1001</OrderID>
  <Products>
    <Product>
      <Name>Laptop</Name>
      <Quantity>1</Quantity>
      <Price>999.99</Price>
    </Product>
    <Product>
      <Name>Mouse</Name>
      <Quantity>2</Quantity>
      <Price>25.50</Price>
    </Product>
  </Products>
  <TotalPrice>1050.99</TotalPrice>
  <Status>PENDING</Status>
</Order>
```

# 通信

## 流程
- 定义接口： 设计`.proto`
- 生成代码： `protoc` 编译成目标语言
- 传输数据： 二进制格式传输 （`HTTP/2`, `gRPC`)
- 序列化/反序列化： 发送方序列化，接收方反序列化
## 场景
gRPC通信： 基于HTTP/2 和 protobuf 的高性能 RPC 框架

常用于 **后端之间** 的通信
## 前后端

前端（如浏览器），使用JavaScript，就将proto转为js
```bash
npm install -g protobufjs
pbjs -t static-module -w commonjs -o data_pb.js data.proto
```
再进行反序列化 和 序列化操作，传回序列化后的数据到后端
> 不展示代码

- 通信方式
	- HTTP API： `fetch` 发送二进制数据
	- WebSocket： 直接传输二进制流