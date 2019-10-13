### mycat
mycat可以认为是一层在应用与数据库之间的代理，通过配置分片规则，其会解析sql并根据对应规则发送给合适的实际数据库执行。

mycat本身不提供主从复制的功能，其读写分离依赖于mysql的主从配置，mysql的主从同步是通过同步Master节点的binlog实现的

mycat的集群配置，可以通过在应用与mycat之间架设一层Haproxy层实现。

mycat自带了很多切片规则，取模，一致性hash，日期，范围等等

mycat跨片join，目前有：全局表，ER分片，sharejoin等。sharejoin目前只支持2个表的join，其是通过先根据条件获取数据关联的ID，然后匹配另一个表的数据实现的；
全局表是在每个节点上会新增修改这些操作，保证每个节点数据的一致性，也就是每个节点都有全部数据，所以不适合大数据量的，一般是字典数据这种，其关联查询时，只会
匹配需要关联的节点上的表数据；ER分片依赖sharding-by-intfile分片策略，关联表的数据(子表与父表)在同一个数据分片上。

mycat实现数据库多租户：mycat作为mysql的代理，web 提交的 sql 过来时在注释中指定 schema, proxy 层根据指定的 schema 转发 sql 请求

### mysql锁
共享锁：阻塞其他会话获取排它锁；排它锁：阻塞其他会话获取任何锁。

锁与索引有关，只有匹配索引条件时，才是行锁，否则是表锁

行锁与表锁：innodb引擎支持行锁以及表锁

间隙锁：范围查询，即使数据表不存在那些数据，也会被锁住。比如数据表现有ID为1-5的数据，获取锁时，条件为 id>4，则锁未释放的话，insert id=6 是无法成功的。

### mysql索引
B-tree，是一种多路自平衡树，允许有多个子节点，可以减少树的高度，具有更高的查询效率

每个索引节点占用一个磁盘块，每个索引节点可以包括多个索引数据，查找时，需要加载磁盘块到内存进行比较，所以索引的长度不宜过长，较短的索引项使得每个磁盘
块可以容纳更多的数据，减少磁盘IO次数

聚簇索引，每个索引节点只包含key以及指针，最末的叶子节点包含数据。聚簇索引节点的key必然是数据表的主键。非主键的索引的节点记录的是对应的主键，
然后通过主键查找数据。

最左匹配特性，比如(name,age,sex)的时候，b+数是按照从左到右的顺序来建立搜索树的；范围查询之后的索引条件无法使用索引

尽量使用区分度高的列做索引。

慢查询

0. 先运行看看是否真的很慢，注意设置SQL_NO_CACHE
1. where条件单表查，锁定最小返回记录表。这句话的意思是把查询语句的where都应用到表中返回的记录数最小的表开始查起，单表每个字段分别查询，
看哪个字段的区分度最高
2. explain查看执行计划，是否与1预期一致（从锁定记录较少的表开始查询）
3. order by limit 形式的sql语句让排序的表优先查
4. 了解业务方使用场景
5. 加索引时参照建索引的几大原则
6. 观察结果，不符合预期继续从0分析

### string为什么是final的
final是不可变，线程安全的，java中大部分对象都是String类型的，为了提高使用效率，节省内存，java会将字面量和符号引用缓存到常量池，如果string是可变的，
那么常量池就无法实现。

### 动态代理能否代理静态方法和final方法
不能。

- JDK动态代理，其代理对象必须实现接口，而接口没有静态方法和final方法。
- cglib动态代理，其是通过字节码技术子类继承目标类实现的，final方法无法被子类继承，静态方法又不是实例方法，所以无法代理。（final方法通过代理类调用时，
实际就是调用的父类的方法，可以执行，不会报错，只不过不会被代理）

JDK动态代理实现

    public class TargetProxy implements InvocationHandler {

        private Object target;
        public TargetProxy(Object target) {
            this.target = target;
        }

        public Object getProxy() {
            return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("before...");
            Object result = method.invoke(target, args);
            System.out.println("after...");
            return result;
        }
    }
    
    public static void callJdk() {
        RealSubject object = new RealSubject();
        TargetProxy targetProxy = new TargetProxy(object);
        Subject subject = (Subject) targetProxy.getProxy();
        subject.show();
    }

cglib实现：

    public class CglibProxy implements MethodInterceptor {

        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            System.out.println("before...");
            Object result = proxy.invokeSuper(obj, args);
            System.out.println("after...");
            return result;
        }
    }
    
    public static void callCglib() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(RealSubject.class);
        enhancer.setCallback(new CglibProxy());
        RealSubject subject = (RealSubject) enhancer.create();
        subject.show();
        subject.testFinal();
    }

### JUC
ConcurrentLinkedQueue：线程安全的队列，底层是链表，通过CAS实现线程安全

CopyOnWriteArrayList：对数据进行修改时，通过复制一份数据进行操作，最后替换原始数据

DelayQueue：延时队列，内部维护了一个优先级队列(PriorityQueue)，越临近的任务优先级越高。通过ReentrantLock实现实现安全的操作。调用take()时，
如果达到了延时时间，则直接出队；否则阻塞等待剩余延迟时间


### AQS
诸如ReentrantLock，Semophore，CountDownLatch等都是基于AQS实现，其模板化了一般共享资源获取失败后地等待模型和操作方法。

其内部维护了一个等待队列，获取共享资源失败的线程会添加到等待队列的尾部，并判断其前置节点的状态，如果为 Signal，则等待线程挂起。

- signal(-1)表示后继节点等待当前节点的唤醒，所以新线程入队时，会将其前继节点状态设置为signal。有线程释放共享资源时，会尝试唤醒当前线程的下一个节点，当然，此时非公平模式下，可能新的线程抢占了资源，那么这个节点会继续挂起
- cancelled(1)表示节点已经被取消，不再参与竞争。
- condition(-2)表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁
- propagate(-3)共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点

对于非公平锁，获取共享资源时，只要判断共享资源当前没有被其他线程持有，就有机会获得；而对于公平锁，即使获取共享资源的瞬间没有其他线程持有，只要等待队列有正常等待共享资源的线程，当前线程都只能添加到等待队列，而不能直接获取。

可重入时通过AQS内部的 state 状态判断维护的，当前线程每次获取共享资源，state都会加一，所以最后也要相应的减1，直到state=0，其他线程才能获取到。
