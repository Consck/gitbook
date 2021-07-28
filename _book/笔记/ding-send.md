## 钉钉机器人消息发送
> 官方文档：https://developers.dingtalk.com/document/app/custom-robot-access/title-7ur-3ok-s1a

### 下载依赖包，添加依赖到服务

下载地址：https://developers.dingtalk.com/document/app/download-the-server-side-sdk
添加依赖：
```
<dependency>
    <groupId>com.dingtalk.open</groupId>
    <artifactId>taobao-sdk-java-auto</artifactId>
    <version>20210727</version>
    <scope>system</scope>
    <!--根据实际情况修改自己的依赖包位置和版本-->
    <systemPath>${basedir}/lib/taobao-sdk-java-auto_1479188381469-20210727.jar</systemPath>
</dependency>
```
### 代码实现

钉钉机器人支持发送多种消息类型，发送markdown消息，代码示例如下

```
public static void send(){
    String first = "https://oapi.dingtalk.com/robot/send?access_token=9b248e6924ee970c3e7741604fbcf7526fcd046d40f4d7dcd1c7ac7fb899d59f";
    String two = "SECee9b2ae9e57a3f30d2f93db74404e4fff3be379d00c2d63366a1ff23288fb92b";
    Long timestamp = System.currentTimeMillis();
    String stringToSign = timestamp + "\n" + two;
    String sign = null;
    try {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(two.getBytes("UTF-8"), "HmacSHA256"));
        byte[] signData = mac.doFinal(stringToSign.getBytes("UTF-8"));
        String base64 = new String(Base64.encodeBase64(signData));
        sign = URLEncoder.encode(base64, "UTF-8");
    } catch (Exception ex) {
        log.info("签名异常");
    }
    String url =  first + "&timestamp=" + timestamp + "&sign=" + sign;

    DingTalkClient client = new DefaultDingTalkClient(url);
    OapiRobotSendRequest request = new OapiRobotSendRequest();
    request.setMsgtype("markdown");
    OapiRobotSendRequest.Markdown markdown = new OapiRobotSendRequest.Markdown();
    markdown.setTitle("test");

    StringBuilder stringBuilder = new StringBuilder("");
    stringBuilder.append("> 来自我发来的消息").append("\n- 爱你呦").append("\n- @176XXXXX001 我成功了哈哈哈");

    OapiRobotSendRequest.At at = new OapiRobotSendRequest.At();
    //艾特某个人
    at.setAtMobiles(Arrays.asList("176XXXXX001","186XXXXX582"));
    //艾特所有人
    //at.setIsAtAll(true);
    request.setAt(at);

    markdown.setText(stringBuilder.toString());
    request.setMarkdown(markdown);


    OapiRobotSendResponse response = null;
    try {
        response = client.execute(request);
    } catch (Exception e) {
        log.error("1111");
    }
    System.out.println(response);
}
```

