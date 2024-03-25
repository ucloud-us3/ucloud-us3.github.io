---
title: '在URL中包含签名'
sidebar:
nav: java-sdk
---

SDK 提供了公共URL签名接口`getPublicURL`和私有URL签名接口`getPrivateURL`。

`getPublicURL`与`getPrivateURL`方法请求的 US3 API 为 `getDownloadUrlFromPublicBucket`与`getDownloadUrlFromPrivateBucket`，具体详见[签名URL API文档](https://docs.ucloud.cn/ufile/api/authorization-url)。

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

### 错误码

| HTTP 状态码 | RetCode | ErrMsg                 | 描述                                |
| ----------- | ------- | ---------------------- | ----------------------------------- |
| 400         | -148653 | bucket not exists      | 存储空间不存在                      |
| 400         | -15036  | check md5 failed       | MD5校验失败                         |
| 401         | -148643 | no authorization found | 上传凭证错误                        |
| 403         | -148643 | invalid signature      | API公私钥错误或者KeyName包含%、#、? |
| 404         | -148654 | file not exist         | 源文件不存在                        |

