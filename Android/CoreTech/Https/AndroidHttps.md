<h1 align="center">Android开发使用Https</h1>

[TOC]

## keystore

**`orbickeystore`文件需要放到assets目录下，并且通过**[KeyStore Explorer](http://keystore-explorer.sourceforge.net/)**工具转换为BKS-V1格式才能使用。**

详细信息：[Https](Https.md)

## Retrofit+okhttp+https

```java
public class IoTAuth {

    private static final String TAG = "IoTAuth";
    private static final String PSW = "iotauth";// orbickeystore文件密码

    private final static String BASE_URL = "https://baidu.com:8443/";

    private OkHttpClient mOkHttpClient;
    private Retrofit mRetrofit;

    public Retrofit initRetrofit() {
        if (mRetrofit == null) {
            initOkHttpClient();
            mRetrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .client(mOkHttpClient)
                    .addConverterFactory(ScalarsConverterFactory.create())
                    .build();
        }
        return mRetrofit;
    }

    private void initOkHttpClient() {
        if (mOkHttpClient == null) {
            SSLSocketFactory factory = registerKeyWithHttpsURLConnection();
            mOkHttpClient = new OkHttpClient.Builder()
                    .connectTimeout(10000, TimeUnit.MILLISECONDS)
                    .writeTimeout(10000, TimeUnit.MILLISECONDS)
                    .readTimeout(10000, TimeUnit.MILLISECONDS)
                    //设置Https请求
                    .sslSocketFactory(factory, mX509TrustManager)
                    .hostnameVerifier(hv)
                    .build();
        }
    }

    private SSLSocketFactory registerKeyWithHttpsURLConnection() {
        Log.d(TAG, "Register certificates with HttpsURLConnection");
        char[] psw = PSW.toCharArray();
        try {
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            InputStream pubKey = HealthApplication.getInstance().getResources().getAssets().open("orbickeystore");
            keyStore.load(pubKey, psw);
            KeyManagerFactory kmf = KeyManagerFactory.getInstance("X509");
            kmf.init(keyStore, psw);
            TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            tmf.init(keyStore);
            TrustManager[] tm = tmf.getTrustManagers();
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(kmf.getKeyManagers(), tm, null);
            return sslContext.getSocketFactory();
        } catch (KeyStoreException
                | KeyManagementException
                | NoSuchAlgorithmException
                | IOException
                | CertificateException
                | UnrecoverableKeyException e) {
            e.printStackTrace();
        }
        return null;
    }

    private final HostnameVerifier hv = (urlHostName, session) -> {
        if (!urlHostName.equalsIgnoreCase(session.getPeerHost())) {
            Log.d(TAG, "Warning: URL host '" + urlHostName + "' is different to SSLSession host '"
                    + session.getPeerHost() + "'.");
        }
        return true;
    };

    private final X509TrustManager mX509TrustManager = new X509TrustManager() {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {

        }

        @Override
        public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException, CertificateException {
            for (X509Certificate cert : x509Certificates) {
                cert.checkValidity();
            }
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    };
}

```

