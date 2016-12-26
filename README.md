# NoHttpsGlide
## Glide 在用OkHttp3.x做支持库的情况下忽略所有的Https 加载图片

  最近在做项目的时候发现Glide加载图片加载不出来, 经过检查发现加载的图片是属于Https链接.
  对比Glide官方提供的Okhttp3支持库发现 可以传入一个忽略所有HTtps OkHttpClent, 所以就有了这个项目
修改官方提供的支持库代码
---

```
@Override
    public void registerComponents(Context context, Glide glide) {
        glide.register(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(getHttpClient()));
    }
```
```
private OkHttpClient getHttpClient() {
        OkHttpClient.Builder mBuilder = new OkHttpClient.Builder();
        mBuilder.sslSocketFactory(createSSLSocketFactory(), new TrustAllManager());
        mBuilder.hostnameVerifier(new TrustAllHostnameVerifier());
        return mBuilder.build();
    }

    /**
     * 默认信任所有的证书
     * TODO 最好加上证书认证，主流App都有自己的证书
     *
     * @return
     */
    @SuppressLint("TrulyRandom")
    private static SSLSocketFactory createSSLSocketFactory() {

        SSLSocketFactory sSLSocketFactory = null;

        try {
            SSLContext sc = SSLContext.getInstance("TLS");
            sc.init(null, new TrustManager[]{new TrustAllManager()},
                    new SecureRandom());
            sSLSocketFactory = sc.getSocketFactory();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return sSLSocketFactory;
    }

    private static class TrustAllManager implements X509TrustManager {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }

    private static class TrustAllHostnameVerifier implements HostnameVerifier {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            return true;
        }
    }

```
### 使用方法
复制以下三个类到你的项目中 :

- OkHttpGlideModule
- OkHttpStreamFetcher
- OkHttpUrlLoader

同时在AndroidManifest.xml 中声明Glide的支持文件
```
<meta-data
            android:name="'Your Package'.OkHttpGlideModule"
            android:value="GlideModule" />
```

## 第一步不要忘记了  OkHttp3.0 和Glide的支持  

```
compile 'com.squareup.okhttp3:okhttp:3.5.0'
    compile 'com.github.bumptech.glide:glide:3.7.0'
```

### 好了, 这次分享到此为止, 如果帮助到你的话, 记得给颗星啊!
