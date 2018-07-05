# gRPC
###1、概念

***

gRPC 一开始由 google 开发，是一款语言中立、平台中立、开源的远程过程调用(RPC)系统。

在 gRPC 里*客户端*应用可以像调用本地对象一样直接调用另一台不同的机器上*服务端*应用的方法，使得您能够更容易地创建分布式应用和服务。与许多 RPC 系统类似，gRPC 也是基于以下理念：定义一个*服务*，指定其能够被远程调用的方法（包含参数和返回类型）。在服务端实现这个接口，并运行一个 gRPC 服务器来处理客户端调用。在客户端拥有一个*存根*能够像服务端一样的方法。

### 2、使用技术 protocol buffers

***

gRPC默认使用protocol buffers，这是Google开源的一套成熟的结构数据序列化机制。用proto files 创建gRPC服务，用protocol buffers消息类型来定义方法参数和返回类型。

通常建议在gRPC里使用proto3。因为这样你可以使用gRPC支持全部范围的语言，并且能避免proto2客户端与proto3服务端交互时出现的兼容性问题。

### 3、定义服务

***

使用protocol buffers接口定义语言来定义服务方法，用protocol buffer 来定义参数和返回类型。客户端和服务端均使用服务定义生成的接口代码（以官网例子为demo）。

在官网例子里使用protocol buffers IDL定义。

service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  required string greeting = 1;
}

message HelloResponse {
  required string reply = 1;
}

#### 3.1、gRPC服务方法

gRPC允许你定义四类服务方法，

1）单项RPC：客户端发送一个请求给服务端，从服务端获取一个应答，就行一个普通的函数调用

2）服务端流式RPC：客户端发送一个请求给服务端，可获取一个数据流用来读取一系列消息。客户端从返回的数据流里一直读取直到没有更多消息为止。

3）客户端流式RPC：客户端提供一个数据流写入并发送一系列消息给服务端。一旦客户端完成消息写入。就等待服务端读取这些消息并返回应答。

4）双向流式RPC：两边都可以分别通过一个读写数据流来发送一系列消息。这两个数据流操作是相互独立的，所以客户端和服务端能按其希望的仁义顺序读写。

#### 3.2、响应时间

1）截止时间

gRPC允许客户端在调用一个远程方法前指定一个最后期限值，这个值指定了在客户端可以等待服务端多长时间来应答，超过这个时间值RPC将结束并返回DEADLINE_EXCEEDED错误。在服务端可以查询这个期限值来看是否一个特定的方法已经过期，或者还剩多长时间来完成这个方法。

注意：并不是所有语言都有一个默认的截止时间

2）RPC终止

在gRPC里，客户端和服务端对调用成功的判断是独立的、本地的，他们的结论可能不一致。这意味着，如你有一个RPC在服务端成功结束（“我已经返回了所有应答”），到那时在客户端可能是失败的。也可能在客户端把所有请求发送完前，服务端却判断调用已经完成了。

3）取消RPC

无论客户端还是服务端均可以在任何时间取消一个RPC，一个取消会立即终止RPC这样可以避免更多操作被执行。它不是一个‘撤销’，在取消前已经完成的不会被回滚。当然，通过异步调用的RPC不能被取消，因为直到RPC结束前，程序控制权还没有交还给应用。

### 4、通讯安全

gRPC被设计成可以利用插件的形式支持多种授权机制。gRPC集成SSL／TLS并对服务端授权所使用的SSl／TLS进行改良，对客户端和服务端交换的所有数据进行了加密。对客户端来讲提供了可选的机制供凭证来获得共同的授权。



## android 系统使用gRPC

***

一个简单RPC，客户端使用存根发送请求到服务器并等待响应返回，就像平常的函数调用一样。我们的服务中定义rpc 方法，指定他们的请求和响应类型，gRPC允许你定义4中类型的service方法，这些都在RouteGuide服务中使用。

1、单项：

rpc GetFeature(Point) returns (Feature){}

2、服务端流式RPC：

rpc ListFeatures（Rectangle）returns （stream Feature）{}

3、客户端流式RPC：

rpc

 
项目实战：
第一步：根build.gradle 增加
dependencies {

  classpath "com.google.protobuf:protobuf-gradle-plugin:0.8.5"
    
 }
 
第二步：项目build.gradle 增加
apply plugin: 'com.google.protobuf'

protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.0.0'
    }
    plugins {
        javalite {
            artifact = "com.google.protobuf:protoc-gen-javalite:3.0.0"
        }
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.12.0' // CURRENT_GRPC_VERSION
        }
    }
    generateProtoTasks {
        all().each { task ->
            task.plugins {
                javalite {}
                grpc {
                    // Options added to --grpc_out
                    option 'lite'
                }
            }
        }
    }
}


dependencies {
    compile 'io.grpc:grpc-okhttp:1.12.0'
    compile 'io.grpc:grpc-protobuf-lite:1.12.0'
    compile 'io.grpc:grpc-stub:1.12.0'
    compile 'org.conscrypt:conscrypt-android:1.1.2'
    compile 'javax.annotation:javax.annotation-api:1.2'
}

