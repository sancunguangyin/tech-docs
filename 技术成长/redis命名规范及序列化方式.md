# redis命名规范及序列化方式最佳实践

## 0.结论

redis命名最佳实践：

* `gnosis:${服务名}:${业务，中间用点}:${keyid}:{redis 类型}`
* `gnosis:${实体/关系}:${业务，中间用点}:${keyid}:{redis 类型}`
* `小写命名`

redsi序列化方案：

* `使用Jackson进行序列化操作`

## 1. 命名规范

### 1.1 解决什么问题

简单、高效、可维护、业务抽象。。。

一致性：当数据库对象（如表、列、索引等）遵循一致的命名规则时，开发人员能够更容易地理解和维护数据库，因为它们知道对象的名称遵循什么样的模式和约定。这种标准化有助于提高开发效率并减少潜在的错误。

可读性：规范的命名使得数据库对象的名字更具描述性，因此不管是新的开发人员还是日后进行维护时，对于理解数据库结构和功能所需的努力都将减少。

可维护性：数据库命名规范有助于避免命名冲突和重复名称。这些问题可能会导致在修改、扩展或维护数据库时出现不必要的困难和混淆。通过遵循清晰的命名规则，可以降低这些问题发生的概率。

语义清晰：一个有意义和一致的命名规范有助于为数据库提供更好的语义，使得开发人员能够更轻松地理解数据的含义和对象之间的关系。这可以避免不正确的数据使用和查询。

总结来说，就是命名要让开发人员望名知意，没有歧义，且易拓展。

1.2 常见的命名规范

典型的命名方式是姓名，通过姓和名来划分人群。

在开发中，常用的命名规范如下

使用下划线连接多个单词
常见C/C++变量和文件命名，数据库命名
使用原因：
便于理解
通过前缀进行层

数据库中对大小写不敏感，且-号作为运算符可能参与运算

int question_task_config_count; // 题目任务配置数
FILE question_task_config_file; // 题目任务配置文件

使用横线连接多个单词
常用与HTML以及网址中
使用原因：
Google浏览器通常会将url也作为搜索项，因此使用横线连接单词易于对url进行分词

对大小写不敏感

blog-content-box
left-tool-box

使用驼峰模式命名：
计算机程序、图灵完备的编程语言常用的命名规范
使用原因：
语义化，使用大写分割单词

根据实践公认较好的命名方式

int questionTaskConfigCount;

个人总结：好的命名规范满足语言清晰、易读、易扩展，另外还要考虑到易写，太复杂也不是好的规范。实际使用中，命名规范要考虑具体的编程环境来选择。

1.2 Redis命名规范

Q：redis的命名规范为什么这么设计，它解决了什么问题？

A：redis并没有强制的命名规范，现有社区中存在一些成熟的方案，主要为了解决三个问题：可读性、可拓展性、以及可组织性。

提高可读性：通过遵循有组织的命名方式，可以使键名在视觉上更容易被理解。
改善可维护性：采用一致且有描述性的命名规范可降低理解和维护项目的难度。
提高组织性：可通过使用命名空间和一致的命名模式，更好地组织离散数据。

Q：redis命名规范的最佳实践？

A：下面给出一个redis的建议最佳实践：

【强制】使用冒号分隔命名空间：在 Redis 中，键的命名通常包含多个部分，这些部分代表了键的层次结构或命名空间。使用冒号（:）作为命名空间分隔符有助于保持键命名的清晰和明确。

例如：user:123:name

【推荐-待讨论】同一业务逻辑含义段的单词之间使用英文半角点号 (.)分割，用来表示一个完整的语义

例如：question.task.config

【强制】具有描述性：键名应具有描述性并能清晰地解释它们所表示的数据。描述性的单词能帮组共同开发者快速了解键名的含义。

例如：user:123:posts:count

【强制】遵循一致的命名规范：确保整个项目中的命名规范保持一致，比如统一使用小写字母、下划线等。这可以提高可读性和可维护性。

【强制】遵循业务需求：根据项目和业务需求为键名添加前缀或后缀。这有助于明确键名的用途，提高代码可读性。

