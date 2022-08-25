---  
title: '比较本地文件和远程文件etag'
sidebar:
  nav: java-sdk
---
SDK 提供`CompareFileEtag`方法比较本地文件和远程文件etag，适用于文件完整性判断，例如文件上传、下载过程中是否发生丢失。

示例如下：

<div class="copyable" markdown="1">

{% highlight go linenos %}
public class ObjectETagSample {
    private static final String TAG = "GetObjectETagSample";
    private static ObjectConfig config = new ObjectConfig("cn-sh2", "ufileos.com");

    public static void main(String[] args) {
        computeLocalFileETag();
        // computeLocalFileETagUseStream();
        // getBucketObjectETag();
        // compareETag();
    }

    public static void computeLocalFileETag() {
        String path = "";
        try {
            File file = new File(path);
            Etag etag = Etag.etag(file);
            JLog.D(TAG, String.format("ETag is [%s]: file path [%s]", etag.geteTag(), path));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void computeLocalFileETagUseStream() {
        String path = "";
        try {
            InputStream in = new FileInputStream(path);
            Etag etag = Etag.etag(in);
            JLog.D(TAG, String.format("ETag is [%s]: file path [%s]", etag.geteTag(), path));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void getBucketObjectETag() {
        String keyName = "";
        String bucketName = "";

        try {
            ObjectProfile objectProfile = UfileClient.object(Constants.OBJECT_AUTHORIZER, config)
                    .objectProfile(keyName, bucketName).execute();

            if (objectProfile == null) {
                JLog.D(TAG, String.format("key [%s] may be not exist", keyName));
                return;
            }

            JLog.D(TAG, String.format("key [%s] ETag: %s", keyName, objectProfile.geteTag()));
        } catch (UfileClientException e) {
            e.printStackTrace();
        } catch (UfileServerException e) {
            e.printStackTrace();
        }
    }

    public static void compareETag() {
        String keyName = "";
        String bucketName = "";
        String localpath = "";

        try {
            File f = new File(localpath);
            if (UfileClient.object(Constants.OBJECT_AUTHORIZER, config).compareEtag(f, keyName, bucketName)) {
                JLog.D(TAG, "ETag compare success");
            } else {
                JLog.D(TAG, "ETag compare fail");
            }
        } catch (UfileClientException e) {
            e.printStackTrace();
        } catch (UfileServerException e) {
            e.printStackTrace();
        }
    }
}

{% endhighlight %}
</div>
