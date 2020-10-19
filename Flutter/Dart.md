#Dart

[变量](#1)  
[内置类型](#2)  
[函数](#3)  
[运算符](#4)  
[异常](#5)  
[类](#6)  
[异步](#7)      

<h3 id="1"></h3>

##变量
1.可以使用var来定义变量，变量的类型可以通过变量值推断出来

```dart
var name = "hi"; //String类型
var age = 18; //int类型
var high = 1.70; //double类型

```
2.也可以使用特定类型来定义变量

```dart
String name = "bruce"; //String类型
int age = 18; //int类型

```
3.如果变量不限于单个类型，则可以使用dynamic或Object来定义变量

```dart
dynamic value = 18;
value = "bruce";
value = 3.5;

Object val = 18;
val = "bruce";
val = 3.5;

```
4.能够放在变量中的所有内容都是对象，所以如果一个变量没有初始化值，那它的默认值就为null。

```dart
int value1;
bool value2;
var value3;
dynamic value4;
print("value4 = $value4");
//结果都为null
```
5.如果不打算更改变量，可以使用final或者const。一个final变量只能被设置一次，而const变量是编译时常量，定义时必须赋值。
<h3 id="2"></h3>
##内置类型
### numbers
包含int和double两种类型，没有像Java中的float类型，int和double都是num的子类型。
### String
Dart的字符串是一系列UTF-16代码单元。可以使用单引号或双引号.
### List
Dart中数组是List对象。

```dart
List arr = ["Bruce", "Nick", "John"];
print("arr = $arr");
```
###Map

```dart
Map map = {
"name": "Bruce",
"age": 18,
"high": 1.70
};

print("map = $map");
print("map['name'] = ${map['name']}");

var map1 = {
1: "hi",
2: "hello",
3: "yep"
};
print("map1 = $map1");
print("map1[1] = ${map1[1]}");

```

<h3 id="3"></h3>

##函数
###定义方法
和绝大多数编程语言一样，Dart函数通常的定义方式为:

```dart
String getName() {
  return "Bruce";
}

```

如果函数体中只包含一个表达式，则可以使用简写语法

```dart
String getName() => "Bruce";

```

###可选参数
Dart函数可以设置可选参数，可以使用命名参数也可以使用位置参数。
命名参数，定义格式如 {param1, param2, …}

```dart
// 函数定义
void showDesc({var name, var age}) {
  if(name != null) {
    print("name = $name");
  }
  if(age != null) {
    print("age = $age");
  }
}

// 函数调用
showDesc(name: "Bruce");

// 输出结果
name = Bruce
```

位置参数，使用 [] 来标记可选参数。

```dart
// 函数定义
void showDesc(var name, [var age]) {
  print("name = $name");
  
  if(age != null) {
    print("age = $age");
  }
}

// 函数调用
showDesc("Bruce");

// 输出结果
name = Bruce

```

函数的可选参数也可以使用 = 设置默认值.

###函数作为参数

Dart中的函数可以作为另一个函数的参数。

```dart
// 函数定义
void println(String name) {
  print("name = $name");
}

void showDesc(var name, Function log) {
  log(name);
}

// 函数调用
showDesc("Bruce", println);

// 输出结果
name = Bruce
```

###匿名函数

```dart
// 函数定义
void showDesc(var name, Function log) {
  log(name);
}

// 函数调用，匿名函数作为参数
showDesc("Bruce", (name) {
    print("name = $name");
  });

// 输出结果
name = Bruce

```

<h3 id ="4"></h3>

##运算符

### ?.

```dart
var name = p?.name; 
//先判断p是否为null，如果是，则name为null；如果否，则返回p.name值
``` 
### ~/
```dart
// 代码语句
var num = 10;
var result = num ~/ 3; //得出一个小于等于(num/3)的最大整数
print("result = $result");

// 输出结果
result = 3

```
###as用来做类型转化。
###is判断对象类型。
###??的使用
```dart
String name;
String nickName = name ?? "Nick"; //如果name不为null，则nickName值为name的值，否则值为Nick
print("nickName = $nickName");
```
###..的使用
级联操作允许对同一个对象进行一系列操作。返回对象本身，相当于kotlin also。

```dart
// 类定义
class Banana {
  var weight;
  var color;
  Banana(this.weight, this.color);
  
  void showWeight() {
    print("weight = $weight");
  }
  
  void showColor() {
    print("color = $color");
  }
}

// 调用
Banana(20, 'yellow')
    ..showWeight()
    ..showColor();
    
// 输出结果
weight = 20
color = yellow

```
<h3 id="5"></h3>

##异常

```dart
// 定义一个抛出异常的函数
void handleOperator() => throw Exception("this operator exception!");

// 函数调用
try {
  handleOperator();
} on Exception catch(e) {
  print(e);
} finally { // finally语句可选
  print("finally");
}

// 输出结果
Exception: this operator exception!
finally

```

<h3 id="6"></h3>

##类
Dart是一种面向对象的语言，具有类和基于mixin的继承。同Java一样，Dart的所有类也都继承自Object。
###构造函数
Dart的构造函数同普通函数一样，可以定义无参和有参，命名参数和位置参数，可选参数和给可选参数设置默认值等。Dart的构造函数有以下几个特点：

可以定义命名构造函数  
可以在函数体运行之前初始化实例变量  
子类默认只有无参无名称的构造函数  
子类定义构造函数时默认继承父类无参构造函数，也可继承指定有参数的构造函数；

```dart
// 类定义
class Tree {
  var desc;
  
  // 命名构造函数
  Tree.init() {
    desc = "this is a seed";
  }
  
  // 函数体运行之前初始化实例变量
  Tree(var des) : desc = des;
}

// 构造函数调用
Tree t = Tree.init();
print("${t.desc}");

Tree t1 = Tree("this is a tree");
print("${t1.desc}");

// 输出结果
this is a seed
this is a tree

```
###mixin继承
```dart
// 类定义
class LogUtil {
  void log() {
    print("this is a log");
  }
}

class Fruit {
  Fruit() {
    print("this is Fruit constructor with no param");
  }
}

class Apple extends Fruit with LogUtil {
  Apple():super() {
    print("this is Apple constructor with no param");
  }
}

// 调用
Apple a = Apple();
a.log(); //可执行从LogUtil继承过来的方法

// 输出结果
this is Fruit constructor with no param
this is Apple constructor with no param
this is a log

```


<h3 id="7"></h3>

## 异步

### Future
一个Future对应一个结果，要么成功，要么失败；其所有API返回值都是一个Future。

#### Future.then
类似于doOnNext，他还有个可选参数onError,可用来捕获异常。

#### Future.catchError

```dart
	Future.delayed(Duration(seconds:2),(){
		return "hello!":
	}).then((data){
		print(data);
	}).catchError((e){
		print(e);
	});
```

#### Future.whenComplete
成功或失败都会执行。

#### Future.wait
接收一个Future数组参数，只有数组中的所有Future执行成功后，才会触发then的成功回调；只要有一个执行失败，则会触发错误回调。

### Async/await
避免回调地狱。写同步代码的方式来写异步任务。

#### 使用Future消除回调地狱

```dart

  Future<String> login(String userName,String pwd){
    ...
  }

  Future<String> getUserInfo(String id){
    ...
  }

  Future<String> saveUserInfo(String userInfo){
    ...
  }
  
  void _login() {
    login("alice", "pwd").then((id){
      return getUserInfo(id);
    }).then((userInfo){
      return saveUserInfo(userInfo);
    }).then((value) => null)
        .catchError((e){
    });

  }

```

#### 使用async/await消除回调地狱

```dart
 task() async{
    try{
      String id = await login("userName", "pwd");
      String userInfo = await getUserInfo(id);
      await saveUserInfo(userInfo);
      //执行接下来的操作
    }catch(e){
      print(e);
    }
  }

```
- async 用于表示函数是异步的,定义的函数会返回一个Future对象，可以使用then来添加回调。
- await 后面跟一个Future，表示等待该任务完成，完成之后才会继续执行下一行，await必须出现在async函数内部了。
- 实际只是语法糖，编译器最终还是会将其转化为Future调用链。

### Stream
可以接收多个异步操作的结果。可以通过多次触发成功或失败事件来传递结果数据或错误异常。

```dart
Stream.fromFutures(<Future>{
      Future.delayed(Duration(seconds: 1),(){
        return "hello";
      }),
      Future.delayed(Duration(seconds: 2),(){
        throw AssertionError("Error");
      }),
      Future.delayed(Duration(seconds: 3),(){
        return "world";
      }),
    }).listen((data) {
      print(data);
    },onError: (e){
      print(e.message);
    },onDone: (){

    });
```
 