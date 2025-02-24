dify-java-client
---

<p style="text-align: left">
    <a href="https://openjdk.org/projects/jdk/17" target="_blank">
        <img alt="maven-central" src="https://img.shields.io/badge/Java-17-blue" /> 
    </a>
    <a href="https://central.sonatype.com/artifact/io.github.yuanbaobaoo/dify-java-client" target="_blank">
        <img alt="maven-central" src="https://img.shields.io/badge/maven--central-0.0.1-green" /> 
    </a>
</p>

Dify Java 客户端，适用于Dify V1 系列API

### 快速开始
- 环境需求  
```code
Java : >= 17
Maven: >= 3
```

- maven
```xml
<dependency>
    <groupId>io.github.yuanbaobaoo</groupId>
    <artifactId>dify-java-client</artifactId>
    <version>0.15.3</version>
</dependency>
```

#### 创建客户端
```java
IDifyClient client = DifyClientBuilder.create()
        .apiKey("app-xxxx")
        .baseUrl("http://localhost:4000/v1")
        .build();

// 调用公共API
String metaInfo = client.getAppMetaInfo();
// 调用自定义API
String result = client.requestJson("/messages", HttpMethod.GET, null, null);
```
除了IDifyClient外，还提供了IDifyChatClient，IDifyChatClient继承自IDifyClient，提供了会话相关的API：
```java
IDifyChatClient client = DifyClientBuilder.create()
        .apiKey("app-xxxx")
        .baseUrl("http://localhost:4000/v1")
        .buildChat();

// 创建消息
ParamMessage m = ParamMessage.builder().query("你是谁").user("abc-123").inputs(new HashMap<>() {{
    put("test", "value");
}}).build();

// 发送阻塞消息
DifyChatResult result = client.sendMessages(m);

// 发送流式消息
CompletableFuture<Void> future = client.sendMessagesAsync(m, (r) -> {
    System.out.println("ok: " + r.getPayload().toJSONString());
});
```

#### 创建外部知识库
当前项目并未对知识库API做实现，只声明的相关参数对象和接口   
```java
public interface IDifyKonwledgeService {
    /**
     * 知识检索
     * @param apiKey Dify传递的API KEY
     * @param args KownledgeArgs
     */
    KnowledgeResult retrieval(String apiKey, KnowledgeArgs args);

}
```
如果你有需求，可以基于声明的参数与接口进行实现即可，以下为参考代码“:
- 1、声明一个endpoint
```java
/**
 * 检索本地知识库内容
 * @param args KnowledgeArgs
 */
@PostMapping("/retrieval")
public KnowledgeResult retrieval(@RequestBody(required = false) KnowledgeArgs args, HttpServletRequest request) {
    return knowledgeService.retrieval(request.getHeader("Authorization"), args);
}
```
- 2、实现接口```io.github.yuanbaobaoo.dify.service.IKnowledgeService```
```java
class KnowledgeService implements IKnowledgeService {
    @Override
    public KnowledgeResult retrieval(String apiKey, KnowledgeArgs args) {
        // TODO 你的本地知识库检索逻辑
    }
}
```

### 支持的API
目前支持的API，可以参考 ```IDifyClient```、```IDifyChatClient```这两个接口，如果接口不满足的，可以调用requestJSON、requestMultipart进行调用
```java
// requestJson内处理了数据结构、鉴权的一些逻辑
String result = client.requestJson("/messages", HttpMethod.GET, null, null);
// requestMultipart内处理了文件上传所需要的一些逻辑
// 如果是单纯的文件上传，可以直接使用
DifyFileResult result = client.uploadFile( new File("pom.xml"), "abc-123");
```