## 介绍
Ognl是一个功能强大的表达式语言，用来获取和设置java对象的属性，它旨在提供一个更高抽象语法来对Java对象进行导航
### 三要素
- 表达式
通过表达式来告诉ognl的核心内容，所有的ognl操作都是针对表达式解析后进行的。通过表达式来告诉ognl操作到底要做什么。
- Root对象
我们指定这个表达式针对的具体对象，就是Root对象。
- 上下文环境
在OGNL内部，所有的操作都会在一个特定的数据环境中运行，这个数据环境就是上下文(Context)。OGNL 的上下文环境是一个 Map 结构，称之为 OgnlContext。Root 对象也会被添加到上下文环境当中去。

## 使用OGNL
`Address`类
```
@Data
public class Address {

	private String port;
	private String address;
	
	public Address(String port,String address) {
		this.port = port;
		this.address = address;
	}
}
```
`User`类
```
@Data
public class User {

	private String name;
	private int age;
	private Address address;
	
	public User() {}
	
	public User(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```
#### 对Root对象的访问
OGNL 使用的是一种链式的风格进行对象的访问。
```
User user = new User("test", 23);
Address address = new Address("330108", "杭州市滨江区");
user.setAddress(address);
System.out.println(Ognl.getValue("name", user));	// test
System.out.println(Ognl.getValue("name.length", user));		// 4
System.out.println(Ognl.getValue("address", user));		// Address(port=330108, address=杭州市滨江区)
System.out.println(Ognl.getValue("address.port", user));	// 110003
```
#### 对上下文对象的访问
使用 OGNL 的时候如果不设置上下文对象，系统会自动创建一个上下文对象，如果传入的参数当中包含了上下文对象则会使用传入的上下文对象。
当访问上下文环境当中的参数时候，需要在表达式前面加上 '#' ，表示了与访问 Root 对象的区别。

```
public static String demo2() throws OgnlException {
	User user = new User("test", 23);
	Address address = new Address("330108", "杭州市滨江区");
	user.setAddress(address);
	Map<String, Object> context = new HashMap<String, Object>();
	context.put("init", "hello");
	context.put("user", user);
	System.out.println(Ognl.getValue("#init", context, user));	// hello
	System.out.println(Ognl.getValue("#user.name", context, user));	// test
	System.out.println(Ognl.getValue("name", context, user));	// test
	return "this is demo2 method";
}
```
#### 对静态变量的访问
格式为`@[class]@[field/method ()]`
```
public static String ONE = "one";
// 对静态变量的访问（@[class]@[field/method()]）
public static void demo3() throws OgnlException {
	Object object1 = Ognl.getValue("@sample.ognl.OgnlDemo@ONE", null);
	Object object2 = Ognl.getValue("@sample.ognl.OgnlDemo@demo2()", null);	// hello、test、test
	System.out.println(object1);	// one	
	System.out.println(object2);	// this is demo2 method
}
```