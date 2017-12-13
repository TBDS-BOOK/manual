## AccessKey调用portal API

### 介绍

套件portal提供主要的resful api都经过单点登录认证，保证用户通过浏览器访问调用portal接口的时候进行过单点登录操作。如果想使用代码进行接口调用就要自己做模拟登陆后才可以进行使用，增加了客户端代码的使用难度。因此我们开发了另一套可以基于accesskey 认证的方式调用portal api的方案。

### 原理：

通过添加header实现   
header名称：Authorization   
header值：通过获取AccessKey并加密获取；格式：TBDS SecretId Timestamp Nonce Signature   
其中：TBDS 为固定字符串，表示以TBDS方式去认证

```
          SecretId 是步骤1生成签名串所用的SecretId 
          Timestamp 是生成签名串所用的Timestamp ,单位：毫秒 
          Nonce 是生成签名串所用的Nonce 
          Signature 是根据步骤2Demo方式生成的签名串
```

TBDS、SecretId、Timestamp、Nonce、Signature直接以空格拼

### 示例：

在http请求头中添加

```
--header "Authorization:TBDS CHTZh3auty0S6gyjJkN6k8G3VGw5nS2GjrSK 1508161241426 64 FDen%2FTOQ9Q%2F%2BV60H8KBn65wWVEI%3D"
```

备注：若添加的api 不在 /api/_ 的请求目录下，如添加/openapi/platform/_的api需要通过AccessKey 方式调用，则修改 tbds-portal-common/resources/custom.properties 的 portal.auth.urls ，添加需要控制的url请求。  
\#api need user login  
portal.auth.urls=/api/_,_.html,/openapi/platform/\*



##### 1．根据用户id获取用户的AccessInfo

```
GET /openapi/access/header/{userId}
```

返回结果：

```
{
    "resultCode": "0",
    "message": null,
    "resultData": {
        "id": 1,
        "secureId": "CHTZh3auty0S6gyjJkN6k8G3VGw5nS2GjrSK",
        "secureKey": "S2GqXVOJvdAjykRrJJdAvsyv4fi5lxxi",
        "userName": "admin",
        "userId": 1,
        "creator": "admin",
        "module": "ALL",
        "operationType": "ALL",
        "enable": "true",
        "createTime": "2017-10-12 16:31:11",
        "updateTime": "2017-10-12 16:31:11",
        "operTypes": ["ALL"]
    }
}
```

##### 2．获取header信息, DEMO

调用：getAccessAuthHeader获取

```
package utils;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.digest.HmacUtils;
import pojo.AccessKey;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Date;
import java.util.Random;

public class AccessUtils {

    public static String getAccessAuthHeader(String prefix_url, Integer userId) throws Exception {
        String request_url = prefix_url + "/openapi/access/header/" + userId;
        String content = ConnectionUtils.get(request_url);
        AccessKey accessKey = getAccessKeyByRequestContent(content);
        if(accessKey == null){return  null;}
        System.out.println("access info=" + accessKey.toString());
        Long timestamp = new Date().getTime();
        Integer nonce = new Random().nextInt(10*8) + 1;
        String signature = AccessUtils.generateSignature(accessKey.getSecureId(), timestamp, nonce, accessKey.getSecureKey());
        String accessHeader =  "TBDS " + accessKey.getSecureId() + " " + timestamp + " " + nonce + " " + signature;
        System.out.println("access encode=" + accessHeader);
        return accessHeader;
    }

    private static AccessKey getAccessKeyByRequestContent(String content){
        JSONObject object = JSONObject.parseObject(content);
        if("0".equals(object.getString("resultCode"))){//操作成功
            return JSON.parseObject(object.getString("resultData"), AccessKey.class);
        } else { //操作失败
            System.out.println(object.getString("message")); //获取失败信息
            return null;
        }
    }

    private static String generateSignature(String secureId, long timestamp, int randomValue, String secureKey) {
        Base64 base64 = new Base64();
        byte[] baseStr = base64.encode(HmacUtils.hmacSha1(secureKey, secureId + timestamp + randomValue));
        String result = "";
        try {
            result = URLEncoder.encode(new String(baseStr), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

获取TBDS认证工具Jar

传入两个参数：1、portal url; 2、用户Id

```
java -jar api-test-1.0-SNAPSHOT-jar-with-dependencies.jar http://10.151.139.105 1
```

##### 3．调用方式

```
curl -X GET --header "Authorization:TBDS CHTZh3auty0S6gyjJkN6k8G3VGw5nS2GjrSK 1508161241426 64 FDen%2FTOQ9Q%2F%2BV60H8KBn65wWVEI%3D" http://10.254.99.17/api/whoami
```



