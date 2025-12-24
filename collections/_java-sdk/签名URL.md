---
title: '在URL中包含签名'
sidebar:
  nav: java-sdk
---

SDK 提供**下载签名URL（公有/私有）**与**上传预签名URL（PUT）**能力。

- 下载签名URL：`getDownloadUrlFromPublicBucket` / `getDownloadUrlFromPrivateBucket`
- 上传预签名URL（PUT）：`getUploadUrlToPrivateBucket`

具体参数与签名规则详见[签名URL API文档](https://docs.ucloud.cn/ufile/api/authorization-url)。

### 代码示例

<div class="copyable" markdown="1">

{% highlight go linenos %}
public static void getPublicURL(String keyName, String bucketName) {
    try {
        /*
        生成公共下载URL
        */
        String public_url = UfileClient.object(Constants.OBJECT_AUTHORIZER, config)
        .getDownloadUrlFromPublicBucket(keyName, bucketName)
        .createUrl();
        JLog.D(TAG, String.format("[PublicUrl] = %s", public_url));
    } catch (UfileParamException e) {
        e.printStackTrace();
    } catch (UfileAuthorizationException e) {
        e.printStackTrace();
    } catch (UfileSignatureException e) {
        e.printStackTrace();
    } catch (UfileClientException e) {
        e.printStackTrace();
    }
}

public static void getPrivateURL(String keyName, String bucketName, int expiresDuration) {
    try {
        /*
        生成私有下载URL
        */
        //  expiresDuration 私有下载路径的有效时间，即：生成的下载URL会在 创建时刻的时间戳 + expiresDuration秒后的时刻过期
        String private_url = UfileClient.object(Constants.OBJECT_AUTHORIZER, config)
        .getDownloadUrlFromPrivateBucket(keyName, bucketName, expiresDuration)
        .createUrl();
        JLog.D(TAG, String.format("[PrivateUrl] = %s", private_url));
    } catch (UfileParamException e) {
        e.printStackTrace();
    } catch (UfileAuthorizationException e) {
        e.printStackTrace();
    } catch (UfileSignatureException e) {
          e.printStackTrace();
    } catch (UfileClientException e) {
        e.printStackTrace();
    }
}
{% endhighlight %}
</div>



### 使用签名URL上传（PUT 预签名URL）

SDK 同时提供**私有空间对象的上传预签名URL**接口 `getUploadUrlToPrivateBucket`，用于在**不暴露 API 公私钥**的情况下，允许客户端直接通过 HTTP PUT 上传文件。
> 版本支持：SDK **v2.7.5+** 支持该接口。

- 生成的 URL 有效期由 `expiresDuration` 控制（单位：秒）
- **上传请求的 `Content-Type` 必须与签名时设置的 `Content-Type` 完全一致**，否则可能导致验签失败（建议使用 `createUrlWithHeaders()` 返回的 headers）
- 注意：**部分地域预签名URL上传不支持覆盖**（同 Key 已存在时可能上传失败），建议使用唯一 Key 或先删除再上传

#### 示例：生成上传签名URL

<div class="copyable" markdown="1">

{% highlight go linenos %}
public static void getPrivateUploadURL(String keyName, String bucketName, int expiresDuration) {
    try {
        // 1) 生成私有上传（PUT）预签名 URL（建议同时获取必须的请求头）
        GenerateObjectPrivateUploadUrlApi api = UfileClient.object(Constants.OBJECT_AUTHORIZER, config)
                .getUploadUrlToPrivateBucket(keyName, bucketName, expiresDuration);

        // Content-Type 会参与签名；上传时必须一致
        api.withContentType("application/octet-stream");

        // 可选：STS 临时密钥
        // api.withSecurityToken(Constants.SECURITY_TOKEN);

        GenerateObjectPrivateUploadUrlApi.UploadUrlInfo urlInfo = api.createUrlWithHeaders();
        JLog.D(TAG, String.format("[PrivateUploadUrl] = %s", urlInfo.getUrl()));
        JLog.D(TAG, String.format("[RequiredHeaders] = %s", urlInfo.getHeaders()));
    } catch (UfileParamException e) {
        e.printStackTrace();
    } catch (UfileAuthorizationException e) {
        e.printStackTrace();
    } catch (UfileSignatureException e) {
        e.printStackTrace();
    } catch (UfileClientException e) {
        e.printStackTrace();
    }
}
{% endhighlight %}
</div>

#### 示例：使用预签名URL执行 PUT 上传（JDK 原生）

上传时可以额外设置：
- 存储类型：`X-Ufile-Storage-Class`（`STANDARD` / `IA` / `ARCHIVE`）

<div class="copyable" markdown="1">

{% highlight go linenos %}
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;

public static void putByPresignedUrl(GenerateObjectPrivateUploadUrlApi.UploadUrlInfo urlInfo,
                                     File file) throws IOException {
    HttpURLConnection conn = (HttpURLConnection) new URL(urlInfo.getUrl()).openConnection();
    conn.setRequestMethod("PUT");
    conn.setDoOutput(true);
    conn.setFixedLengthStreamingMode(file.length());

    // 生成 URL 时返回的 headers 建议原样带上（尤其是 Content-Type，参与签名）
    if (urlInfo.getHeaders() != null) {
        for (String k : urlInfo.getHeaders().keySet()) {
            conn.setRequestProperty(k, urlInfo.getHeaders().get(k));
        }
    }

    // 可选：指定存储类型（默认 STANDARD）
    // conn.setRequestProperty("X-Ufile-Storage-Class", "IA"); // STANDARD | IA | ARCHIVE


    try (OutputStream out = conn.getOutputStream();
         InputStream in = new FileInputStream(file)) {
        byte[] buf = new byte[8192];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }

    int code = conn.getResponseCode();
    if (code / 100 != 2) {
        String err;
        try (InputStream es = conn.getErrorStream()) {
            err = es == null ? "" : new String(es.readAllBytes());
        }
        throw new IOException("PUT failed, code=" + code + ", err=" + err);
    }
}
{% endhighlight %}
</div>

### 错误码

| HTTP 状态码 | RetCode | ErrMsg                 | 描述                       |
| ----------- | ------- | ---------------------- |--------------------------|
| 400         | -148653 | bucket not exists      | 存储空间不存在                  |
| 400         | -15036  | check md5 failed       | MD5校验失败                  |
| 401         | -148643 | no authorization found | 上传凭证错误或者禁止覆盖             |
| 403         | -148643 | invalid signature      | API公私钥错误或者KeyName包含%、#、? |
| 404         | -148654 | file not exist         | 源文件不存在                   |

