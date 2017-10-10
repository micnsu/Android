### 	网络请求框架（Rxjava+retrofit2.0）

#### 环境配置

1. Android studio导入gradle依赖

   ```
   compile 'com.squareup.retrofit2:retrofit:2.1.0'
   compile 'com.squareup.retrofit2:converter-scalars:2.0.0-beta4'
   compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
   compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
   compile 'io.reactivex:rxandroid:1.2.1'
   ```

2. retrofitAPIservice

   ```
   public interface HttpAPI {
       //基础地址
       String BASE_URL = "http://211.149.161.37/";

       /**
        * 提交json数据
        * @param
        * @return
        */
       @Headers({"Content-Type: application/json", "Accept: application/json"})
       @POST("{path}")
       Observable<ResponseBody> questJsonValue(@Path(value = "path",encoded = true) String path, @Body RequestBody requestBody);

       /**
        * 以对象的形式进行返回
        * @param path 请求地址
        * @param params 参数集
        * @return
        */
       @GET("{path}")
       Observable<ResponseBody> getRequest(@Path(value = "path",encoded = true) String path, @QueryMap Map<String, String> params);

       /**
        *以post 表单的形式进行传输
        * @param path
        * @param params
        * @return
        */
       @FormUrlEncoded
       @POST("{path}")
       Observable<ResponseBody> doPost(@Path(value = "path",encoded = true) String path,
                                       @FieldMap Map<String,String> params);

       /**
        *以post 表单的形式进行传输
        * @param path
        * @param
        * @return
        */
       @FormUrlEncoded
       @POST("{path}")
       Observable<ResponseBody> doPost2(@Path(value = "path",encoded = true) String path, @Field("uid")int uid,
                                        @Field("token")String token);
       /**
        *以post 表单的形式进行传输
        * @param path
        * @param params
        * @return
        */
       @FormUrlEncoded
       @POST("{path}")
       Observable<ResponseBody> doPost3(@Path(value = "path",encoded = true) String path, @FieldMap Map<String,String> params);
       /**
        * 上传文件
        * @return
        */
       @POST("{path}")
       Observable<ResponseBody> upLoadFile(@Path(value = "path",encoded = true) String path,@Body RequestBody file);

       @POST("{path}")
       Observable<ResponseBody> doPostNoParams(@Path(value = "path",encoded = true)String path);

   }
   ```

3. 构建基础拦截器

   ```
   public class LogIntercept implements Interceptor {
       private static final String TAG = "Interceptor";

       public LogIntercept() {
           super();
       }

       @Override
       public Response intercept(Chain chain) throws IOException {

           Request request = chain.request();
           long t1 = System.nanoTime();
           Log.i(TAG, String.format("Sending request %s on %s%n%s", request.url(), chain.connection(), request.headers()));
           Response response = chain.proceed(request);
           long t2 = System.nanoTime();
           Log.i(TAG, String.format("Received response for %s in %.1fms%n%s",
                   response.request().url(), (t2 - t1) / 1e6d, response.headers()));
                   //打印请求相差时间
           return response;
       }
   }
   ```

4. 构建HttpRequestClient

   ```
   public class HttpRequestServer {
       private static Context mContext;
       //基础请求地址
       private static String base_url;
       private static OkHttpClient mClient;
       private static HttpAPI mHttpAPI;

       private HttpRequestServer(Context context) {
           this(context, null, null);
       }

       private HttpRequestServer(Context context, String base_url, OkHttpClient client) {
           mClient = client;
           this.mContext = context;
           this.base_url = base_url;

           buildServer();
       }

       private void buildServer() {
           Retrofit retrofit = new Retrofit.Builder()
                   .baseUrl(HttpAPI.BASE_URL)
                   .client(mClient)
                   .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                   .addConverterFactory(GsonConverterFactory.create())
                   .build();
           mHttpAPI = retrofit.create(HttpAPI.class);
       }

       private static class HttpRequestServerHolder {
           public static OkHttpClient client = new OkHttpClient.Builder()
                   .addInterceptor(new LogIntercept())
                   .connectTimeout(5, TimeUnit.SECONDS)
                   .readTimeout(5, TimeUnit.SECONDS)
                   .build();
           public static HttpRequestServer server = new HttpRequestServer(mContext, base_url, client);
       }
    
       public static HttpRequestServer create(Context context, String base_url) {
           Retrofit retrofit = new Retrofit.Builder()
                   .baseUrl(base_url)
                   .client(mClient)
                   .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                   .addConverterFactory(GsonConverterFactory.create())
                   .build();
           mHttpAPI = retrofit.create(HttpAPI.class);
    
           return null;
       }
    
       public static HttpRequestServer create(Context context) {
           if (context == null)
               mContext = context;
    
           return HttpRequestServerHolder.server;
       }
    
       /**
        * 使用json数据进行post请求
        *
        * @param path     基于baseUral 后的地址
        * @param json     json字符串
        * @param callBack Subscriber的回调
        */
       public void doPostWithJson(String path, String json, Subscriber<ResponseBody> callBack) {
           RequestBody body = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"), json);
           mHttpAPI.questJsonValue(path, body)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
    
       }
    
       /**
        * 使用参数的形式进行post提交
        * ep:username=abc&age=14
        * Map<String,String> params params.put("username","abc") params.put("age","14")
        *
        * @param path     基于baseUral 后的地址
        * @param params
        * @param callBack
        */
       public void doPostWithParam(String path, Map params, Subscriber<ResponseBody> callBack) {
           mHttpAPI.doPost(path, params)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
    
       }
       /**
        * 使用参数的形式进行post提交
        * ep:username=abc&age=14
        * Map<String,String> params params.put("username","abc") params.put("age","14")
        *
        * @param path     基于baseUral 后的地址
        * @param
        * @param callBack
        */
       public void doPostWithParamForUser(String path,int uid,String token, Subscriber<ResponseBody> callBack) {
           mHttpAPI.doPost2(path, uid,token)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
    
       }
    
       /**
        * 文件上传
        *
        * @param path      基于baseUral 后的地址
        * @param callBack
        */
       public void uploadFile(String path, RequestBody multipart, Subscriber<ResponseBody> callBack) {
           mHttpAPI.upLoadFile(path,multipart)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
       }
    
       /**
        * 不需要传参的post
        * @param path
        * @param callBack
        */
       public void doPost(String path,Subscriber<ResponseBody> callBack){
           mHttpAPI.doPostNoParams(path)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
       }
    
       public void doGet(String path,Map<String,String> params,Subscriber<ResponseBody> callBack){
           mHttpAPI.getRequest(path,params)
                   .subscribeOn(Schedulers.io())
                   .unsubscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
                   .subscribe(callBack);
       }

   }
   ```

5. 如何调用
   ```
     HttpRequestServer.create(getActivity()).doPost(advertisementURL, new Subscriber<ResponseBody>() {
       @Override
       public void onCompleted() {
         	//请求结束后执行
       }
    
       @Override
       public void onError(Throwable e) {
           //请求或响应失败执行
       }
    
       @Override
       public void onNext(ResponseBody responseBody) {
          //请求成功后的执行，需要判断相应code是否为200
       }
   });
   ```