【推荐】保持长度适中：避免使用过长的键名，因为长键名会占用更多的内存并降低可读性。但是，过短的键名又可能降低可读性，因此要权衡长度和描述性。

【推荐】命名空间划分：业务模块名:业务逻辑含义:其他:value类型

例如：question.task.config : give.up.task.ids : userId : zset

gnosis:${服务名}:${业务，中间用点}:${keyid}:{redis 类型}
gnosis:${实体/关系}:${业务，中间用点}:${keyid}:{redis 类型}
 小写命名

2.序列化
2.1 解决什么问题

序列化主要解决两个问题，存储和传输

数据持久化存储：序列化允许将对象的状态转换为可存储或可传输的格式。当需要将内存中的对象存储到文件、数据库或其他持久存储设备时，序列化是必需的。例如将内存中的对象存储到磁盘中的csv文件中时，就使用了csv格式进行序列化
通信：通过将对象序列化为字节流，可将其在不同进程之间进行传输或通过网络发送到远程计算机。接收方可以将字节流反序列化为相应的程序对象，从而实现有效的通信。例如最经典的网络协议栈，就是通过定义一套公认的协议，将应用层的数据对象，序列化成物理层传输的二进制流进行传输，然后在接受方根据协议进行反序列化获得可读的数据。
2.2 Redis中常见的序列化方法

JSON：

优点：
可读性好，通用性强。
多种编程语言支持，跨平台。
结构简单，易于理解。
缺点：
序列化和反序列化开销相对较大。
占用空间较多，因为键和值都以字符串形式保存。
不支持二进制数据和循环引用。
使用：
Jackson
Gson
FastJson-存在安全问题，不推荐使用
性能比较：
Jackson：通常在许多基准测试中具有较高的性能表现，尤其是在对象结构复杂或对象数量较多的情况下。
Gson：性能表现相对较低，但在一些简单对象中可能不会有明显影响。
FastJson：在大部分基准测试中，FastJson 的性能与 Jackson 相当。通常认为 FastJson 更适用于大型 JSON 数据的解析。

MessagePack：

原理：采用类似于Huffman编码的方式对数据进行压缩，序列化成二进制表示
 优点：
紧凑的存储格式，空间占用少。
高性能，序列化和反序列化速度快。
支持多种编程语言。
缺点：
相对于 JSON，可读性差。
对于某些简单的结构，MessagePack 的空间占用可能与 JSON 相近。
使用：JAVA中可以使用msgpack-java库
用法：

原理：采用类似于Huffman编码的方式对数据进行压缩，序列化成二进制表示
 优点：
紧凑的存储格式，空间占用少。
高性能，序列化和反序列化速度快。
支持多种编程语言。
缺点：
相对于 JSON，可读性差。
对于某些简单的结构，MessagePack 的空间占用可能与 JSON 相近。
使用：JAVA中可以使用msgpack-java库
用法：

```
<dependency>
  <groupId>org.msgpack</groupId>
  <artifactId>msgpack-core</artifactId>
  <version>0.8.16</version>
</dependency>
```

import org.msgpack.core.MessageBufferPacker;
import org.msgpack.core.MessagePack;
import org.msgpack.core.MessageUnpacker;

// 初始化 MessagePack 工具
MessagePack.PackerConfig packerConfig = new MessagePack.PackerConfig().withAutoIndent(false);
MessagePack.UnpackerConfig unpackerConfig = new MessagePack.UnpackerConfig().withAllowReadingStringAsBinary(true).withAllowReadingBinaryAsString(true);

// 使用 MessagePack 进行序列化
MessageBufferPacker packer = packerConfig.newPacker();
packer.packString("example_string");
packer.packInt(42);
byte[] packedBytes = packer.toByteArray();

// 使用 MessagePack 进行反序列化
MessageUnpacker unpacker = unpackerConfig.newUnpacker(new ByteArrayInputStream(packedBytes));
String stringValue = unpacker.unpackString();
int intValue = unpacker.unpackInt();

BSON：

原理：是一种拓展的Json二进制表示格式，相比于Json能支持更多的数据类型，常用于MongoDB数据库
 优点：
