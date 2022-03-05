<h1 align="center">okhttp配置https证书</h1>

[toc]

## android okhttp3 配置https证书

一、写在前面，客户端的证书，一般是由服务端提供的，我们来认识一下：

ca.crt       ：服务端证书

client.crt   ：客户端证书

client.key   ：客户端证书秘钥

ca.crt就是我们客户端单向验证时使用的证书， 那么client.crt和client.key就应该是双向验证用到的bks了，于是重点就是他们间的转换了

准备工作，我们用到两个工具：

openssl：证书格式转换及秘钥获取

下载地址：https://download.csdn.net/download/hhbbeijing/13454495

keytool：

> 介绍：OpenSSL整个软件包大概可以分成三个主要的功能部分：SSL协议库、应用程序以及密码算法库。OpenSSL的目录结构自然也是围绕这三个功能部分进行规划的。作为一个基于密码学的安全开发包，OpenSSL提供的功能相当强大和全面，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。

 

二、正文

先看一段okhttps创建的代码：

```java
OkHttpClient.Builder builder = new OkHttpClient().newBuilder()
        .connectTimeout(15, TimeUnit.SECONDS)
        .readTimeout(15,TimeUnit.SECONDS)
        .addNetworkInterceptor(logInterceptor)
        .hostnameVerifier(new MYHostnameVerifier(HOSTNAME));
if(trustManager != null){
    //SSLSocketFactory X509TrustManager
    builder.sslSocketFactory(getSSLSocketFactory(trustManager),trustManager);
}
mOkHttpClient = builder.build();
```


需要两个参数：

SSLSocketFactory：（2）：   BKS_PWD  JKSNAME

trustManager：（1）  其中CERTNAME是信任的服务端证书，但格式为cer，由ca.crt格式转换而来。（可以直接修改后缀）


（1）：

```java
private X509TrustManager getTrustManager(){
    InputStream certificate = null;
    X509TrustManager trustManager = null;
    try {
        certificate = mContext.getAssets().open(CERTNAME);
        trustManager = trustManagerForCertificates(certificate);
    } catch (Exception e) {
        //e.printStackTrace();
    }
    return trustManager;
}
```

（2）：

```java
private SSLSocketFactory getSSLSocketFactory(X509TrustManager trustManager) throws NoSuchAlgorithmException, KeyManagementException {
    SSLContext context = SSLContext.getInstance("TLS");
    TrustManager[] trustManagers = {trustManager};
    KeyManager[] keyManagers = getKeyManager();
    context.init(keyManagers, trustManagers, new SecureRandom());
    return context.getSocketFactory();
}

private KeyManager[] getKeyManager() {
    try {
        String pwd = BKS_PWD;
        InputStream bks = mContext.getAssets().open(JKSNAME);
        KeyStore clientKeyStore = KeyStore.getInstance("BKS");
        clientKeyStore.load(bks, pwd.toCharArray());
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        keyManagerFactory.init(clientKeyStore, pwd.toCharArray());
        return keyManagerFactory.getKeyManagers();
    } catch (Exception e) {
        //e.printStackTrace();
    }
    return null;
}
```

重点来了，如何

1、android平台能识别的客户端证书 为：.bks格式的，需要格式转换，分两步：

    第一步，将 client.crt 和 client.key转换成 client.p12

   openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12 -name tomcat -CAfile ca.crt -caname root -chain

  第二步，把.p12格式转换成.bks

  使用 【portecle】工具转换为 .bks （JKSNAME），记住输入的密码 （BKS_PWD）。

2、由ca.crt格式转换而来。（可以直接修改后缀）或者  openssl x509 -in ca.crt -out server.cer -outform der

## [如何使用key和crt文件转换为tomcat用到的jks格式证书？](https://www.oschina.net/question/2266279_221175)

第一步，从key和crt生成pkcs12格式的keystore

```bash
openssl pkcs12 -export -in mycert.crt -inkey mykey.key 
                        -out mycert.p12 -name tomcat -CAfile myCA.crt 
                        -caname root -chain
```

第二步 生成tomcat需要的keystore

```bash
keytool -importkeystore -v -srckeystore mycert.p12 -srcstoretype pkcs12 -srcstorepass 123456 -destkeystore tomcat.keystore -deststoretype jks -deststorepass 123456 
```

## Android平台实现SSL单双向验证

环境：服务器：apache服务器，openssl。

           客户端：PC、java平台、android平台。

思路：

１、先搞定ssl单向验证，再解决双向。

２、先PC，再java平台，再android，不一定非得这样，自由选择，个人是为了弄清整个流程，多走了些路。

过程步骤：

