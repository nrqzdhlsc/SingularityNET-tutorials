-----

原文链接：https://developers.google.com/protocol-buffers/docs/overview

译者：BING

时间：20190527

----

## 开发者指引

欢迎来到protocol buffers的开发者文档--一个语言中立、平台中立。可扩展的序列化结构数据，可用于通信协议，数据存储等等的数据格式。

本文档旨在帮助这些想用protocol buffers的Java，C++，Python开发者们在他们的应用中使用该数据格式。

本概览首先会介绍protocol buffers，并且告诉你需要了解哪些东西来开始这段历程--然后你就可以进入到 [教程](https://developers.google.com/protocol-buffers/docs/tutorials) 或者深入了解 [protocol buffer编码](https://developers.google.com/protocol-buffers/docs/encoding)。API[参考手册](https://developers.google.com/protocol-buffers/docs/reference/overview)也会提供这是那种语言，包含编写`.proto`文件的语言和风格指引。

### 什么是protocol buffers?

Protocol buffers是一种灵活的，高效的，自动的序列化结构数据的方法。像XML，但是更小，更快，更简单。你可以定义一次想要结构化的数据格式，然后你就可以使用特殊的代码从不同的数据流用不同的语言读写结构化数据。你甚至可以更新数据结构，而不用破坏已经部署的程序，这程序是使用旧版本的数据格式编译的。

### 它们是如何工作的？

你可以指定你想要序列化的数据格式，在`.proto`文件中定义protocol buffer的`message`类型。

每一个`message`都是信息的一段逻辑记录，包含一系列名-值对。下面是一段非常基础的`.proto`文件案例，定义了包含一个人的信息：

```protobuf
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}
```

如你所见，`message`的格式很简单，每个`message`类型都有一个或多个独特的**记号字段**，每个字段有名字和数值类型。数值类型可以是数字(整数，浮点数)，布尔值，字符串，原生字节，后者是其他的`message`类型（可嵌套）。可允许你层次化结构数据。你看指定可选字段，必选字段以及重复字段。你能在[Protocol Buffer语言指导手册](https://developers.google.com/protocol-buffers/docs/proto).找到更多编写`.proto`文件的指引。

一旦定义好了`messages`，就可以为你的应用程序语言运行protocol buffer编译器编译`.proto`文件以**生成数据访问类**。这些为每个字段提供了简单访问器(像`name()`和`set_name()`)，同时提供了序列化/解析整个结构的方法，翻译为原生字节，或者解析原生字节。因此，如果你用C++，运行编译器，可以将上面的文件编译为一个`Person`类。你可以在程序中用这个类，增加数据，序列化数据，读取数据等。你也可以编写下面的代码：

```c++
Person person;
person.set_name("John Doe");
person.set_id(1234);
person.set_email("jdoe@example.com");
fstream output("myfile", ios::out | ios::binary);
person.SerializeToOstream($output);
```

然后，你可以读取你的数据：

```c++
fstream input("myfile", ios::in | ios::binary);
Person person;
person.ParseFromIstream(&input);
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

你可以为`message`格式**添加新的字段，而不打破向后兼容性**，旧的二进制格式在解析时会忽略新的字段。因此，如果你有一个通讯协议使用了protocol buffers作为数据格式，你可以扩展协议，而不用担心破坏现存的代码。

你可以在 [API参考部分](https://developers.google.com/protocol-buffers/docs/reference/overview)获得完整的参考，关于如何使用生成protocol buffer的代码。同时，你能在[Protocol Buffer编码](https://developers.google.com/protocol-buffers/docs/encoding)找到更多关于protocol buffer信息编码的细节。

### 为什么不用XML呢？

Protocol buffers有很多比XML序列化结构数据的有点。它：

- 更简单
- 数据量小3~10倍
- 快20~100倍
- 语义更清晰
- 生成数据访问类，更容易在程序中使用

For example, let's say you want to model a `person` with a `name` and an `email`. In XML, you need to do

比如，假定你想要一个`person`类模型，有两个字段，`name`和`email`，在XML中，你需要：这么写：

```xml
<person>
  	<name>John Doe</name>
  	<email>jdoe@example.com</email>
</person>
```

对应的protocol buffer的`message`写法如下：

```protobuf
person {
	name: "John Doe",
	email: "jdoe@example.com"
}
```

当这个`message`被编码为protocol buffer二进制格式时，它大约是28字节的长度，需要100~200纳秒来解析。注意这里的文本格式只是为了方便人类阅读，方便调试和编辑。XML版本至少需要69字节，即使删除了空白，大约也需要5000~10000纳秒来解析。

同时，操作protocol buffer格式更简单：

```c++
 cout << "Name: " << person.name() << endl;
 cout << "E-mail: " << person.email() << endl;
```

而操作XML则需要这样做：

```c++
cout << "Name: "
       << person.getElementsByTagName("name")->item(0)->innerText()
       << endl;
cout << "E-mail: "
       << person.getElementsByTagName("email")->item(0)->innerText()
       << endl;
```

但是，protocol buffers并非总是优于XML，比如，protocol buffers就不适合对文本文档建模，比如HTML。因为你很难插入数据到文本结构。XML是人类易读，且人类易于编辑的格式。protocol buffers，至少它原本格式就不适合。XML从某种程度上说是自我描述的。protocol buffers只是在你有`message`定（.proto格式）义的时候才知道其意义。

### proto3简介

最新版本是第三版，介绍了新的语言版本，也添加了新的语言特性。更易用且扩展到了更多语言：我们当前版本可以用Java, C++, Python, Java Lite, Ruby和Object-C以及C#。同时你能用最新的GO语言的protoc插件来生成proto3，参考这个 [golang/protobuf](https://github.com/golang/protobuf) Github仓库。更多语言支持正在进行中。

注意，两张语言版本的API并不完全兼容。为了避免不便，我们将继续支持先前的版本。

你可以在这里[release notes](https://github.com/protocolbuffers/protobuf/releases)看到大部分的不同点，以及学习最新的proto3的语法e [Proto3 Language Guide](https://developers.google.com/protocol-buffers/docs/proto3)。完整的proto3的文档即将上线。

### 一点点历史

Protocol buffers 最初由谷歌开发，为了应对索引服务请求/响应协议。在protocol buffers 之前，有一种针对请求响应的格式，用手工处理请求响应的方式，这支持很多版本的协议，代码很丑：

```js
 if (version == 3) {
   ...
 } else if (version > 4) {
   if (version == 5) {
     ...
   }
   ...
 }
```

显式格式化的协议也会使新协议版本的推出变得复杂，因为开发人员必须确保请求发起人和处理请求的实际服务器之间的所有服务器都了解新协议，然后才能开始使用新协议。

Protocol buffers设计用于解决以下许多问题：

- 可以很容易地引入新字段，而不需要检查数据的中间服务器可以简单地解析数据并传递数据，而无需了解所有字段。
- 格式更能自我描述，并且可以用多种语言（C++、Java等）来处理。

但是用户仍然需要手写解析代码。

As the system evolved, it acquired a number of other features and uses:

随着系统的升级，它还需要更多新的特征并使用：

- 自动生成的序列化和反序列化代码以避免手工解析。
- 除了用于短期的RPC（远程过程调用）请求之外，人们开始使用protocol buffers作为一种方便的自我描述格式来持久地存储数据（例如，在bigtable中）

- 服务器RPC接口开始声明为协议文件的一部分，协议编译器生成stub类，用户可以使用服务器接口的实际实现重写这些存根类。

Protocol buffers现在是Google的**数据语言**——在编写时，在Google代码树中12183.proto份文件里定义了48162种不同的`message`类型。它们既用于RPC系统，也用于在各种存储系统中持久存储数据。

除非另有说明，否则此页面的内容将根据Creative Commons Attribution 4.0许可证进行许可，代码示例将根据Apache 2.0许可证进行许可。有关详细信息，请参阅我们的网站策略。Java是Oracle和/或其附属公司的注册商标。

END.

