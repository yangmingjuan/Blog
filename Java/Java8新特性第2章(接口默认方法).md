###Java8新特性第2章(接口默认方法)
> 转载请注明出处：[http://www.jianshu.com/p/2b6880cb9f37](http://www.jianshu.com/p/2b6880cb9f37)

***

在Java中一个接口一旦发布就已经被定型，除非我们能够一次性的更新所有该接口的实现，否者在接口的添加新方法将会破坏现有接口的实现。默认方法就是为了解决这一问题的，这样接口在发布之后依然能够继续演化。

默认方法就是向接口增加新的行为。它是一种新的方法：接口方法可以是抽象的或者是默认的。默认方法拥有默认实现，接口实现类通过继承得到该默认实现。默认方法不是抽象的，所以我们可以放心的向函数式接口里增加默认方法，而不用担心函数式接口单抽象方法的限制。

	public interface Iterator<E> {

    	boolean hasNext();

    	E next();

    	default void remove() {
        	throw new UnsupportedOperationException("remove");
    	}

    	default void forEachRemaining(Consumer<? super E> action) {
        	Objects.requireNonNull(action);
        	while (hasNext())
           		action.accept(next());
    	}
	}

和其他方法一样，默认方法也可以被继承。

除了上面看到的默认方法，Java8中还允许我们在接口中定义静态方法。这使得我们可以从接口中直接调用它相关的辅助方法，而不是从其它的辅助类中调用（如Collections）。在做集合中元素比较的时候，我们一般需要使用静态辅助方法生成实现Comparator的比较器，在Java8中我们可以直接把该静态方法定义在Comparator接口中：

	public static <T, U extends Comparable<? super U>>
    	Comparator<T> comparing(Function<T, U> keyExtractor) {
  		return (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
	}