１、在pc上用apache搭建了一个http服务器，用openssl建立自签名的CA证书ca.crt，签发服务器证书server.crt，签发客户端证书client.crt。（apache+openssl配置ssl通信网上资料很多）

２、安装ca.crt，配置服务器，开启单向验证，用浏览器测试验证单向ssl通信。

３、将client.crt和client.key打包生成pkcs12格式的client.pfx文件。

４、配置服务器，开启双向验证，通过浏览器导入client.pfx文件，测试验证双向ssl通信。


重点：

Java平台默认识别jks格式的证书文件，但是android平台只识别bks格式的证书文件，需要在java中配置BC库，个人推荐参考：http://hi.baidu.com/yaming/item/980f253e17f585be124b142d，配置好BC库，看看有没有keytool工具，没有自己弄个Keytool工具

代码参考：http://momoch1314.iteye.com/blog/540613，由于服务端有apache，上面的代码就不用了，此处列出客户端，此文中是通过socket通信的，建议改成https通信，效果会更好，因为和apache服务器打交道，处理起来会更方便。（里面有些东西需要微调的，希望各位自己改一下，对于某些人来说，可能会有些坑，有疑问，请留言，本屌尽力解答）

```java
public class MySSLSocket extends Activity {  
    private static final int SERVER_PORT = 50030;//端口号  
    private static final String SERVER_IP = "218.206.176.146";//连接IP  
    private static final String CLIENT_KET_PASSWORD = "123456";//私钥密码  
    private static final String CLIENT_TRUST_PASSWORD = "123456";//信任证书密码  
    private static final String CLIENT_AGREEMENT = "TLS";//使用协议  
    private static final String CLIENT_KEY_MANAGER = "X509";//密钥管理器  
    private static final String CLIENT_TRUST_MANAGER = "X509";//  
    private static final String CLIENT_KEY_KEYSTORE = "BKS";//密库，这里用的是BouncyCastle密库  
    private static final String CLIENT_TRUST_KEYSTORE = "BKS";//  
    private static final String ENCONDING = "utf-8";//字符集  
    private SSLSocket Client_sslSocket;  
    private Log tag;  
    private TextView tv;  
    private Button btn;  
    private Button btn2;  
    private Button btn3;  
    private EditText et;  
      
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        tv = (TextView) findViewById(R.id.TextView01);  
        et = (EditText) findViewById(R.id.EditText01);  
        btn = (Button) findViewById(R.id.Button01);  
        btn2 = (Button) findViewById(R.id.Button02);  
        btn3 = (Button) findViewById(R.id.Button03);  
          
        btn.setOnClickListener(new Button.OnClickListener(){  
            @Override  
            public void onClick(View arg0) {  
                if(null != Client_sslSocket){  
                    getOut(Client_sslSocket, et.getText().toString());  
                    getIn(Client_sslSocket);  
                    et.setText("");  
                }  
            }  
        });  
        btn2.setOnClickListener(new Button.OnClickListener(){  
            @Override  
            public void onClick(View arg0) {  
                try {  
                    Client_sslSocket.close();  
                    Client_sslSocket = null;  
                } catch (IOException e) {  
                    e.printStackTrace();  
                }  
            }  
        });  
        btn3.setOnClickListener(new View.OnClickListener(){  
            @Override  
            public void onClick(View arg0) {  
                init();  
                getIn(Client_sslSocket);  
            }  
        });  
    }  
      
    public void init() {  
        try {  
            //取得SSL的SSLContext实例  
            SSLContext sslContext = SSLContext.getInstance(CLIENT_AGREEMENT);  
            //取得KeyManagerFactory和TrustManagerFactory的X509密钥管理器实例  
            KeyManagerFactory keyManager = KeyManagerFactory.getInstance(CLIENT_KEY_MANAGER);  
            TrustManagerFactory trustManager = TrustManagerFactory.getInstance(CLIENT_TRUST_MANAGER);  
            //取得BKS密库实例  
            KeyStore kks= KeyStore.getInstance(CLIENT_KEY_KEYSTORE);  
            KeyStore tks = KeyStore.getInstance(CLIENT_TRUST_KEYSTORE);  
            //加客户端载证书和私钥,通过读取资源文件的方式读取密钥和信任证书  
            kks.load(getBaseContext()  
                    .getResources()  
                    .openRawResource(R.raw.kclient),CLIENT_KET_PASSWORD.toCharArray());  
            tks.load(getBaseContext()  
                    .getResources()  
                    .openRawResource(R.raw.lt_client),CLIENT_TRUST_PASSWORD.toCharArray());  
            //初始化密钥管理器  
            keyManager.init(kks,CLIENT_KET_PASSWORD.toCharArray());  
            trustManager.init(tks);  
            //初始化SSLContext  
            sslContext.init(keyManager.getKeyManagers(),trustManager.getTrustManagers(),null);  
            //生成SSLSocket  
            Client_sslSocket = (SSLSocket) sslContext.getSocketFactory().createSocket(SERVER_IP,SERVER_PORT);  
        } catch (Exception e) {  
            tag.e("MySSLSocket",e.getMessage());  
        }  
    }  
          
    public void getOut(SSLSocket socket,String message){  
        PrintWriter out;  
        try {  
            out = new PrintWriter(  
                    new BufferedWriter(  
                            new OutputStreamWriter(  
                                    socket.getOutputStream()  
                                    )  
                            ),true);  
            out.println(message);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
      
    public void getIn(SSLSocket socket){  
        BufferedReader in = null;  
        String str = null;  
        try {  
            in = new BufferedReader(  
                    new InputStreamReader(  
                            socket.getInputStream()));  
            str = new String(in.readLine().getBytes(),ENCONDING);  
        } catch (UnsupportedEncodingException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        new AlertDialog  
        .Builder(MySSLSocket.this)  
        .setTitle("服务器消息")  
        .setNegativeButton("确定", null)  
        .setIcon(android.R.drawable.ic_menu_agenda)  
        .setMessage(str)  
        .show();  
    }  
} 
```

