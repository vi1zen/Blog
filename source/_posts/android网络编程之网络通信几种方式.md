---
title: Android网络编程之网络通信的几种方式
date: 2017-3-6 21:31:30
categories:
 - Android
tags: 
 - 网络
---

如今，手机应用渗透到各行各业，数量难以计数，其中大多数应用都会使用到网络，与服务器的交互势不可挡，那么android当中访问网络有哪些方式呢？

<!-- more -->

现总结了六种方式：

> * 针对TCP/IP的Socket、ServerSocket
> * 针对UDP的DatagramSocket、DatagramPackage。这里需要注意的是，考虑到Android设备通常是手持终端,IP都是随着上网进行分配的。不是固定的。因此开发也是有一点与普通互联网应用有所差异的。
> * 针对直接URL的HttpURLConnection。
> * Google集成了Apache HTTP客户端，可使用HTTP进行网络编程。
> * 使用WebService。Android可以通过开源包如jackson去支持Xmlrpc和Jsonrpc,另外也可以用Ksoap2去实现Webservice。
> * 直接使用WebView视图组件显示网页。基于WebView 进行开发，Google已经提供了一个基于chrome-lite的Web浏览器，直接就可以进行上网浏览网页。

1.socket与serverSocket

```java
public class TestNetworkActivity extends Activity implements OnClickListener{
 private Button connectBtn;
 private Button sendBtn;
 private TextView showView;
 private EditText msgText;
 private Socket socket;
 private Handler handler;
 @Override
 protected void onCreate(Bundle savedInstanceState) {

  super.onCreate(savedInstanceState);
  setContentView(R.layout.test_network_main);
  connectBtn = (Button) findViewById(R.id.test_network_main_btn_connect);
  sendBtn = (Button) findViewById(R.id.test_network_main_btn_send);
  showView = (TextView) findViewById(R.id.test_network_main_tv_show);
  msgText = (EditText) findViewById(R.id.test_network_main_et_msg);
  connectBtn.setOnClickListener(this);
  sendBtn.setOnClickListener(this);

  handler = new Handler(){
   @Override
   public void handleMessage(Message msg) {
    super.handleMessage(msg);
    String data = msg.getData().getString("msg");
    showView.setText("来自服务器的消息:"+data);
   }
  };
 }
 @Override
 public void onClick(View v) {
  //连接服务器
  if(v == connectBtn){
   connectServer();
  }
  //发送消息
  if(v == sendBtn){
   String msg = msgText.getText().toString();
   send(msg);
  }
 }
 /** 
  *连接服务器的方法
  */
 public void connectServer(){
  try {
   socket = new Socket("192.168.1.100",4000);
   System.out.println("连接服务器成功");
   recevie();
  } catch (Exception e) {
   System.out.println("连接服务器失败"+e);
   e.printStackTrace();
  } 
 }
 /**
  *发送消息的方法
  */
 public void send(String msg){
  try {
   PrintStream ps = new PrintStream(socket.getOutputStream());
   ps.println(msg);
   ps.flush();
  } catch (IOException e) {

   e.printStackTrace();
  }
 }
 /**
  *读取服务器传回的方法
  */
 public void recevie(){
  new Thread(){
   public void run(){
    while(true){
     try {
      InputStream is = socket.getInputStream();
      BufferedReader br = new BufferedReader(new InputStreamReader(is));
      String str = br.readLine();
      Message message = new Message();
      Bundle bundle = new Bundle();
      bundle.putString("msg", str);
      message.setData(bundle);
      handler.sendMessage(message);

     } catch (IOException e) {

      e.printStackTrace();
     }
    }
   }
  }.start();

 }
}
```
2.URL、URLConnection、httpURLConnection、ApacheHttp、WebView

