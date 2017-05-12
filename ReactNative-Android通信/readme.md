RN与原生之间的通信归纳 
===================
>**方式一：通过Callbacks的方式**
>代码如下所示


```
 public class CommApiModule extends ReactContextBaseJavaModule {

    private Context mContex;

    private String mModuleName = "RestApi";

    static final int CODE_ERROR=-1; //请求失败
    public CommApiModule(ReactApplicationContext reactContext) {
        super(reactContext);
        mContex = reactContext;
    }

    @Override
    public String getName() {
        return mModuleName;
    }


    @ReactMethod
    public void doPost(String endpoint,ReadableMap params,Callback callback)
    {
        PostFormBuilder builder=OkHttpUtils.getInstance().post();
        if (!"".equals(endpoint))
        {
            ReadableMapKeySetIterator iterator=params.keySetIterator();
            while (iterator.hasNextKey())
            {
                String key=iterator.nextKey();
                if ("headers".equals(key))
                {
                    //执行header的数据转化
                    ReadableMapKeySetIterator iteratorheader=params.getMap("headers").keySetIterator();
                    while (iteratorheader.hasNextKey())
                    {
                        String headerKey=iteratorheader.nextKey();
                        builder.addHeader(headerKey,params.getMap("headers").getString(headerKey));
                    }
                }
                else
                {
                    builder.addParams(key,params.getString(key));
                }
            }
            builder.addHeader("User-Agent", CoreApplication.M_USER_AGENT);
            TimeZone tz=TimeZone.getDefault();
            builder.addHeader("X-Timezone",tz.getID()+";offset="+tz.getRawOffset()/1000);
            StyleWeReactHttpCallBack mcallback=new StyleWeReactHttpCallBack();
            callback.invoke(200,responsedata);
<!--             mcallback.mCallbackHandler=callback;
            builder.url(RequestAPI.BASE_URL + endpoint).build().execute(mcallback); -->
        }
        else
        {
            ToastUtil.showToastMessage(getCurrentActivity(),"rest api can't been empty");
        }
    }
｝
```
而Native那边实现是：restApi.js
```   
import { NativeModules } from 'react-native';
module.exports = NativeModules.RestApi;  
```   
使用： CartIndex.js
```   
import RestApi from '../../model/restApi';

      RestApi.doGet("cart"
          , {
            "ke1":"value1",
            "ke2":"value2",
            "headers":[
                    {
                    "key1":"value1"
                }
            ]
          }
          , (code, message) => {
              this.setState({ lodyType: -1, isPosting: false });
              this.isLoading = false;
              if (code != 200) return;
              var res = JSON.parse(message);

              console.log("this is the message="+message);

          }
      );

```   
>**方式二：通过Promises的方式**

Promises是ES6的一个新的特性，在React Native中你会看到Promises的大量使用.原生模块也是支持Promises的，这对喜欢使用Promises的小伙伴则是一个很好的消息。

在原生模块中：
```
public class RNTestModule extends ReactContextBaseJavaModule{
    public RNTestModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    @Override
    public String getName() {
        return "RNTest";
    }
    @ReactMethod
    public void measureLayout(
            int tag,
            int ancestorTag,
            Promise promise) {
        try {
            WritableMap map = Arguments.createMap();
            map.putDouble("relativeX",1);
            map.putDouble("relativeY", 1);
            map.putDouble("width", 2);
            map.putDouble("height",3);

            promise.resolve(map);
        } catch (IllegalViewOperationException e) {
            promise.reject(e);
        }
    }
}
```
> - 上述代码中， measureLayout 方法接收的最后一个为Promise，当相应的处理结果出来之后原生模块通过调用Promise的相应方法来向JS模块传递处理成功，或处理失败的数据。

> - 提示：在原生模块中Promise类型的参数要放在最后一位，这样JS调用的时候才能返回一个Promise。

在JS模块中：