第三步：Activity中增加
/**
     * 首页数据请求 认证
     */
    private class GrpcTask extends AsyncTask<Void, Void, String> {
        private String mHost;
        private String mName;
        private int mPort;
        private ManagedChannel mChannel;

        public GrpcTask(String host, int port, String name) {
            this.mHost = host;
            this.mName = name;
            this.mPort = port;
        }

        @Override
        protected void onPreExecute() {

        }

        @Override
        protected String doInBackground(Void... nothing) {
            try {
                Security.insertProviderAt(Conscrypt.newProvider(), 1);
                /**
                 * okhttp 通讯加密
                 */
                OkHttpChannelBuilder okHttpChannelBuilder = OkHttpChannelBuilder.forAddress(mHost, mPort);
                if(isDebug) {
                    okHttpChannelBuilder.hostnameVerifier(new HostnameVerifier() {
                        @Override
                        public boolean verify(String hostname, SSLSession session) {
                            return true;
                        }
                    });
                    final TrustManager[] trustAllCerts = new TrustManager[] {
                            new X509TrustManager() {
                                @Override
                                public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                                }

                                @Override
                                public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                                }

                                @Override
                                public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                                    return new java.security.cert.X509Certificate[]{};
                                }
                            }
                    };


                    final SSLContext sslContext = SSLContext.getInstance("SSL");
                    sslContext.init(null, trustAllCerts, new java.security.SecureRandom());
                    final SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
                    okHttpChannelBuilder.sslSocketFactory(sslSocketFactory);

                }else{
                    final SSLContext sslContext = SSLContext.getInstance("SSL");
                    sslContext.init(null, null, new java.security.SecureRandom());
                    SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
                    okHttpChannelBuilder.sslSocketFactory(sslSocketFactory);
                }

                mChannel=okHttpChannelBuilder.build();
                /**
                 * 新闻资讯
                 */
                NewsServGrpc.NewsServBlockingStub news_stub = NewsServGrpc.newBlockingStub(mChannel).withCallCredentials(new     PtokenCallCredentials(getActivity()));
                GetNewsRequest  mGetNewsRequest = GetNewsRequest.newBuilder().setWhichColumn(newsType)
                                                                             .setPagingCond(PagingCondition.newBuilder().setCurrentPage(pageIndex)
                                                                                                                        .setSizePerPage(pageSize).build())
                                                                             .build();
                //数据返回
                GetNewsResponse newsList = news_stub.getNews(mGetNewsRequest);

                Map<String,NewsList>  mMap = newsList.getNewsMapMap();

                NewsList mNewsList = mMap.get("cpdt");
                PagingResult mPagingResult = mNewsList.getPagingResult();
                //总数量
                int  sizeTotal = mPagingResult.getSizeTotal();
                //每页数据量大小
                int  sizePerPage = mPagingResult.getSizePerPage();
                //当前页码
                int  currentPage = mPagingResult.getCurrentPage();
                //新闻数量
                int newsCount = mNewsList.getNewsListCount();


                /**
                 * 数据库 清理
                 */
                SQLiteDatabase dba_top = SQLiteDatabase.openDatabase(
                        "/data/data/cn.com.topsec.www.topseccloud/topsec_cmp",
                        null, SQLiteDatabase.NO_LOCALIZED_COLLATORS);
                Cursor cursor_top = dba_top.query(sqlite_table_name, null,null
                        , null, null, null, null);

                if(cursor_top.moveToNext()){//有数据清空表数据
                    dba_top.execSQL("DELETE FROM "+sqlite_table_name);
                }else{//无数据增加数据

                }
                //遍历所有 数据
                ContentValues cv = new ContentValues();
                if(newsCount>0){

                    for(int i =0 ;i<newsCount;i++){
                        String id = mNewsList.getNewsList(i).getId();
                        String title = mNewsList.getNewsList(i).getTitle();
                        String author = mNewsList.getNewsList(i).getAuthor();
                        String pubDate = mNewsList.getNewsList(i).getPubDate();
                        String htmlUrl = mNewsList.getNewsList(i).getHtmlUrl();
                        String imgUrl = mNewsList.getNewsList(i).getImgUrl();
                        String column = mNewsList.getNewsList(i).getColumn();



                        cv.put("id",id);
                        cv.put("title",title);
                        cv.put("author",author);
                        cv.put("sortdate",pubDate);
                        cv.put("path",htmlUrl);
                        cv.put("toplevel",1);
                        cv.put("typepath",imgUrl);
                        cv.put("uuid",1);
                        cv.put("typeimg",column);
                        cv.put("version",1);
                        cv.put("shoucang",0);
                        cv.put("readtype",0);

                        dba_top.insert(sqlite_table_name,null,cv);
                    }

                }

                cursor_top.close();
                dba_top.close();

                //登录失败
                Message message = Message.obtain(mHandler);
                message.what = 1;
                mHandler.sendMessageDelayed(message, 1000); //通过延迟发送消息，每隔一秒发送一条消息



                return "yes";

            } catch (Exception e) {
                e.printStackTrace();
                StringWriter sw = new StringWriter();
                PrintWriter pw = new PrintWriter(sw);
                e.printStackTrace(pw);
                pw.flush();
                /** 异常截获 **/
                errorMessage = e.getMessage();
                //登录失败
                Message message = Message.obtain(mHandler);
                message.what = 2;
                mHandler.sendMessageDelayed(message, 1000); //通过延迟发送消息，每隔一秒发送一条消息
                return "Login... : " + System.lineSeparator() + sw;
            }
        }

        @Override
        protected void onPostExecute(String result) {

            try {
                mChannel.shutdown().awaitTermination(1, TimeUnit.SECONDS);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }

            Log.d(TAG, result);

        }
    }
























