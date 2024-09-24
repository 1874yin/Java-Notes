## 内网调用 OSS 获取图片资源返回给前端

#### 需求

客户端通过外网访问oss获取图片需要额外付费，考虑到成本问题，修改技术方案为：

客户端将请求链接发给服务端，服务端根据请求做一定的截取或拼接，通过内网调用oss，再将下载下来的图片流返回给前端。

```
package com.xiaojie.ossDownloader.service;

import okhttp3.Request;
import okhttp3.Response;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import javax.servlet.http.HttpServletResponse;

import okhttp3.OkHttpClient;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.UnsupportedEncodingException;
import java.util.Base64;
import java.util.concurrent.TimeUnit;

/**
 * 需求：客户端通过外网访问oss获取图片需要额外付费，考虑到成本问题，修改技术方案为：
 * 客户端将请求链接发给服务端，服务端根据请求做一定的截取或拼接，通过内网调用oss，
 * 再将下载下来的图片流返回给前端。
 */

@Service
public class PictureServiceImpl implements PictureService {

    @Value("${alioss.ak}")
    private String accessKeyId;

    @Value("${url.prefix}")
    private String urlPrefix;

    @Value("${oss.connect.time:3000}")
    private int ossConnectTime;


    @Override
    public boolean getOssPicture(String path, HttpServletResponse response) throws IOException {
        String url = getOssUrl(path);
        Request requestDownload = new Request.Builder()
                .url(url)
                .build();
        OkHttpClient client = new OkHttpClient();
        client = client.newBuilder().connectTimeout(ossConnectTime, TimeUnit.MILLISECONDS).build();
        Response responseDownload = client.newCall(requestDownload).execute();
        if (responseDownload.isSuccessful() && responseDownload.body() != null) {
            InputStream is = responseDownload.body().byteStream();
            writeImageFile(response, is);
        } else {
            return false;
        }
        return true;
    }

    private void writeImageFile(HttpServletResponse response, InputStream is) {
        OutputStream out = null;
        try {
            out = response.getOutputStream();
            int len = 0;
            byte[] b = new byte[1024];
            while ((len = is.read(b)) != -1) {
                out.write(b, 0, len);
            }
            out.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (out != null) {
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private String getOssUrl(String path) throws UnsupportedEncodingException {
        Base64.Decoder decoder = Base64.getDecoder();
        String decodePath = new String(decoder.decode(path), "UTF-8");
        StringBuffer sb = new StringBuffer();
        String[] split = decodePath.split("&");
        for (int i = 0; i < split.length; i++) {
            if (!split[i].startsWith("Version")) {
                sb.append(split[i]).append("&");
            }
        }
        sb.append("OSSAccessKeyId=").append(accessKeyId);
        return urlPrefix + sb;
    }
}
```