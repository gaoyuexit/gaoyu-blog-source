---
title: 状态管理之Bloc初体验
author: Gao先生
toc: false
top: false
mathjax: false
comments: true
copyright: true
categories: Flutter
tags:
  - Flutter
  - Bloc
date: 2020-04-29 16:58:41
updated: 2020-04-29 16:58:41
---
<meta name="referrer" content="no-referrer" />

# 前言

> Flutter和SwiftUI是声明式和响应式编程在移动端的典范, 使用它们开发复杂业务, 状态管理是绕不过去的话题. 
<!-- more -->
BLoC【Business Logic Component】设计模式是`paolo soares` 和 `cong hui` 在2018年`Google dartconf`上提出的，具体设计思想和于传统开发方式的比较, 可以参考[YouTube演示视频](https://www.youtube.com/watch?v=PLHln7wHgPE)

BloC设计模式是Flutter解决状态管理的一种方案, 今天笔者将使用[bloc](https://github.com/felangel/bloc)(实现该设计模式且比较流行的库)来构建一个工程中完整且常见的页面,  作为Flutter初学者, 希望以此和大家交流, 共同进步,   你可以在[这里](https://github.com/juliver-d/bloc_example)找到该项目的完整代码~

<img src='https://github.com/juliver-d/bloc_example/blob/master/source/bloc_page.gif?raw=true' alt='效果' width='30%'>

# BLoC

为什么选用BLoC ? 

> BLoC将视图层与业务逻辑分离变得容易，从而使代码**简单**，**可测** 并且**可重用**, 更重要的是它以一种数据驱动的规范化流程来编写, 各个层级各司其职, 清晰分明

如果你是Web开发者, 你肯定熟悉React+Redux、如果你是iOS开发者, 你可能会使用过RxSwift+ReactorKit,  如果你熟悉单项数据流开发模式, 那么你对BLoC就非常容易上手, BLoC就是用reactive programming方式构建应用, 一个由流构成的完全异步的世界。从而达到界面与业务分离的逻辑。

<img src='https://github.com/juliver-d/bloc_example/blob/master/source/bloc_architecture_full.png?raw=true' alt='架构'>

通过bloc[官网](https://bloclibrary.dev/#/architecture?id=presentation-layer)介绍的流程图来看, 它将应用分为了三层: 展示层, 逻辑层, 数据层. 而通过流程来看, bloc逻辑层作为三层的中枢, 处理三个层级之间的交流.

并且我们可以根据官方的命名规范来组织各个层级结构: 

- 数据层(Data Layer):  职责是从一个或多个源中检索/处理数据。
  - 使用`DataProvider`来提供原始数据(eg: CRUD等)
  - 使用`Repository`来包装一个或多个`DataProvider`, 然后与bloc进行通讯

- 逻辑层(Business Logic Layer):  职责是接收表示层的事件输入, 请求检索数据, 进而以数据建立表示层的新状态
  - 多个bloc之间也可以进行通讯: 每个bloc都有一个状态流, , 其他bloc可以订阅该流, 以便对自己bloc内部做出新状态的改变

- 展示层(Presentation Layer): 职责是向bloc发送用户事件, 根据一个或多个bloc的状态来进行渲染改变



# 示例

## 需求

在大家了解过[bloc widgets](https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets)的api之后, 那就跟着我一起进行页面的开发吧😸

这里我已经准备好了页面所需的数据来源了:

```json
{
	"code": 0,
	"errMsg": "操作成功",
	"data": {
		"time": {
			"start_time": "21:00",
			"end_time": "22:00"
		},
		"address": "北京",
		"types": [{
			"name": "普通公寓：住宅用地，公寓产权，70年，户型面积小",
			"value": "1",
			"selected": 2
		}, {
			"name": "商务公寓：商业用地，商务公寓产权，40年",
			"value": "2",
			"selected": 1
		}, {
			"name": "酒店式公寓：商业用地，商务公寓或公寓产权，40年，面积较小",
			"value": "3",
			"selected": 2
		}],
		"count": "3"
	}
}
```

简单梳理下页面的需求:

- 根据json数据, 展示页面的上的内容
- 点击提交将修改后的内容提交至服务器, 提交参数为:`start_time`, `end_time`,` address`, `count`和选中的`type`对应的value值
- 并且在点击页面返回的时候, 需要校验当然页面是否有修改过的数据, 并对用户进行弹窗展示

## 数据

定义展示模型: 

```dart
class FormModel {
  ///时间
  Time time;
  ///地点
  String address;
  ///类型
  List<SelectValueItem> types;
  ///次数
  String count;
  
  ///help -
  get timeString => '${time.start}-${time.end}';
  get counString => '${count}次';
  
	///转化为提交模型
  FormSubmit toSubmit() {
    return FormSubmit(
      start: time.start,
      end: time.end,
      address: address,
      type: types.firstWhere((e) => e.selected, orElse: null)?.value ?? '0',
      count: count,
    );
  }
}
```

定义提交模型

```dart
///提交模型
class FormSubmit extends Equatable {
  ///起始时间
  final String start;
  ///结束时间
  final String end;
  ///地址
  final String address;
  ///类型
  final String type;
  ///次数
  final String count;
}
```

模拟网络请求(FormRepository)

```dart
const _delay = Duration(milliseconds: 300);
Future<void> wait() => Future.delayed(_delay);

class FormRepository {
  /// 抓取表单数据
  Future<FormModel> fetchFormData() async {
    await wait(); //模拟延迟

    String jsonString = await rootBundle.loadString("assets/json/form_data.json");
    final jsonResult = json.decode(jsonString);
    FormModel model = FormModel.fromJson(jsonResult['data']);
    return model;
  }
  /// 表单保存
  Future<bool> saveForm(FormSubmit model) async {
    await wait();
    return true;
  }
}
```

## 展示

展示数据的逻辑, 我们定义一个form_bloc处理, 为了编写bloc模板, [Bloc Code Generator](https://plugins.jetbrains.com/plugin/12129-bloc-code-generator)插件可以方便的生成构建Bloc的模版代码, 有兴趣的同学可以尝试使用

定义Event:

```dart
@immutable
abstract class FormEvent {}
//请求数据事件
class FormDataRequestEvent extends FormEvent {}
```

定义State:

```dart
@immutable
abstract class FormDataState {}
//初始状态
class FormInitState extends FormDataState {}
//加载成功状态
class FormLoadedState extends FormDataState {
  final FormModel data;
  FormLoadedState({this.data});
}
//数据加载失败状态
class FormFailureState extends FormDataState {
  final String error;
  FormFailureState({@required this.error});
}
//数据请求中状态
class FormInProgressState extends FormDataState {}
```

定义bloc: 

```dart
class FormBloc extends Bloc<FormEvent, FormDataState> {
  @override
  FormDataState get initialState => FormInitState();

  final FormRepository repo;
  FormBloc({@required this.repo}) : assert(repo != null);

  @override
  Stream<FormDataState> mapEventToState(
      FormEvent event,
      ) async* {
    //请求
    if (event is FormDataRequestEvent) {
      yield FormInProgressState(); //请求中状态..
      try {
        final model = await repo.fetchFormData();
        yield FormLoadedState(data: model);//请求成功状态(data)
      } catch (error) {
        yield FormFailureState(error: error.toString());//请求失败状态(error)
      }
    }
  }
```

展示数据: (在页面build的时候, 就可以直接发送请求事件)

```dart
///发送事件
class FormPage extends StatelessWidget {
  final repo = FormRepository();
  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider<FormBloc>(
          create: (ctx) => FormBloc(repo: repo)..add(FormDataRequestEvent()),//发送请求事件
        ),
      ],
      child: FormContent(),
    );
  }
}

///页面内容展示 (这里使用BlocConsumer, 防止listener,builder嵌套)
body: Container(
  color: Color(0xFFF6FAFF),
  child: BlocConsumer<FormBloc, FormDataState>(
    listener: (context, state) {
      if (state is FormFailureState) {//请求失败->toast提示
        ToastUtil.show(state.error);
      }
    },
    builder: (context, state) {
      if (state is FormInProgressState) {//请求中->展示loading
        return LoadingIndicator(); 
      }
      if (state is FormLoadedState) {//请求成功->根据data构建UI
        var model = state.data;
        return SingleChildScrollView( 
          child: Column(
            children: <Widget>[
              _sectionHeader(),
              ContentItem(title: '选择区间', content: model.timeString, hasArrow: true),
              _divider(),
              ContentItem(title: '选择地址', content: model.address, hasArrow: false,),
              _sectionHeader(),
              TypeItem(items: model.types ?? [], onIndexTap: (index) => _setType(index),),
              _divider(),
              ContentItem(title: '选择次数', content: model.counString, hasArrow: false,),
              _buildSaveButton(),
            ],
          ),
        );
      }
      return Container();
    },
  ),
),
```

## 更改

我们在👆上面的展示的图中可以看到,  选择区间, 更改选择类型, 这些都是需要更改数据源来进行UI更改的,  根据**单项数据流模式**, 数据都应由bloc来进行管理, UI层面只进行响应式的更新

所以, 我们最好不要在UI层面来操作数据源, 而是发送`选择区间`,`选择类型`的事件, 来让bloc进行数据源状态的改变

定义两个Event:

```dart
//更新时间
class FormUpdateTimeEvent extends FormEvent {
  final String start;
  final String end;
  FormUpdateTimeEvent({this.start, this.end});
}

//更新类型
class FormUpdateTypeEvent extends FormEvent {
  final int selectIndex;
  FormUpdateTypeEvent({@required this.selectIndex});
}
```

更改相应的State (更改后, bloc获取到的state中的data为最新的数据源)

```dart
Stream<FormDataState> mapEventToState(
    FormEvent event,
    ) async* {
  //更新类型
  if (event is FormUpdateTypeEvent) {
    if (state is FormLoadedState) {
      var model = (state as FormLoadedState).data;
      model.types.forEach((e) => e.selected = false ); //列表中取消所有选中
      model.types[event.selectIndex].selected = true;  //selectIndex下标数据置为选中
      yield (state as FormLoadedState).copyWith(data: model); //发送新数据源的FormLoadedState状态
    }
  }
  //更新时间
  if (event is FormUpdateTimeEvent) {
    if (state is FormLoadedState) {
      var model = (state as FormLoadedState).data;
      model.time.start = event.start;
      model.time.end = event.end;
      yield (state as FormLoadedState).copyWith(data: model);//发送新数据源的FormLoadedState状态
    }
  }
```

那么我们只需要在页面UI层面, 发送定义的这两个事件即可

```dart
///选择区间
void _setTime(String start, String end) {
  BlocProvider.of<FormBloc>(context).add(FormUpdateTimeEvent(start: start, end: end));
}
///选择类型
void _setType(int index) {
  BlocProvider.of<FormBloc>(context).add(FormUpdateTypeEvent(selectIndex: index));
}
```

总结以上完成的显示和更改, 我们用的一个bloc和对应的多个状态, 其实我们也可以将所有逻辑包装成一个页面的state, 将所有的数据包含在一个state中, 所有的Event对应的State都为同一个, 那么事件的处理一般都为`yeild state.copyWith(..)`   

伪代码如这样:

```dart
enum DataState {
  init,
  loading,
  loaded,
  fail,
}

class FormPageDataState extends FormDataState {
  FormModel data, //数据
  FormSubmit submitData,//提交数据
  DataState dataState,
  ... //等其他数据
  
  FormPageDataState copyWith({
  FormModel data,
  FormSubmit submitData,
  DataState dataState,
	}) {
  return FormLoadedState(
    data: data ?? this.data,
    submitData: submitData ?? this.submitData,
    dataState: dataState ?? this.dataState,
  );
}
```

## 提交

提交这部分逻辑 , 我决定使用另一个bloc来处理逻辑`submit_bloc`, 我们会思考提交的状态有什么: 

默认状态, 提交请求中, 请求成功, 请求失败

咦 ??  那所有的请求几乎都是这样, 那我们不如定义一个普通情况下都通用的状态State: 

```dart
@immutable
abstract class NetworkState {}

class NetworkInitState extends NetworkState {}

class NetworkSuccessState<T> extends NetworkState { //使用泛型定义数据
  final T data;
  NetworkSuccessState({this.data});
}

class NetworkFailureState extends NetworkState {
  final String error;
  NetworkFailureState({@required this.error});
}

class NetworkInProgressState extends NetworkState {}
```

我们来看下提交的Bloc, 是不是简单多了: 

```dart
/// - Event
//提交
class SubmitEvent {
  SubmitEvent({this.submit});
  FormSubmit submit;
}

/// - State -> NetworkState


/// - Bloc
class SubmitBloc extends Bloc<SubmitEvent, NetworkState> {
  @override
  NetworkState get initialState => NetworkInitState();

  final FormRepository repo;
  SubmitBloc({@required this.repo}) : assert(repo != null);

  @override
  Stream<NetworkState> mapEventToState(
      SubmitEvent event,
      ) async* {
    yield NetworkInProgressState();
    try {
      bool result = await repo.saveForm(event.submit);
      yield NetworkSuccessState<bool>(data: result);
    } catch (error) {
      yield NetworkFailureState(error: error.toString());
    }
  }
}
```

## 校验

点击返回按钮的时候, 我们需要判断需要提交的数据是否变化了, 我们在该页面请求成功的State中, 添加一个初始化提交模型

```dart
class FormLoadedState extends FormDataState {
  final FormModel data;
  final FormSubmit initSubmit; //初始化提交模型
  FormLoadedState({this.data, this.initSubmit});

  FormLoadedState copyWith({
    FormModel data,
    FormSubmit initSubmitModel,
  }) {
    return FormLoadedState(
      data: data ?? this.data,
      initSubmit: initSubmitModel ?? this.initSubmit,
    );
  }
}

//请求Event对应的State为: 
if (event is FormDataRequestEvent) {
  yield FormInProgressState();
  try {
    final model = await repo.fetchFormData();
    yield FormLoadedState(data: model, initSubmit: model.toSubmit()); //state中增加一个initSubmit属性
  } catch (error) {
    yield FormFailureState(error: error.toString());
  }
}
```

我们看下UI层的事件处理: 

```dart
Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: () => _exitPage(context),
      child: ...
    )
}
//退出
Future<bool> _exitPage(BuildContext context) async {
  var state = BlocProvider.of<FormBloc>(context).state;
  if (state is FormLoadedState) {
    if (state.data.toSubmit() != state.initSubmit) {
      //数据有变化, 弹窗
      JLDialog.cancelConfirmTextDialog(context, content: '内容已经变更，是否保存？').then((confirm) {
        confirm ? _confirmSave() : Navigator.of(context).pop();
      });
      return false;
    }
  }
  return true;
}
```

[完整示例代码在这里](https://github.com/juliver-d/bloc_example)



# 参考

https://github.com/felangel/bloc

https://github.com/4seer/openflutterecommerceapp