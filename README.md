### 1.Flutter屏幕算法面试题基础(一)
假如蓝湖设计图给你一张轮播图，宽度是 x 高度是 y（x px * y px），左右间隔是t，如何使用屏幕算法适配全机型屏幕宽和高？其实这种形式是可以直接用AspectRatio写宽高比

答案：计算方式一
宽度：整宽 - t * 2（左右的）。
高度：(整宽 - t * 2 ) * y / x。

答案：计算方式二
flutter_screenutil 插件
ScreenUtilInit(
designSize: const Size(375, 820),
builder: (context, child) {
return ();
},
)

### 2.Flutter项目框架基础面试题(二)
开发亿荐项目使用的框架和插件

getX: ：高性能的状态管理、智能的依赖注入和便捷的路由管理
创建Page使用Obx组件或GetX或者GetBuilder进行页面数据渲染处理,(可以对页面进行手动刷新渲染)
创建Bindings对GetxController进行绑定和初始化
创建State对数据进行保存和共享处理
创建GetxController对视图UI层的数据和事件处理分离；

dio: 网络请求库
创建ConnectivityInterceptor进行网络异常处理
创建ErrorInterceptor进行日志处理
创建TokenInterceptor 进行全局Token异常处理
创建SparrowDioConfig 设置全局token,处理输出错误函数,存储设备信息参数
创建SparrowDio类,创建get 、post、put、delete、download网络请求方式管理
创建HttpUrls管理配置网络请求地址
创建ResultData来对请求数据进行初步处理

cached_network_image:图片缓存
connectivity_plus:检查手机联网情况
flutter_screenutil:屏幕适配
flutter_easyrefresh:上下拉刷新组件
flutter_native_image: flutter_image_compress :图片压缩工具
photo_view:图片浏览
event_bus:消息通知
tim_ui_kit_calling_plugin : tim_ui_kit :腾讯IM插件
tencent_trtc_cloud:腾讯直播插件
customRouteAnimatio:自定义路由动画
signUtil: 对接口请求进行加签名处理

** 3 isolate是怎么进行通信和实例化的？**
isolate线程之间的通讯主要是通过port来进行的,这个port消息传递是异步的,通过Dart源码看到,实例化一个一个isolate的过程包括：实例化isolate结构体；在堆中分配线程内存；配置port等过程

loadData() async {
// 通过spawn新建一个isolate，并绑定静态方法
ReceivePort receivePort = ReceivePort();
await Isolate.spawn(dataLoader, receivePort.sendPort);

// 获取新的isolate监听port
SendPort sendPort = await receivePort.first;
//调用sendReceive自定义方法
List dataList = await sendReceive(sendPort,
'http://www.flutterj.com');
print('dataList $dataList');
}

// isolate绑定方法
static dataLoader(SendPort sendPort) async {
// 创建监听port，并将sendPort传给外界来调用
ReceivePort receivePort = ReceivePort();
sendPort.send(receivePort.sendPort);
// 监听外界调用
await for (var msg in receivePort) {
String requestURL = msg[0];
SendPort callbackPort = msg[1];

Client client = Client();
Response response = await client.get(requestURL);
List dataList = json.decode(response.body);
// 回调返回值给调用者
callbackPort.send(dataList);
}
}

// 创建自己的监听port，并且向新的isolate发送消息
Future sendReceive(SendPort sendPort, String url) {
ReceivePort receivePort = ReceivePort();
sendPort.send([url, receivePort.sendPort]);
// 接收到返回值， 返回给调用者
return receivePort.first;
}

4.Flutter是怎么实现热重载的，原理和过程是怎么样的？
Flutter热重载是基于State的，也就是我们在代码中经常出现的setState方法，通过这个来修改后，会执行相应的build方法，这就是热重载的基本过程。

5.为什么说Flutter性能好？说下和其他跨平台的本质区别！
flutter只关心向CPU提供视图数据,GPU的 VSync信号同步到 UI线程,UI线程使用 Dart来构建抽象的视图结构，这份数据结构在 GPU线程进行图层合成，视图数据提供给 Skia引擎渲染为 GPU数据，这些数据通过 OpenGL或者 Vulkan提供给 GPU。

6.说下Widget 和 element 和 RenderObject 之间的关系？
widget是用户界面的一部分,并且是不可变的
element是渲染树中特定位置的Widget的实例。
RenderObject是渲染树中的一个对象，它的层次结构是渲染库的核心

Widget会被inflate（填充）到Element，并由Element管理底层渲染树。Widget并不会直接管理状态及渲染,而是通过State这个对象来管理状态。Flutter创建Element的可见树，相对于Widget来说，是可变的，通常界面开发中，我们不用直接操作Element,而是由框架层实现内部逻辑。就如一个UI视图树中，可能包含有多个TextWidget(Widget被使用多次)，但是放在内部视图树的视角，这些TextWidget都是填充到一个个独立的Element中。Element会持有renderObject和widget的实例。记住，Widget 只是一个配置，RenderObject 负责管理布局、绘制等操作。
在第一次创建 Widget 的时候，会对应创建一个 Element， 然后将该元素插入树中。如果之后 Widget 发生了变化，则将其与旧的 Widget 进行比较，并且相应地更新 Element。重要的是，Element 不会被重建，只是更新而已。

7. Flutter 如何与 Android iOS 通信？
Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种
BasicMesageChannel；用于传递字符串和半结构化的信息
MethodChannel;用于传递方法调用,flutter主动调用native的方法,并获得相应的返回值
EventChannel;用于数据流event streams）的通信。