```java
public class TestURLActivity extends Activity implements OnClickListener {
 private Button connectBtn;
 private Button urlConnectionBtn;
 private Button httpUrlConnectionBtn;
 private Button httpClientBtn;
 private ImageView showImageView;
 private TextView showTextView;
 private WebView webView;
 @Override
 protected void onCreate(Bundle savedInstanceState) {

  super.onCreate(savedInstanceState);
  setContentView(R.layout.test_url_main);
  connectBtn = (Button) findViewById(R.id.test_url_main_btn_connect);
  urlConnectionBtn = (Button) findViewById(R.id.test_url_main_btn_urlconnection);
  httpUrlConnectionBtn = (Button) findViewById(R.id.test_url_main_btn_httpurlconnection);
  httpClientBtn = (Button) findViewById(R.id.test_url_main_btn_httpclient);
  showImageView = (ImageView) findViewById(R.id.test_url_main_iv_show);
  showTextView = (TextView) findViewById(R.id.test_url_main_tv_show);
  webView = (WebView) findViewById(R.id.test_url_main_wv);
  connectBtn.setOnClickListener(this);
  urlConnectionBtn.setOnClickListener(this);
  httpUrlConnectionBtn.setOnClickListener(this);
  httpClientBtn.setOnClickListener(this);
 }
 @Override
 public void onClick(View v) {
  // 直接使用URL对象进行连接
  if (v == connectBtn) {

   try {
    URL url = new URL("http://192.168.1.100:8080/myweb/image.jpg");
    InputStream is = url.openStream();
    Bitmap bitmap = BitmapFactory.decodeStream(is);
    showImageView.setImageBitmap(bitmap);
   } catch (Exception e) {

    e.printStackTrace();
   }
  }
  // 直接使用URLConnection对象进行连接   
  if (v == urlConnectionBtn) {
   try {

    URL url = new URL("http://192.168.1.100:8080/myweb/hello.jsp");
    URLConnection connection = url.openConnection();
    InputStream is = connection.getInputStream();
    byte[] bs = new byte[1024];
    int len = 0;
    StringBuffer sb = new StringBuffer();
    while ((len = is.read(bs)) != -1) {
     String str = new String(bs, 0, len);
     sb.append(str);
    }
    showTextView.setText(sb.toString());
   } catch (Exception e) {

    e.printStackTrace();
   }
  }
  // 直接使用HttpURLConnection对象进行连接
  if (v == httpUrlConnectionBtn) {

   try {
    URL url = new URL(
      "http://192.168.1.100:8080/myweb/hello.jsp?username=abc");
    HttpURLConnection connection = (HttpURLConnection) url
      .openConnection();
    connection.setRequestMethod("GET");
    if (connection.getResponseCode() == HttpURLConnection.HTTP_OK) {
     String message = connection.getResponseMessage();
     showTextView.setText(message);
    }
   } catch (Exception e) {

    e.printStackTrace();
   }
  }
  // 使用ApacheHttp客户端进行连接(重要方法)
  if (v == httpClientBtn) {
   try {

    HttpClient client = new DefaultHttpClient();
    // 如果是Get提交则创建HttpGet对象，否则创建HttpPost对象
    // HttpGet httpGet = new
    // HttpGet("http://192.168.1.100:8080/myweb/hello.jsp?username=abc&pwd=321");
    // post提交的方式
    HttpPost httpPost = new HttpPost(
      "http://192.168.1.100:8080/myweb/hello.jsp");
    // 如果是Post提交可以将参数封装到集合中传递
    List dataList = new ArrayList();
    dataList.add(new BasicNameValuePair("username", "aaaaa"));
    dataList.add(new BasicNameValuePair("pwd", "123"));
    // UrlEncodedFormEntity用于将集合转换为Entity对象
    httpPost.setEntity(new UrlEncodedFormEntity(dataList));
    // 获取相应消息
    HttpResponse httpResponse = client.execute(httpPost);
    // 获取消息内容
    HttpEntity entity = httpResponse.getEntity();
    // 把消息对象直接转换为字符串
    String content = EntityUtils.toString(entity);
                                //showTextView.setText(content);

    //通过webview来解析网页
    webView.loadDataWithBaseURL(null, content, "text/html", "utf-8", null);
    //给点url来进行解析
    //webView.loadUrl(url);
   } catch (ClientProtocolException e) {

    e.printStackTrace();
   } catch (IOException e) {

    e.printStackTrace();
   }
  }
 }
}
```
3.使用webService

```java
public class LoginActivity extends Activity implements OnClickListener{
 private Button loginBtn;
 private static final String SERVICE_URL = "http://192.168.1.100:8080/loginservice/LoginServicePort";
 private static final String NAMESPACE = "http://service.lovo.com/";
 @Override
 protected void onCreate(Bundle savedInstanceState) {

  super.onCreate(savedInstanceState);
  setContentView(R.layout.login_main);
  loginBtn = (Button) findViewById(R.id.login_main_btn_login);
  loginBtn.setOnClickListener(this);
 }
 @Override
 public void onClick(View v) {

  if(v == loginBtn){
   //创建WebService的连接对象
   HttpTransportSE httpSE = new HttpTransportSE(SERVICE_URL);
   //通过SOAP1.1协议对象得到envelop
   SoapSerializationEnvelope envelop = new SoapSerializationEnvelope(SoapEnvelope.VER11);
   //根据命名空间和方法名来创建SOAP对象
   SoapObject soapObject = new SoapObject(NAMESPACE, "validate");
   //向调用方法传递参数
   soapObject.addProperty("arg0", "abc");
   soapObject.addProperty("arg1","123");
   //将SoapObject对象设置为SoapSerializationEnvelope对象的传出SOAP消息
   envelop.bodyOut = soapObject;
   try {
    //开始调用远程的方法
    httpSE.call(null, envelop);
    //得到远程方法返回的SOAP对象
    SoapObject resultObj = (SoapObject) envelop.bodyIn;
    //根据名为return的键来获取里面的值，这个值就是方法的返回值
    String returnStr = resultObj.getProperty("return").toString();
    Toast.makeText(this, "返回值："+returnStr, Toast.LENGTH_LONG).show();
   } catch (IOException e) {

    e.printStackTrace();
   } catch (XmlPullParserException e) {

    e.printStackTrace();
   }
  }
 }
}
```