###Java8新特性第1章(Lambda表达式)
> 转载请注明出处：[http://www.jianshu.com/p/6e400da4a239](http://www.jianshu.com/p/6e400da4a239)

***

在介绍Lambda表达式之前，我们先来看只有单个方法的Interface（通常我们称之为回调接口）：

	public interface OnClickListener {
		void onClick(View v);
	}
	
我们是这样使用它的：

	button.setOnClickListener(new View.OnClickListener() {
    	@Override
    	public void onClick(View v) {
			v.setText("lalala");
    	}
  	});

这种回调模式在各种框架中非常流行,但是像上面这样的匿名内部类并不是一个好的选择，因为：

* 语法冗余；
* 匿名内部类中的this指针和变量容易产生误解；
* 无法捕获非final局部变量；
* 非静态内部类默认持有外部类的引用，部分情况下会导致外部类无法被GC回收，导致内存泄露。

令人高兴的是Java8为我们带来了Lambda,下面我们看看利用Lambda如何实现上面的功能：

	button.setOnClickListener(v -> v.setText("lalala"));

怎么样？！五行代码用一行就搞定了！！！

> 在这里补充个概念`函数式接口`；前面提到的OnClickListener接口只有一个方法，Java中大多数回调接口都有这个特征：比如Runnable和Comparator；我们把这些只拥有一个方法的接口称之为`函数式接口`。

####1.Lambda表达式
匿名内部类最大的问题在于其冗余的语法，比如前面的OnClickListener中五行代码仅有一行是在执行任务。Lambda表达式是匿名方法，前面我们也看到了它用极其轻量的语法解决了这一问题。

下面给大家看几个Lambda表达式的例子：

	(int x, int y) -> x + y                      //接收x和y两个整形参数并返回他们的和
	() -> 66                                     //不接收任何参数直接返回66
	(String name) -> {System.out.println(name);} //接收一个字符串然后打印出来
	(View view) -> {view.setText("lalala");}     //接收一个View对象并调用setText方法
	
Lambda表达式语法由`参数列表`、`->`和`函数体`组成。函数体既可以是一个表达式也可以是一个代码块。

* __表达式__：表达式会被执行然后返回结果。它简化掉了`return`关键字。
* __代码块__：顾名思义就是一坨代码，和普通方法中的语句一样。

<!--lambda经常出现在嵌套环境中，如作为方法的参数：

	Runnable runnable = () -> {doSomething();};
	new Thread(runnable);
	
	//也可以这样写
	new Thread(() -> {doSomething();});-->

####2.目标类型
通过前面的例子我们可以看到，lambda表达式没有名字，那我们怎么知道它的类型呢？答案是通过上下文推导而来的。例如，下面的表达式的类型是`OnClickListener`

	OnClickListener listener = (View v) -> {v.setText("lalala");};
	
这就意味着同样的lambda表达式在不同的上下文里有不同的类型

	Runnable runnable = () -> doSomething();  //这个表达式是Runnable类型的
	Callback callback = () -> doSomething();  //这个表达式是Callback类型的
	
编译器利用lambda表达式所在的上下文所期待的类型来推导表达式的类型，这个__被期待的类型__被称为`目标类型`。lambda表达式只能出现在__目标类型__为`函数式接口`的上下文中。

Lambda表达式的类型和目标类型的方法签名必须一致，编译器会对此做检查，一个lambda表达式要想赋值给目标类型`T`则必须满足下面所有的条件：

* `T`是一个函数式接口
* lambda表达式的参数必须和`T`的方法参数在数量、类型和顺序上一致（一一对应）
* lambda表达式的返回值必须和`T`的方法的返回值一致或者是它的子类
* lambda表达式抛出的异常和`T`的方法的异常一致或者是它的子类

由于目标类型是知道lambda表达式的参数类型，所以我们没必要把已知的类型重复一遍。也就是说lambda表达式的参数类型可以从目标类型获取：

	//编译器可以推导出s1和s2是String类型
	Comparator<String> c = (s1, s2) -> s1.compareTo(s2);
	//当表达式的参数只有一个时括号也是可以省略的
	button.setOnClickListener(v -> v.setText("lalala"));
	
> ps: Java7中的泛型方法和<>构造器也是通过目标类型来进行类型推导的,如：
>  
 	List<Integer> intList = Collections.emptyList>();
 	List<String> strList = new ArrayList<>();

####3.作用域
在内部类中使用变量名和this非常容易出错。内部类通过继承得到的成员变量（包括来说object的）可能会把外部类的成员变量覆盖掉，未做限制的this引用会指向内部类自己而非外部类。

而lambda表达式的语义就十分简单：它不会从父类中继承任何变量，也不用引入新的作用域。lambda表达式的参数及函数体里面的变量和它外部环境的变量具有相同的语义（this关键字也是一样）。

下面我们举个栗子吧!

	public class HelloLambda {

    	Runnable r1 = () -> System.out.println(this);
    	Runnable r2 = () -> System.out.println(toString());

    	@Override
    	public String toString() {
        	return "Hello, lambda!";
    	}

    	public static void main(String[] args) {
        	new HelloLambda().r1.run();  
        	new HelloLambda().r2.run();
    	}
	}

上面的代码最终会打印两个`Hello, lambda!`，与之相类似的内部类则会打印出类似`HelloLambda$1@32a890`和`HelloLambda$1@6b32098`这种出乎意料的字符串。

总结：基于词法作用域的理念，lambda表达式不可以掩盖任何其所在上下文的局部变量。

####4.变量捕获
在Java7中，编译器对内部类中引用的外部变量（即捕获的变量）要求非常严格：如果捕获的变量没有被声明为`final`就会产生一个编译错误。但是在Java8中放宽了这一限制--对于lambda表达式和内部类，允许在其中捕获那些符合有效只读的局部变量（如果一个局部变量在初始化后从未被修改过，那么它就是有效只读）。

	Runnable getRunnable(String name){
		String hello = "hello";
		return () -> System.out.println(hello+","+name);
	}
对于`this`的引用以及通过`this`对未限定字段的引用和未限定方法的调用本质上都属于使用`final`局部变量。包含此类引用的lambda表达式相当于捕获了`this`实例。在其他情况下，lambda对象不会保留任何对`this`的应用。

这个特性对内存管理是极好的：要知道在java中一个非静态内部类会默认持有外部类实例的强引用，这往往会造成内存泄露。而在lambda表达式中如果没有捕获外部类成员则不会保留对外部类实例的引用。

不过尽管Java8放宽了对捕获变量的语法限制，但试图修改捕获变量的行为是被禁止的，比如下面这个例子就是非法的：

	int sum  = 0;
	list.forEach(i -> {sum += i;});
	
为什么要禁止这种行为呢？因为这样的lambda表达式很容易引起[race condition](https://zh.wikipedia.org/zh-cn/%E7%AB%B6%E7%88%AD%E5%8D%B1%E5%AE%B3)

lambda表达式不支持修改捕获变量的另外一个原因是我们可以使用更好的方式来实现同样的效果：使用规约(condition)。java.util.stream包提供了各种规约操作，关于Java8中的`Stream API`我们放到下一章介绍。

####5.方法引用
lambda表达式允许我们定义一个匿名方法，并以函数式接口的方式使用它。Java8能够在已有的方法上实现同样的特性。

方法引用和lambda表达式拥有相同的特性（他们都需要一个目标类型，并且需要被转化为函数式接口的实例）,不过我们不需要为方法引用提供方法体，我们可以直接通过方法名引用已有方法。

以下面的代码为例，假设我们要按照`userName`排序

	class User{

        private String userName;

        public String getUserName() {
            return userName;
        }
		...
    }
    
    List<User> users = new ArrayList<>();
    Comparator<User> comparator = Comparator.comparing(u -> u.getUserName());
    Collections.sort(users, comparator);
    
我们可以用方法引用替换上面的lambda表达式

	Comparator<User> comparator = Comparator.comparing(User::getUserName);
	
这里的`User::getUserName`被看做是lambda表达式的简写形式。尽管方法引用不一定会把代码变得更紧凑，但它拥有更明确的语义--如果我们想要调用的方法拥有一个名字，那么我们就可以通过方法名调用它。

<!--因为函数式接口的方法参数对应于隐式方法调用时的参数，所以被引用方法签名可以通过放宽类型，装箱以及组织到参数数组中的方式对其参数进行操作，就像在调用实际方法一样：

	Consumer<Integer> b1 = System::exit;    // void exit(int status)
	Consumer<String[]> b2 = Arrays:sort;    // void sort(Object[] a)
	Consumer<String> b3 = MyProgram::main;  // void main(String... args)
	Runnable r = Myprogram::mapToInt        // void main(String... args)-->

方法引用有很多种，它们的语法如下：

* 静态方法引用：ClassName::methodName
* 实例上的实例方法引用：instanceReference::methodName
* 超类上的实例方法引用：super::methodName
* 类型上的实例方法引用：ClassName::methodName
* 构造方法引用：Class::new
* 数组构造方法引用：TypeName[]::new