8.什么是widget? 在flutter里有几种类型的widget？分别有什么区别？能分别说一下生命周期吗？
widget在flutter里面是基础UI组件,分为statefulWidget和statelessWidget两种,statelessWidget不会自己重新构建自己，但是statefulWidget会
statelessWidget的生命周期，生命周期只有build过程。build是用来创建Widget的，在每次页面刷新时会调用build。

StatefulWidget的生命周期依次为：

createElement创建元素树,子类一般不会重写此方法调用

createState,是StatefulWidget创建State的方法，只调用一次

initState,是StatefulWidget创建后调用的第一个方法,只执行一次,这时View没有渲染,但StatefulWidget 已经被加载到渲染树里了,一般在这里进行初始化操作

didChangeDependencies,是会在initState后立即调用，当StatefulWidget依赖的InheritedWidget发生变化之后，didChangeDependencies才会调用，所以didChangeDependencies可以调用多次。(备注：widget树中，若节点的父级结构中的层级 或 父级结构中的任一节点的widget类型有变化，节点会调用didChangeDependencies；若仅仅是父级结构某一节点的widget的某些属性值变化，节点不会调用didChangeDependencies)

build方法会在didChangeDeoendencies之后立即调用,在之后setState()刷新时,每次会重新调用build绘制
addPostFrameCallback是StatefulWidget渲染结束之后的回调,只会调用一次,一般是在initState里添加回调：

didUpdateWidget，状态发生改变的时候调用备注:（widget树中，若节点调用setState方法，节点本身不会触发didUpdateWidget，此节点的子节点 会 调用didUpdateWidget）

deactivate页面切换时候会触发调用

dispose(组件移除时)一般在dispose中做一些取消监听、动画的操作，和initState相对使用

9.说说APP的生命周期函数，AppLifecycleState
AppLifecycleState.paused,应用进入后台 paused
AppLifecycleState.resumed,应用进入前台 resumed活跃中
AppLifecycleState.inactive,应用进入非活动状态 , 如来了个电话 , 电话应用进入前台
AppLifecycleState.detached,应用程序仍然在 Flutter 引擎上运行 , 但是与宿主 View 组件分离

10.什么是flutter里的key? 有什么用？
key是Widgets，Elements和SemanticsNodes的标识符。
key有LocalKey 和 GlobalKey两种。
Globalkey可以主动获取以及主动改变子控件的状态。(备注:GlobalKey允许 Widget 在应用中的任何位置更改父级而不会丢失 State。)
LocalKey 如果要修改集合中的控件的顺序或数量。包含ValueKey ，ObjectKey，UniqueKey

11.介绍下FFlutter的FrameWork层和Engine层，以及它们的作用
Flutter的FrameWork层是用Drat编写的框架（SDK），它实现了一套基础库，包含Material（Android风格UI）和Cupertino（iOS风格）的UI界面，下面是通用的Widgets（组件），之后是一些动画、绘制、渲染、手势库等。这个纯 Dart实现的 SDK被封装为了一个叫作 dart:ui的 Dart库。我们在使用 Flutter写 App的时候，直接导入这个库即可使用组件等功能。

Flutter的Engine层是Skia 2D的绘图引擎库，其前身是一个向量绘图软件，Chrome和 Android均采用 Skia作为绘图引擎。Skia提供了非常友好的 API，并且在图形转换、文字渲染、位图渲染方面都提供了友好、高效的表现。Skia是跨平台的，可以轻松实现对原生界面的绘制,性能媲美原生

12.介绍下Widget、State、Context 概念
Widget：在Flutter中，几乎所有东西都是Widget。将一个Widget想象为一个可视化的组件（或与应用可视化方面交互的组件），当你需要构建与布局直接或间接相关的任何内容时，你正在使用Widget。
Widget树：Widget以树结构进行组织。包含其他Widget的widget被称为父Widget(或widget容器)。包含在父widget中的widget被称为子Widget。
Context：是已创建的所有Widget树结构中的某个Widget的位置引用,将context作为widget树的一部分,其中context所对应的widget被添加到此树中,一个context只从属于一个widget，它和widget一样是链接在一起的，并且会形成一个context树
State：定义了StatefulWidget实例的行为，它包含了用于”交互/干预“Widget信息的行为和布局。应用于State的任何更改都会强制重建Widget。

13.Dart中var与dynamic的区别
使用var来声明变量，dart会在编译阶段自动推导出类型。而dynamic不在编译期间做类型检查而是在运行期间做类型校验。

14.const和final的区别
const 的值在编译期确定，final 的值在运⾏时确定。

15.art的一些重要概念？
在Dart中，一切都是对象，所有的对象都是继承自Object
Dart是强类型语言，但可以用var或 dynamic来声明一个变量，Dart会自动推断其数据类型,dynamic类似c#
没有赋初值的变量都会有默认值null
Dart支持顶层方法，如main方法，可以在方法内部创建方法
Dart支持顶层变量，也支持类变量或对象变量
Dart没有public protected private等关键字，如果某个变量以下划线（_）开头，代表这个变量在库中是私有的

16.说说mixin extends implement 之间的关系?
继承（关键字 extends）、混入 mixins （关键字 with）、接口实现（关键字 implements）
这三者可以同时存在，前后顺序是extends -> mixins -> implements。
Flutter中的继承是单继承，子类重写超类的方法要用@OverRide，子类调用超类的方法要用super。

17..Navigator是什么，为什么Navigator可以实现无需上下文路由导航？
Navigator是在Flutter中负责管理维护页面堆栈的导航器。MaterialApp在需要的时候，会自动为我们创建Navigator。Navigator.of(context)，会使用context来向上遍历Element树，找到MaterialApp提供的NavigatorState再调用其push/pop方法完成导航操作。
所以如果在MaterialApp的navigatorKey属性内设置好一个Key就可以直接使用这个Key来进行路由导航，无需上下文
