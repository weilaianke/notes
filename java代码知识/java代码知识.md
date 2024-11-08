#java代码简易知识
#一基础方法
###1.如何在代码里面使用POST或GET请求
```
Map<String, String> accessParams = new HashMap<>();
accessParams.put("appId", appId);
accessParams.put("timestamp", timestamp);
accessParams.put("sign", sign);
accessParams.put("accessToken", accessToken);
 Map<String, Object> map =restTemplate.exchange(
 serverUrl + "/auth/user/info", // URL endpoint
 HttpMethod.POST, // HTTP method
 new HttpEntity<>(accessParams, createHeaders()),
 Map.class // Expected response type
);
```
这个方法是通过restTemplate.exchange()方法来发送POST请求，并且将请求参数封装成一个HttpEntity对象，最后通过createHeaders()方法来设置请求头。
##2.如何把信息放在header中
```
ServerHttpRequest.Builder mutate2 = request.mutate();
mutate.header("accessToken", accessToken);
```
这里面放到是request.mutate()，然后通过header()方法来设置。
```
HttpHeaders headers = request.getHeaders();
String accessToken = headers.getFirst("accessToken");
```
这里面是获取header里面的信息，通过getFirst()方法来获取。

