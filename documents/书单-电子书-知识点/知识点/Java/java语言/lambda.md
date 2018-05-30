#### 引言
商家有这样一批苹果，每个苹果都有颜色和重量的区别，现商家在制定进销存计划时，需要先对现有的苹果进行分类。

	// 初始化数据
    List<Apple> apples = Arrays.asList(new Apple("red", 10), new Apple("red", 20),new Apple("green", 15), new Apple("yellow", 10));

分类要求：

1. 颜色是红色的
2. 颜色是红色并且重量大于10的。
3. 颜色不是红色的
4. 颜色是红色或者重量大于10的
5. 苹果重量列表

可能看到的第一眼，一般的思维是，每个需求写一个for循环，过滤匹配需要的数据，easy game。但是会有很多重复的代码，因为筛选红色和非红色，只是判断语句不同，甚至筛选重量也是一样的。而且

有经验的可能想到的是设计模式中的 **策略模式**，定义一个接口，不同的需求有不同的实现类，如果不想写多个实现类，也可以使用匿名内部类

	public interface AppleFilter {

	    boolean match(Apple apple);
	}

	public class ColorFilter implements AppleFilter {

	    @Override
	    public boolean match(Apple apple) {
	        return "red".equals(apple.getColor());
	    }
	}

	public static void main(String[] args) {
        
        AppleFilter af = new ColorFilter();
        filter(apples, af);
		filter(apples, new AppleFilter() {
            @Override
            public boolean match(Apple apple) {
                return apple.getWeight() > 10;
            }
        });
    }

    public static List<Apple> filter(List<Apple> apples, AppleFilter filter) {
        List<Apple> result = new ArrayList<>();
        for (Apple t : apples) {
            if (filter.match(t)) {
                result.add(t);
            }
        }
        return result;
    }

```AppleFilter``` 的这种实现方式，其实就是**行为参数化**，虽然参数是一个接口，但是实际起作用的是其match方法，也就是将match这个行为当做参数传给了filter方法。

但是如果我们将其改成lambda表达式，更简单。

	public class Client {

	    public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
	        List<R> result = new ArrayList<>();
	        for (T t : list) {
	            result.add(f.apply(t));
	        }
	        return result;
	    }
	
	    public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	        List<T> result = new ArrayList<>();
	        for (T t : list) {
	            if (p.test(t)) {
	                result.add(t);
	            }
	        }
	        return result;
	    }
	
	    public static void main(String[] args) {
	        // 初始化数据
	        List<Apple> apples = Arrays.asList(new Apple("red", 10), new Apple("red", 20),
	                new Apple("green", 15), new Apple("yellow", 10));
	        // 定义谓词，匹配颜色为红色
	        Predicate<Apple> redApples = apple -> "red".equals(apple.getColor());
	        List<Apple> one = filter(apples, redApples);
	        System.out.println(JSON.toJSONString(one));
	        // 谓词操作，匹配的结果是红色的非
	        Predicate<Apple> notRedApples = redApples.negate();
	        List<Apple> two = filter(apples, notRedApples);
	        System.out.println(JSON.toJSONString(two));
	        // 定义谓词，匹配重要大于10的
	        Predicate<Apple> heavyApples = apple -> apple.getWeight() > 10;
	        // 谓词操作，匹配红色，并且重量大于10的
	        Predicate<Apple> redHeavyApples = redApples.and(heavyApples);
	        List<Apple> three = filter(apples, redHeavyApples);
	        System.out.println(JSON.toJSONString(three));
	        // 谓词操作，匹配红色，或者重量大于10的
	        Predicate<Apple> redOrHeavyApples = redApples.or(heavyApples);
	        List<Apple> four = filter(apples, redOrHeavyApples);
	        System.out.println(JSON.toJSONString(four));
	        // 对象转换，返回集合列表对应的重量属性列表
	        List<Integer> appleWeights = map(apples, Apple::getWeight);
	        System.out.println(JSON.toJSONString(appleWeights));
	        // 排序，按颜色排序
	        apples.sort(Comparator.comparing(Apple::getColor));
	        System.out.println(JSON.toJSONString(apples));
	    }
	}

几行代码就能实现所有的需求。

#### 函数式接口
```@FunctionInteface``` ， 是指，只有一个抽象方法的接口。但是可以包含多个默认实现方法和静态方法。

