# 基于微信支付官网的 [wxpay-sdk-3.0.9](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=11_1) 版本, 有修改

## 1. 改动

1. `WXPay` 类的默认签名类型改为 MD5

2. `WXPayConfig` 类的抽象方法改成 public 修饰

3. `WXPayConfig` 类增加方法: `public abstract String getNotifyUrl();`

4. `WXPayConfig` 类增加了 `getWXPayDomain()` 方法的实现

5. 修改了 `README` 文件的 `MyConfig` 类的示例

5. `WXPayConfig` 类增加方法: `public abstract boolean useSandbox();`

## 2. maven 地址

```xml
<dependency>
    <groupId>red.htt</groupId>
    <artifactId>wxpay-sdk</artifactId>
    <version>3.0.9.1</version>
</dependency>
```

<br/>
<br/>
<br/>

# 微信支付 Java SDK

对[微信支付开发者文档](https://pay.weixin.qq.com/wiki/doc/api/index.html)中给出的 API 进行了封装。

com.github.wxpay.sdk.WXPay 类下提供了对应的方法：

| 方法名           | 说明              |
| ---------------- | ----------------- |
| microPay         | 刷卡支付          |
| unifiedOrder     | 统一下单          |
| orderQuery       | 查询订单          |
| reverse          | 撤销订单          |
| closeOrder       | 关闭订单          |
| refund           | 申请退款          |
| refundQuery      | 查询退款          |
| downloadBill     | 下载对账单        |
| report           | 交易保障          |
| shortUrl         | 转换短链接        |
| authCodeToOpenid | 授权码查询 openid |

- 注意:
- 证书文件不能放在 web 服务器虚拟目录，应放在有访问权限控制的目录中，防止被他人下载
- 建议将证书文件名改为复杂且不容易猜测的文件名
- 商户服务器要做好病毒和木马防护工作，不被非法侵入者窃取证书文件
- 请妥善保管商户支付密钥、公众帐号 SECRET，避免密钥泄露
- 参数为`Map<String, String>`对象，返回类型也是`Map<String, String>`
- 方法内部会将参数会转换成含有`appid`、`mch_id`、`nonce_str`、`sign\_type`和`sign`的 XML
- 可选 HMAC-SHA256 算法和 MD5 算法签名
- 通过 HTTPS 请求得到返回数据后会对其做必要的处理（例如验证签名，签名错误则抛出异常）
- 对于 downloadBill，无论是否成功都返回 Map，且都含有`return_code`和`return_msg`，若成功，其中`return_code`为`SUCCESS`，另外`data`对应对账单数据

## 示例

MyConfig:

```java
package com.github.wxpay.sdk;


import java.io.*;

public class MyConfig extends WXPayConfig {

    private byte[] certData;

    public MyConfig() throws Exception {
        String certPath = "/path/to/apiclient_cert.p12";
        File file = new File(certPath);
        InputStream certStream = new FileInputStream(file);
        this.certData = new byte[(int) file.length()];
        certStream.read(this.certData);
        certStream.close();
    }

    @Override
    public String getAppID() {
        return "wx8888888888888888";
    }

    @Override
    public String getMchID() {
        return "12888888";
    }

    @Override
    public String getKey() {
        return "88888888888888888888888888888888";
    }

    @Override
    public String getNotifyUrl() {
        return "http://u54j5r.natappfree.cc";
    }

    @Override
    public InputStream getCertStream() {
        return new ByteArrayInputStream(this.certData);
    }

    @Override
    public int getHttpConnectTimeoutMs() {
        return 8000;
    }

    @Override
    public int getHttpReadTimeoutMs() {
        return 10000;
    }
}
```

统一下单：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("body", "腾讯充值中心-QQ会员充值");
        data.put("out_trade_no", "2016090910595900000012");
        data.put("device_info", "");
        data.put("fee_type", "CNY");
        data.put("total_fee", "1");
        data.put("spbill_create_ip", "123.12.12.123");
        data.put("notify_url", config.getNotifyUrl());
        data.put("trade_type", "NATIVE");  // 此处指定为扫码支付
        data.put("product_id", "12");

        try {
            Map<String, String> resp = wxpay.unifiedOrder(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

订单查询：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("out_trade_no", "2016090910595900000012");

        try {
            Map<String, String> resp = wxpay.orderQuery(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

退款查询：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("out_trade_no", "2016090910595900000012");

        try {
            Map<String, String> resp = wxpay.refundQuery(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

下载对账单：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("bill_date", "20140603");
        data.put("bill_type", "ALL");

        try {
            Map<String, String> resp = wxpay.downloadBill(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

其他 API 的使用和上面类似。

暂时不支持下载压缩格式的对账单，但可以使用该 SDK 生成请求用的 XML 数据：

```java
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayUtil;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("bill_date", "20140603");
        data.put("bill_type", "ALL");
        data.put("tar_type", "GZIP");

        try {
            data = wxpay.fillRequestData(data);
            System.out.println(WXPayUtil.mapToXml(data));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

收到支付结果通知时，需要验证签名，可以这样做：

```java

import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayUtil;

import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        String notifyData = "...."; // 支付结果通知的xml格式数据

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> notifyMap = WXPayUtil.xmlToMap(notifyData);  // 转换成map

        if (wxpay.isPayResultNotifySignatureValid(notifyMap)) {
            // 签名正确
            // 进行处理。
            // 注意特殊情况：订单已经退款，但收到了支付结果成功的通知，不应把商户侧订单状态从退款改成支付成功
        }
        else {
            // 签名错误，如果数据里没有sign字段，也认为是签名错误
        }
    }

}
```

HTTPS 请求可选 HMAC-SHA256 算法和 MD5 算法签名：

```
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayConstants;

public class WXPayExample {

    public static void main(String[] args) throws Exception {
        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config, WXPayConstants.SignType.HMACSHA256);
        // ......
    }
}
```

若需要使用 sandbox 环境：

```
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayConstants;

public class WXPayExample {

    public static void main(String[] args) throws Exception {
        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config, WXPayConstants.SignType.MD5, true);
        // ......
    }

}
```