``` async test() {
  try {
    var {
        relativeX,
        relativeY,
        width,
        height,
    } = await RNTest.measureLayout(100, 100);

    console.log(relativeX + ':' + relativeY + ':' + width + ':' + height);  
  } catch (e) {
    console.error(e);
  }
}
``` 
> - 在上述代码中，通过ES7的新特性async/await来修饰了 test 方法，来以同步方式调用原生模块的 measureLayout 方法，如果原生模块处理成功， 那么JS中relativeX,relativeY,width,height会获得相应的值，如果原生模块处理失败，则会抛出异常。

如果，不希望以同步的形式调用，可以这样写：
``` 
test2(){
  RNTest.measureLayout(100,100).then(e=>{
    console.log(e.relativeX + ':' + e.relativeY + ':' + e.width + ':' + e.height);
    this.setState({
      relativeX:e.relativeX,
      relativeY:e.relativeY,
      width:e.width,
      height:e.height,
    })
  }).catch(error=>{
    console.log(error);
  });
}
``` 
> - 如果，不希望以同步的形式调用，可以这样写：

``` test2(){
  RNTest.measureLayout(100,100).then(e=>{
    console.log(e.relativeX + ':' + e.relativeY + ':' + e.width + ':' + e.height);
    this.setState({
      relativeX:e.relativeX,
      relativeY:e.relativeY,
      width:e.width,
      height:e.height,
    })
  }).catch(error=>{
    console.log(error);
  });
}
``` 
以上就是通过Promises的方式向JS传递数据的方式，小伙伴们看懂了吗。上述两种方式，与，都可以向JS模块传递数据，但都是只能传递一次。 如果，你需要多次向JS模块传递数据(如：按键事件)上述方式还是不够好，下面就像大家分享可以多次传递数据的方式。

>**方式三：通过发送事件的方式**

原生模块支持另外一种向JS模块传递数据的方式，通过发送事件的方式。原生模块，可以向JS传递事件而不需要直接的调用，就像Android中的广播，iOS中的通知中心。下面就向大家演示通过 RCTDeviceEventEmitter ，来向JS传递事件。

在原生模块中：
```
@Override
public void onHandleResult(String barcodeData) {
    WritableMap params = Arguments.createMap();
    params.putString("result", barcodeData);
    sendEvent(getReactApplicationContext(), "onScanningResult", params);
}

private void sendEvent(ReactContext reactContext,String eventName, @Nullable WritableMap params) {
    reactContext.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
            .emit(eventName, params);
}
```

上述代码向JS模块发送了一个名为“onScanningResult”的事件，并携带了“params”作为参数。

在JS模块中：

下面是在JS代码中进行监听原生模块发出的名为“onScanningResult”的事件。
```
componentDidMount() {
    //注册扫描监听
    DeviceEventEmitter.addListener('onScanningResult',this.onScanningResult);
}
onScanningResult = (e)=> {
    this.setState({
        scanningResult: e.result,
    });
    // DeviceEventEmitter.removeListener('onScanningResult',this.onScanningResult);//移除扫描监听
}
```

> - 在JS中通过 DeviceEventEmitter 注册监听了名为“onScanningResult”的事件，当原生模块发出名为“onScanningResult”的事件后，绑定在该事件上的 onScanningResult = (e) 会被回调。 然后通过 e.result 就可获得事件所携带的数据。

> - 心得：如果在JS中有多处注册了 onScanningResult 事件，那么当原生模块发出事件后，这几个地方会同时收到该事件。不过大家也可以通过 DeviceEventEmitter.removeListener('onScanningResult',this.onScanningResult) 来移除对名为“onScanningResult”事件的监听。

> - 另外，JS模块也支持通过 Subscribable mixin，也注册监听事件，因为ES6已经不再推荐使用mixin，所以在这里也就不向大家介绍了。

对比
>**三种方式的优缺点**

方式                  |   性能
-------------------|-------------------|    
通过Callbacks的方式| 只能传递一次  传递可控，JS模块调用一次，原生模块传递一次
通过Promises的方式|  只能传递一次  传递可控，JS模块调用一次，原生模块传递一次
通过发送事件的方式|  原生模块主动传递，JS模块被动接收   可多次传递





















