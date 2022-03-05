<h1 align="center">Https</h1>

[toc]

## 1.Android: Trusting SSL certificates

We use [a self-signed SSL certificate](http://en.wikipedia.org/wiki/Self-signed_certificate) for the test version of our backend web service. Since our certificate isn't signed by a [CA](http://en.wikipedia.org/wiki/Certificate_authority) that Android trusts by default, we need to add our server's public certificate to our Android app's trusted store.

These same instructions apply to trusting a custom CA, except you'd get the public certificate directly from the CA instead of from a server.

Required tools:

- [OpenSSL](http://www.openssl.org/)'s command line client
- [Java SE 6](http://java.sun.com/javase/downloads/index.jsp) (for `keytool`)
- [Bouncy Castle's provider jar](http://www.bouncycastle.org/latest_releases.html)

1. Grab the public certificate from the server you want to trust. Replace `${MY_SERVER}` with your server's address.

```bash
echo | openssl s_client -connect ${MY_SERVER}:443 2>&1 | \
 sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > mycert.pem
```

For example, here's the PEM-encoded public certificate from google.com:

```
-----BEGIN CERTIFICATE-----
MIIDITCCAoqgAwIBAgIQL9+89q6RUm0PmqPfQDQ+mjANBgkqhkiG9w0BAQUFADBM
MQswCQYDVQQGEwJaQTElMCMGA1UEChMcVGhhd3RlIENvbnN1bHRpbmcgKFB0eSkg
THRkLjEWMBQGA1UEAxMNVGhhd3RlIFNHQyBDQTAeFw0wOTEyMTgwMDAwMDBaFw0x
MTEyMTgyMzU5NTlaMGgxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlh
MRYwFAYDVQQHFA1Nb3VudGFpbiBWaWV3MRMwEQYDVQQKFApHb29nbGUgSW5jMRcw
FQYDVQQDFA53d3cuZ29vZ2xlLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkC
gYEA6PmGD5D6htffvXImttdEAoN4c9kCKO+IRTn7EOh8rqk41XXGOOsKFQebg+jN
gtXj9xVoRaELGYW84u+E593y17iYwqG7tcFR39SDAqc9BkJb4SLD3muFXxzW2k6L
05vuuWciKh0R73mkszeK9P4Y/bz5RiNQl/Os/CRGK1w7t0UCAwEAAaOB5zCB5DAM
BgNVHRMBAf8EAjAAMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6Ly9jcmwudGhhd3Rl
LmNvbS9UaGF3dGVTR0NDQS5jcmwwKAYDVR0lBCEwHwYIKwYBBQUHAwEGCCsGAQUF
BwMCBglghkgBhvhCBAEwcgYIKwYBBQUHAQEEZjBkMCIGCCsGAQUFBzABhhZodHRw
Oi8vb2NzcC50aGF3dGUuY29tMD4GCCsGAQUFBzAChjJodHRwOi8vd3d3LnRoYXd0
ZS5jb20vcmVwb3NpdG9yeS9UaGF3dGVfU0dDX0NBLmNydDANBgkqhkiG9w0BAQUF
AAOBgQCfQ89bxFApsb/isJr/aiEdLRLDLE5a+RLizrmCUi3nHX4adpaQedEkUjh5
u2ONgJd8IyAPkU0Wueru9G2Jysa9zCRo1kNbzipYvzwY4OA8Ys+WAi0oR1A04Se6
z5nRUP8pJcA2NhUzUnC+MY+f6H/nEQyNv4SgQhqAibAxWEEHXw==
-----END CERTIFICATE-----
```

2. Android has built-in support for the Bouncy Castle keystore format (BKS). Put Bouncy Castle's jar in your classpath, and create a keystore containing only your trusted key.

```bash
export CLASSPATH=bcprov-jdk16-145.jar
CERTSTORE=res/raw/mystore.bks
if [ -a $CERTSTORE ]; then
    rm $CERTSTORE || exit 1
fi
keytool \
      -import \
      -v \
      -trustcacerts \
      -alias 0 \
      -file <(openssl x509 -in mycert.pem) \
      -keystore $CERTSTORE \
      -storetype BKS \
      -provider org.bouncycastle.jce.provider.BouncyCastleProvider \
      -providerpath /usr/share/java/bcprov.jar \
      -storepass ez24get
```

3. Create a custom Apache `HttpClient` that uses your custom store for HTTPS connections.

```java
import android.content.Context;
import org.apache.http.conn.ClientConnectionManager;
import org.apache.http.conn.scheme.PlainSocketFactory;
import org.apache.http.conn.scheme.Scheme;
import org.apache.http.conn.scheme.SchemeRegistry;
import org.apache.http.conn.ssl.SSLSocketFactory;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.conn.SingleClientConnManager;

import java.io.InputStream;
import java.security.KeyStore;

public class MyHttpClient extends DefaultHttpClient {

  final Context context;

  public MyHttpClient(Context context) {
    this.context = context;
  }

  @Override protected ClientConnectionManager createClientConnectionManager() {
    SchemeRegistry registry = new SchemeRegistry();
    registry.register(
        new Scheme("http", PlainSocketFactory.getSocketFactory(), 80));
    registry.register(new Scheme("https", newSslSocketFactory(), 443));
    return new SingleClientConnManager(getParams(), registry);
  }

  private SSLSocketFactory newSslSocketFactory() {
    try {
      KeyStore trusted = KeyStore.getInstance("BKS");
      InputStream in = context.getResources().openRawResource(R.raw.mystore);
      try {
        trusted.load(in, "ez24get".toCharArray());
      } finally {
        in.close();
      }
      return new SSLSocketFactory(trusted);
    } catch (Exception e) {
      throw new AssertionError(e);
    }
  }
}
```

That's it! If you think this kind of stuff is fun, [Square is hiring](https://squareup.com/jobs).



## 2.Trusting all certificates using HttpClient over HTTPS

You basically have four potential solutions to fix a "Not Trusted" exception on Android using httpclient:

1. Trust all certificates. Don't do this, unless you really know what you're doing.
2. Create a custom SSLSocketFactory that trusts only your certificate. This works as long as you know exactly which servers you're going to connect to, but as soon as you need to connect to a new server with a different SSL certificate, you'll need to update your app.
3. Create a keystore file that contains Android's "master list" of certificates, then add your own. If any of those certs expire down the road, you are responsible for updating them in your app. I can't think of a reason to do this.
4. Create a custom SSLSocketFactory that uses the built-in certificate KeyStore, but falls back on an alternate KeyStore for anything that fails to verify with the default.

This answer uses solution #4, which seems to me to be the most robust.

The solution is to use an SSLSocketFactory that can accept multiple KeyStores, allowing you to supply your own KeyStore with your own certificates. This allows you to load additional top-level certificates such as Thawte that might be missing on some Android devices. It also allows you to load your own self-signed certificates as well. It will use the built-in default device certificates first, and fall back on your additional certificates only as necessary.

First, you'll want to determine which cert you are missing in your KeyStore. Run the following command:

```java
openssl s_client -connect www.yourserver.com:443
```

And you'll see output like the following:

```java
Certificate chain
 0 s:/O=www.yourserver.com/OU=Go to 
   https://www.thawte.com/repository/index.html/OU=Thawte SSL123 
   certificate/OU=Domain Validated/CN=www.yourserver.com
   i:/C=US/O=Thawte, Inc./OU=Domain Validated SSL/CN=Thawte DV SSL CA
 1 s:/C=US/O=Thawte, Inc./OU=Domain Validated SSL/CN=Thawte DV SSL CA
   i:/C=US/O=thawte, Inc./OU=Certification Services Division/OU=(c) 
   2006 thawte, Inc. - For authorized use only/CN=thawte Primary Root CA
```

As you can see, our root certificate is from Thawte. Go to your provider's website and find the corresponding certificate. For us, it was [here](https://www.thawte.com/roots/index.html), and you can see that the one we needed was the one Copyright 2006.

If you're using a self-signed certificate, you didn't need to do the previous step since you already have your signing certificate.

Then, create a keystore file containing the missing signing certificate. Crazybob has [details how to do this on Android](http://blog.crazybob.org/2010/02/android-trusting-ssl-certificates.html), but the idea is to do the following:

If you don't have it already, download the bouncy castle provider library from: http://www.bouncycastle.org/latest_releases.html. This will go on your classpath below.

Run a command to extract the certificate from the server and create a pem file. In this case, mycert.pem.

```bash
echo | openssl s_client -connect ${MY_SERVER}:443 2>&1 | \
 sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > mycert.pem
```

Then run the following commands to create the keystore.

```bash
export CLASSPATH=/path/to/bouncycastle/bcprov-jdk15on-155.jar
CERTSTORE=res/raw/mystore.bks
if [ -a $CERTSTORE ]; then
    rm $CERTSTORE || exit 1
fi
keytool \
      -import \
      -v \
      -trustcacerts \
      -alias 0 \
      -file <(openssl x509 -in mycert.pem) \
      -keystore $CERTSTORE \
      -storetype BKS \
      -provider org.bouncycastle.jce.provider.BouncyCastleProvider \
      -providerpath /path/to/bouncycastle/bcprov-jdk15on-155.jar \
      -storepass some-password
```

You'll notice that the above script places the result in `res/raw/mystore.bks`. Now you have a file that you'll load into your Android app that provides the missing certificate(s).

To do this, register your SSLSocketFactory for the SSL scheme:

```java
final SchemeRegistry schemeRegistry = new SchemeRegistry();
schemeRegistry.register(new Scheme("http", PlainSocketFactory.getSocketFactory(), 80));
schemeRegistry.register(new Scheme("https", createAdditionalCertsSSLSocketFactory(), 443));

// and then however you create your connection manager, I use ThreadSafeClientConnManager
final HttpParams params = new BasicHttpParams();
...
final ThreadSafeClientConnManager cm = new ThreadSafeClientConnManager(params,schemeRegistry);
```

To create your SSLSocketFactory:

```java
protected org.apache.http.conn.ssl.SSLSocketFactory createAdditionalCertsSSLSocketFactory() {
    try {
        final KeyStore ks = KeyStore.getInstance("BKS");

        // the bks file we generated above
        final InputStream in = context.getResources().openRawResource( R.raw.mystore);  
        try {
            // don't forget to put the password used above in strings.xml/mystore_password
            ks.load(in, context.getString( R.string.mystore_password ).toCharArray());
        } finally {
            in.close();
        }

        return new AdditionalKeyStoresSSLSocketFactory(ks);

    } catch( Exception e ) {
        throw new RuntimeException(e);
    }
}
```

And finally, the AdditionalKeyStoresSSLSocketFactory code, which accepts your new KeyStore and checks if the built-in KeyStore fails to validate an SSL certificate:

```java
/**
 * Allows you to trust certificates from additional KeyStores in addition to
 * the default KeyStore
 */
public class AdditionalKeyStoresSSLSocketFactory extends SSLSocketFactory {
    protected SSLContext sslContext = SSLContext.getInstance("TLS");

    public AdditionalKeyStoresSSLSocketFactory(KeyStore keyStore) throws NoSuchAlgorithmException, KeyManagementException, KeyStoreException, UnrecoverableKeyException {
        super(null, null, null, null, null, null);
        sslContext.init(null, new TrustManager[]{new AdditionalKeyStoresTrustManager(keyStore)}, null);
    }

    @Override
    public Socket createSocket(Socket socket, String host, int port, boolean autoClose) throws IOException {
        return sslContext.getSocketFactory().createSocket(socket, host, port, autoClose);
    }

    @Override
    public Socket createSocket() throws IOException {
        return sslContext.getSocketFactory().createSocket();
    }

    /**
     * Based on http://download.oracle.com/javase/1.5.0/docs/guide/security/jsse/JSSERefGuide.html#X509TrustManager
     */
    public static class AdditionalKeyStoresTrustManager implements X509TrustManager {

        protected ArrayList<X509TrustManager> x509TrustManagers = new ArrayList<X509TrustManager>();


        protected AdditionalKeyStoresTrustManager(KeyStore... additionalkeyStores) {
            final ArrayList<TrustManagerFactory> factories = new ArrayList<TrustManagerFactory>();

            try {
                // The default Trustmanager with default keystore
                final TrustManagerFactory original = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
                original.init((KeyStore) null);
                factories.add(original);

                for( KeyStore keyStore : additionalkeyStores ) {
                    final TrustManagerFactory additionalCerts = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
                    additionalCerts.init(keyStore);
                    factories.add(additionalCerts);
                }

            } catch (Exception e) {
                throw new RuntimeException(e);
            }

            /*
             * Iterate over the returned trustmanagers, and hold on
             * to any that are X509TrustManagers
             */
            for (TrustManagerFactory tmf : factories)
                for( TrustManager tm : tmf.getTrustManagers() )
                    if (tm instanceof X509TrustManager)
                        x509TrustManagers.add( (X509TrustManager)tm );

            if( x509TrustManagers.size()==0 )
                throw new RuntimeException("Couldn't find any X509TrustManagers");
        }

        /*
         * Delegate to the default trust manager.
         */
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            final X509TrustManager defaultX509TrustManager = x509TrustManagers.get(0);
            defaultX509TrustManager.checkClientTrusted(chain, authType);
        }

        /*
         * Loop over the trustmanagers until we find one that accepts our server
         */
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            for( X509TrustManager tm : x509TrustManagers ) {
                try {
                    tm.checkServerTrusted(chain,authType);
                    return;
                } catch( CertificateException e ) {
                    // ignore
                }
            }
            throw new CertificateException();
        }

        public X509Certificate[] getAcceptedIssuers() {
            final ArrayList<X509Certificate> list = new ArrayList<X509Certificate>();
            for( X509TrustManager tm : x509TrustManagers )
                list.addAll(Arrays.asList(tm.getAcceptedIssuers()));
            return list.toArray(new X509Certificate[list.size()]);
        }
    }
}
```

## 3.How to import an existing X.509 certificate and private key in Java keystore to use in SSL?

Believe or not, keytool does not provide such basic functionality like importing private key to keystore. You can try this [workaround](http://web.archive.org/web/20190426215445/http://cunning.sharp.fm/2008/06/importing_private_keys_into_a.html) with merging PKSC12 file with private key to a keystore:

```java
keytool -importkeystore \
  -deststorepass storepassword \
  -destkeypass keypassword \
  -destkeystore my-keystore.jks \
  -srckeystore cert-and-key.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass p12password \
  -alias 1
```

Or just use more user-friendly [KeyMan](https://www.ibm.com/developerworks/mydeveloperworks/groups/service/html/communityview?communityUuid=6fb00498-f6ea-4f65-bf0c-adc5bd0c5fcc) from IBM for keystore handling instead of keytool.

## 4.android load BKS error: wrong version of key store

```
java.io.IOException: Wrong Version of key store
at com.android.org.bouncycastle.jce.provider.JDKKeyStore.engineLoad(JDKKeyStore.java:812)
at java.security.KeyStore.load(KeyStore.java:589)
```

首先需要下载工具KeyStore Explorer:  

http://keystore-explorer.sourceforge.net/

1.安装后，打开原来的BKS证书，选择Tools->Change Type

2.选择BKS-V1，保存即可使用。

亲测可用，在4.0.3,4.4.4,5.0上均可以正常使用。

## 5.Android SSL BKS证书生成

Android SSL BKS证书生成, 以及PFX与JKS证书的转换

1. 生成服务器jks证书:

   ```bash
   keytool -genkey -alias peer -keystore peer.jks
   ```

2. 导出cert证书:

   ```bash
   keytool -exportcert -alias peer -file peer.cert -keystore peer.jks
   ```

3. 生成Android客户端bks密钥库

   需要用到 bcprov-ext-jdk15on-160b03.jar,
   官网:http://www.bouncycastle.org/latest_releases.html
   https://downloads.bouncycastle.org/betas/

　　将jar包放到 Java\jdk1.8.0_20\jre\lib\ext目录下

　　生成私钥库

```
keytool -importcert -keystore keyStore.bks -file peer.cert -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```


　　生成公钥库

```
keytool -importcert -trustcacerts -keystore trustStore.bks -file peer.cert -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```

4. 查看密钥库详情

```
keytool -list -v -keystore keyStore.bks -storepass 123456 -storetype BKS
```

5. 把Android系统的bks格式证书证书复制到Android项目的asset目录中，参考上篇文章即可实现单向的SSL加密TCP通信。





## 参考

* [Android: Trusting SSL certificates](http://blog.crazybob.org/2010/02/android-trusting-ssl-certificates.html)
* [Trusting all certificates using HttpClient over HTTPS](https://stackoverflow.com/questions/2642777/trusting-all-certificates-using-httpclient-over-https/6378872#6378872)
* [Https Connection Android](https://stackoverflow.com/questions/995514/https-connection-android)
* [The Legion of the Bouncy Castle](http://www.bouncycastle.org/latest_releases.html)--依赖包
* [How to import an existing X.509 certificate and private key in Java keystore to use in SSL?](https://stackoverflow.com/questions/906402/how-to-import-an-existing-x-509-certificate-and-private-key-in-java-keystore-to)
* https://blog.csdn.net/w690333243/article/details/79768845