单向：

１、用keytool将ca.crt导入到bks格式的证书库ca.bks，用于验证服务器的证书，命令如下：

keytool -import -alias ca -file ca.crt -keystore ca.bks -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider

２、服务器配置成单向验证，将ca.bks放到android工程的assets或raw下，对应的读取就是代码中的

```
kks.load(getBaseContext()  
                    .getResources()  
                    .openRawResource(R.drawable.kclient),CLIENT_KET_PASSWORD.toCharArray());
```


不一定是R.drawable.kclient，自己根据实际做修改，读取文件，不懂网上查，不啰嗦了。

至此，单向ssl通信应该是OK了。

(PS:  针对２中的操作不一定非得这么做，也可以把ca.bks导入到android平台下的cacerts.bks文件中，然后从这个文件读取认证，怎么导入，网上资料很多，如：http://blog.csdn.net/haijun286972766/article/details/6247675


调试中遇到的问题，提一下：

一般在模拟器中能通过，在真实平台上就没问题了。

这里需要注意的是证书的有效期，一定要在证书的有效期内操作

双向：

双向在单向的基础上实现，不过要先生成android平台能识别的客户端证书，这个玩意也伤脑筋，网上提到生成bks格式客户端证书的资料很少，鲜有借鉴之用。

在这个点上，太伤脑筋了，估计很多伙计也在这儿卡得蛋疼，一开始是毫无头绪，在PC、JAVA平台上生成客户端证书，都能测通，但是转到android平台就傻眼了，用keytool将其它工具生成的crt证书，导成bks格式，不通；用keytool工具新生成bks格式证书，也不通；

各种能想的方法试尽，一度怀疑自己是不是哪个细节出错了，理论上肯定能做的东西，怎么看不到一点可实现性，找资料连续几天，一点进展都没。

后面看国外的资料上提到先用openssl生成pkcs12的.pfx格式证书，然后用工具portecle转换成BKS格式，在android平台上使用，一开始是直接强制性转换，出错，怎么转都转不成功，但是转换成jks格式又没问题，只能根据提示错误，找解决方案，试了好多还是不行，又迷茫了；

１、最后看到国外的资料上的一句话，顿悟灵光，用portecle工具，先建立一个bks格式的keystore，然后将client.pfx中的key pair导入（import key pair），再保存bks文件，测试成功，事实证明：二了一点。

PS:用portecle直接转应该是可以的，只是我一直没转成功过，可能是我的java环境有问题，老提示illegal key size。

２、将服务器配置成双向验证，将ca.bks放到android工程的assets或raw下，对应的读取就是代码中的

```
tks.load(getBaseContext()  
   .getResources()  
   .openRawResource(R.drawable.lt_client),CLIENT_TRUST_PASSWORD.toCharArray());
```



## 参考文献

* [android okhttp3 配置https证书](https://blog.csdn.net/hhbbeijing/article/details/110677525)
* [如何用第三方开源免费软件portecle从https网站上导出SSL的CA证书?](https://blog.csdn.net/chancein007/article/details/34514439)
* [Https双向验证证书：Android+OpenSSL](https://blog.csdn.net/liudehuaii18/article/details/50373652)
* [Android平台实现SSL单双向验证](https://blog.csdn.net/love_xsq/article/details/43735305)
* [java支持https_ssl双向证书访问](https://my.oschina.net/u/437309/blog/4414762)

