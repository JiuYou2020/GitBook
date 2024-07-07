# ZipKin

报错如下

```log
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "http://localhost:9411/api/v2/spans": Connection refused: no further information
	at org.springframework.web.client.RestTemplate.createResourceAccessException(RestTemplate.java:905) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:885) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:781) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.web.client.RestTemplate.exchange(RestTemplate.java:663) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.boot.actuate.autoconfigure.tracing.zipkin.ZipkinRestTemplateSender$RestTemplateHttpPostCall.doExecute(ZipkinRestTemplateSender.java:68) ~[spring-boot-actuator-autoconfigure-3.2.0.jar:3.2.0]
	at org.springframework.boot.actuate.autoconfigure.tracing.zipkin.ZipkinRestTemplateSender$RestTemplateHttpPostCall.doExecute(ZipkinRestTemplateSender.java:48) ~[spring-boot-actuator-autoconfigure-3.2.0.jar:3.2.0]
	at zipkin2.Call$Base.execute(Call.java:391) ~[zipkin-2.23.2.jar:na]
	at zipkin2.reporter.AsyncReporter$BoundedAsyncReporter.flush(AsyncReporter.java:299) ~[zipkin-reporter-2.16.3.jar:na]
	at zipkin2.reporter.AsyncReporter$Flusher.run(AsyncReporter.java:378) ~[zipkin-reporter-2.16.3.jar:na]
	at java.base/java.lang.Thread.run(Thread.java:833) ~[na:na]
Caused by: java.net.ConnectException: Connection refused: no further information
	at java.base/sun.nio.ch.Net.pollConnect(Native Method) ~[na:na]
	at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:672) ~[na:na]
	at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:542) ~[na:na]
	at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:597) ~[na:na]
	at java.base/java.net.Socket.connect(Socket.java:633) ~[na:na]
	at java.base/sun.net.NetworkClient.doConnect(NetworkClient.java:178) ~[na:na]
	at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:531) ~[na:na]
	at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:636) ~[na:na]
	at java.base/sun.net.www.http.HttpClient.<init>(HttpClient.java:279) ~[na:na]
	at java.base/sun.net.www.http.HttpClient.New(HttpClient.java:384) ~[na:na]
	at java.base/sun.net.www.http.HttpClient.New(HttpClient.java:406) ~[na:na]
	at java.base/sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(HttpURLConnection.java:1309) ~[na:na]
	at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1242) ~[na:na]
	at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1128) ~[na:na]
	at java.base/sun.net.www.protocol.http.HttpURLConnection.connect(HttpURLConnection.java:1057) ~[na:na]
	at org.springframework.http.client.SimpleClientHttpRequest.executeInternal(SimpleClientHttpRequest.java:79) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.http.client.AbstractStreamingClientHttpRequest.executeInternal(AbstractStreamingClientHttpRequest.java:70) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:66) ~[spring-web-6.1.1.jar:6.1.1]
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:879) ~[spring-web-6.1.1.jar:6.1.1]
	... 8 common frames omitted
```

问题解决,添加如下依赖

```xml
<!-- httpclient5,tracing好像要这个,不然拒绝连接-->
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.3</version>
</dependency>
```