支持二进制数据类型和循环引用。
支持多种编程语言和平台。
缺点：
数据存储占用空间相对较大。
相对于 MessagePack，性能较低。
用法：Java 中可使用官方的 bson 库进行 BSON 的处理。其他编程语言可能有相应的库支持。
```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>bson</artifactId>
    <version>4.0.3</version>
</dependency>
```

```
import org.bson.Document;

// 构建 BSON 文档结构
Document document = new Document()
        .append("example_string", "value")
        .append("example_number", 42)
        .append("example_array", Arrays.asList("item1", "item2"));

// BSON 序列化
byte[] bsonBytes = document.toBsonDocument(Document.class, CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry())).toJson().getBytes();

// BSON 反序列化
Document documentFromBson = Document.parse(new String(bsonBytes));
```

Protocol Buffers：

原理：Protocol Buffers (protobuf) 是 Google 开发的一种高效的、灵活的、可扩展的二进制序列化格式，专门设计用于网络通信和数据存储。protobuf 使用特定的数据结构和类型定义语法，并通过编译器生成各个编程语言所需的数据访问类。
 优点：
高性能，序列化和反序列化速度快。
固定的格式和类型。
具有紧凑的存储格式。

支持多种编程语言。

缺点：
需要定义数据结构协议。
可读性较差，需要编译为相应的对象。
用法：需要使用 protobuf-java 及 protobuf 编译器（protoc）来处理 Protocol Buffers 定义文件（.proto 文件）并生成所需的 Java 类。

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.12.4</version>
</dependency>
```

```
// 1. 创建一个 .proto 文件来定义数据结构，例如 "example.proto" 文件，内容如下
syntax = "proto3";
option java_package = "com.example.myproject.protos";

// 2.使用 protoc 编译器从 .proto 文件生成对应的 Java 类：
message Example {
    string example_string = 1;
    int32 example_number = 2;
}
protoc --java_out=. example.proto

// 3. 可以使用生成的 Java 类进行序列化和反序列化操作
import com.example.myproject.protos.Example;

// 构建一个 Example 类实例
Example example = Example.newBuilder().setExampleString("value").setExampleNumber(42).build();

// 使用 Protocol Buffers 序列化
byte[] protobufBytes = example.toByteArray();

// 使用 Protocol Buffers 反序列化
Example exampleFromProtobuf = Example.parseFrom(protobufBytes);
```

2.3 如何选择----在B端中场景的实践

对于高性能和紧凑数据存储的需求，MessagePack 和 Protocol Buffers 可能更为合适；而对于可读性和通用性的需求，JSON 可能更为合适。

在 B 端场景中，如果对高性能和压缩存储的要求不高，而更注重数据的可读性和通用性，建议选择 JSON 作为 Redis 的序列化方法。

原因如下：

可读性：JSON 格式具有较好的可读性，因为它是一种结构化的文本表示。不像二进制序列化格式，JSON 数据可以直接在文本编辑器中查看和理解。这对于开发人员调试和分析数据非常有帮助，特别是在 B 端场景中。

通用性：JSON 是一种通用的数据交换格式，被广泛应用于各种编程语言和框架，跨平台兼容性强。这意味着，无论您目前或将来使用哪种编程语言或框架，都可以轻易地处理 JSON 数据。

易于理解：由于 JSON 的结构简单，易于理解，即使为那些不熟悉 JSON 的人员，也可以很快地了解其中的数据结构和含义。

简单用法示例：

**Jackson（公司组建支持）：**
```
import com.fenbi.common.util.JsonUtils;

// 创建一个 ObjectMapper 实例

// JSON 序列化
String jsonString = JsonUtils.writeValue(anObject);
// JSON 反序列化
MyClass anObject = JsonUtils.readValue(jsonString, MyClass.class);
```

**Gson：**
```
import com.google.gson.Gson;

// 创建一个 Gson 实例
Gson gson = new Gson();
// JSON 序列化
String jsonString = gson.toJson(anObject);
// JSON 反序列化
MyClass anObject = gson.fromJson(jsonString, MyClass.class);
```

**FastJson：**
```
import com.alibaba.fastjson.JSON;

// JSON 序列化
String jsonString = JSON.toJSONString(anObject);
// JSON 反序列化
MyClass anObject = JSON.parseObject(jsonString, MyClass.class);
```
