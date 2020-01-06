举例说明一下：

有A，B,C三个方法，但是在调用每一个方法之前，要求打印一个日志：某一个方法被开始调用了！

在调用每个方法之后，也要求打印日志：某个方法被调用完了！

一般人会在每一个方法的开始和结尾部分都会添加一句日志打印吧，这样做如果方法多了，就会有很多重复的代码，显得很麻烦，这时候有人会想到，为什么不把打印日志这个功能封装一下，然后让它能在指定的地方（比如执行方法前，或者执行方法后）自动的去调用呢？如果可以的话，业务功能代码中就不会掺杂这一下其他的代码，所以AOP就是做了这一类的工作，比如，日志输出，事务控制，异常的处理等。

AOP的一些术语
==============

#. 通知/增强（Advice）
   
   所谓通知是指拦截到Joinpoint之后所要做的事情就是通知。【增强的逻辑】

   也就是上面说的 安全，事物，日志等。
#. 连接点（JoinPoint）
   
   所谓连接点是指那些被拦截到的点。在spring中,这些点指的是【需要被代理的方法】,因为spring只支持方法类型的连接点。

   连接点是spring允许你使用通知的地方，基本每个方法的前，后（两者都有也行），或抛出异常时都可以是连接点。
#. 切点（Pointcut）
   
   所谓切入点是指我们要对哪些Joinpoint进行拦截的定义。【被代理方法中被增强的方法】

   在上面说的连接点的基础上来定义切点，你的一个类里，有15个方法，那就有几十个连接点了对把，但是你并不想在所有方法附近都使用通知（使用叫织入，以后再说），你只想让其中的几个，在调用这几个方法之前，之后或者抛出异常时干点什么，那么就用切点来定义这几个方法，让切点来筛选连接点，选中那几个你想要的方法。
#. 切面（Aspect）
   
   是切点【被增强的方法】和通知【增强的逻辑方法】（引介）的结合。

   连接点就是为了让你好理解切点，搞出来的，明白这个概念就行了。通知说明了干什么和什么时候干（什么时候通过方法名中的before,after，around等就能知道），而切点说明了在哪干（指定到底是哪个方法），这就是一个完整的切面定义。
#. 引入（introduction）

   引介是一种特殊的通知,在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field。

   允许我们向现有的类添加新方法属性。就是把切面（也就是新方法属性：通知定义的）用到目标类中。
#. 目标（target）
   
   代理的目标对向，也就是被代理的对象

   织入中所提到的目标对象，也就是要被通知的对象，也就是真正的业务逻辑，他可以在毫不知情的情况下，被咱们织入切面。而自己专注于业务本身的逻辑。
#. 代理(proxy)
   
   一个类被AOP织入增强后，就产生一个结果代理类。就是产生的代理对象

   整套aop机制都是通过代理实现。
#. 织入(weaving)
   
   把切面应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。

   这个过程有3种方式，spring采用的是运行时。
关键：切点定义了哪些连接点会得到通知。

**在 Spring AOP 中，所有的可执行方法都是 Join point，所有的 Join point 都可以植入 Advice；而 pointcut 可以看作是一种描述信息，它修饰的是 Join point，用来确认在哪些 Join point 上执行 Advice**

其他的一些内容
=============
AOP中的Joinpoint可以有多种类型：构造方法调用，字段的设置和获取，方法的调用，方法的执行，异常的处理执行，类的初始化。也就是说在AOP的概念中我们可以在上面的这些Joinpoint上织入我们自定义的Advice，但是在Spring中却没有实现上面所有的joinpoint，确切的说，Spring只支持方法执行类型的Joinpoint。

Advice注解一共有五种，分别是：
-----------------------------
#. @Before前置通知
   
   通知在切入点运行前执行，不会影响切入点的逻辑，虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在 before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)
#. @After后置通知
   
   通知在切入点正常运行结束后执行，在一个 join point 正常返回后执行的 advice，如果切入点抛出异常，则在抛出异常前执行。
#. @AfterThrowing异常通知
   
   通知在切入点抛出异常前执行，如果切入点正常运行（未抛出异常），则不执行。
#. @AfterReturning返回通知
   
   通知在切入点正常运行结束后执行，如果切入点抛出异常，则不执行
#. @Around环绕通知
   
   通知是功能最强大的通知，可以在切入点执行前后自定义一些操作。环绕通知需要负责决定是继续处理join point(调用ProceedingJoinPoint的proceed方法)还是中断执行。

在Spring中，通过动态代理和动态字节码技术实现了AOP

使用表达式配置切点

exection(访问修饰符 类全路径.方法名(参数)

#. exection(* com.kulvv.add(..))
#. exection(* com.kulvv.*(..))
#. exection(* *.*(..))
#. exection(* save*(..)) //匹配以save开头的方法

@Aspect

在增强类中配制，配制切面。

@Before("execution(* com.kulvv.aop.User.add(..))")

在增强类中配制，配制增强

@Pointcut

在被增强的类中配制，配制切点，如果省略id属性，切点的名称就是方法名第一个字母小写

在 Spring AOP 中，使用 @Aspect 注解标识一个类是一个切面，然后在切面中定义切点（pointcut）和 增强（advice）

定义需要 aop 拦截的方法，模拟一个 User 的增删改操作：

接口：

public interface IUserService {
    void add(User user);
    User query(String name);
    List<User> qyertAll();
    void delete(String name);
    void update(User user);
}

接口实现：

@Service("userServiceImpl")
public class UserServiceImpl implements IUserService {

    @Override
    public void add(User user) {
        System.out.println("添加用户成功，user=" + user);
    }

    @Override
    public User query(String name) {
        System.out.println("根据name查询用户成功");
        User user = new User(name, 20, 1, 1000, "java");
        return user;
    }

    @Override
    public List<User> qyertAll() {
        List<User> users = new ArrayList<>(2);
        users.add(new User("zhangsan", 20, 1, 1000, "java"));
        users.add(new User("lisi", 25, 0, 2000, "Python"));
        System.out.println("查询所有用户成功, users = " + users);
        return users;
    }

    @Override
    public void delete(String name) {
        System.out.println("根据name删除用户成功, name = " + name);
    }

    @Override
    public void update(User user) {
        System.out.println("更新用户成功, user = " + user);
    }
}

3. 定义 AOP 切面

在 Spring AOP 中，使用 @Aspect 注解标识的类是一个切面，然后在切面中定义切点（pointcut）和 增强（advice）：

3.1 前置增强，@Before()，在目标方法执行之前执行

@Component
@Aspect
public class UserAspectj {

    // 在方法执行之前执行
    @Before("execution(* main.tsmyk.mybeans.inf.IUserService.add(..))")
    public void before_1(){
        System.out.println("log: 在 add 方法之前执行....");
    }
}

上述的方法 before_1() 是对接口的 add() 方法进行 前置增强，即在 add() 方法执行之前执行，

如果想要获取目标方法执行的参数等信息呢，我们可在 切点的方法中添参数 JoinPoint ，通过它了获取目标对象的相关信息：

    @Before("execution(* main.tsmyk.mybeans.inf.IUserService.add(..))")
    public void before_2(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        User user = null;
        if(args[0].getClass() == User.class){
            user = (User) args[0];
        }
        System.out.println("log: 在 add 方法之前执行, 方法参数 = " + user);
    }

3.2 后置增强，@After()，在目标方法执行之后执行，无论是正常退出还是抛异常，都会执行

    // 在方法执行之后执行
    @After("execution(* main.tsmyk.mybeans.inf.IUserService.add(..))")
    public void after_1(){
        System.out.println("log: 在 add 方法之后执行....");
    }

3.3 返回增强，@AfterReturning()，在目标方法正常返回后执行，出现异常则不会执行，可以获取到返回值：

@AfterReturning(pointcut="execution(* main.tsmyk.mybeans.inf.IUserService.query(..))", returning="object")
public void after_return(Object object){
    System.out.println("在 query 方法返回后执行, 返回值= " + object);
}

当一个方法同时被 @After() 和 @AfterReturning() 增强的时候，先执行哪一个呢？

@AfterReturning(pointcut="execution(* main.tsmyk.mybeans.inf.IUserService.query(..))", returning="object")
public void after_return(Object object){
	System.out.println("===log: 在 query 方法返回后执行, 返回值= " + object);
}

@After("execution(* main.tsmyk.mybeans.inf.IUserService.query(..))")
public void after_2(){
	System.out.println("===log: 在 query 方法之后执行....");
}

即使 @After() 放在  @AfterReturning() 的后面，它也先被执行，即 @After() 在 @AfterReturning() 之前执行。

3.4 异常增强，@AfterThrowing，在抛出异常的时候执行，不抛异常不执行。

@AfterThrowing(pointcut="execution(* main.tsmyk.mybeans.inf.IUserService.query(..))", throwing = "ex")
public void after_throw(Exception ex){
	System.out.println("在 query 方法抛异常时执行, 异常= " + ex);
}

现在来修改一下它增强的 query() 方法，让它抛出异常：

@Override
public User query(String name) {
	System.out.println("根据name查询用户成功");
	User user = new User(name, 20, 1, 1000, "java");
	int a = 1/0;
	return user;
}

3.5 环绕增强，@Around，在目标方法执行之前和之后执行

@Around("execution(* main.tsmyk.mybeans.inf.IUserService.delete(..))")
public void test_around(ProceedingJoinPoint joinPoint) throws Throwable {
	Object[] args = joinPoint.getArgs();
	System.out.println("log : delete 方法执行之前, 参数 = " + args[0].toString());
	joinPoint.proceed();
	System.out.println("log : delete 方法执行之后");
}


以上就是 Spring AOP 的几种增强。

上面的栗子中，在每个方法上方的切点表达式都需要写一遍，现在可以使用 @Pointcut 来声明一个可重用的切点表达式，之后在每个方法的上方引用这个切点表达式即可。：

// 声明 pointcut
@Pointcut("execution(* main.tsmyk.mybeans.inf.IUserService.query(..))")
public void pointcut(){
}

@Before("pointcut()")
public void before_3(){
	System.out.println("log: 在 query 方法之前执行");
}
@After("pointcut()")
public void after_4(){
	System.out.println("log: 在 query 方法之后执行....");
}

指示符

在上面的栗子中，使用了 execution 指示符，它用来匹配方法执行的连接点，也是 Spring AOP 使用的主要指示符，在切点表达式中使用了 通配符 (*)  和  (.. )，其中，(* )可以表示任意方法，任意返回值，(..)表示方法的任意参数 ，接下来来看下其他的指示符。
1. within

匹配特定包下的所有类的所有 Joinpoint（方法），包括子包，注意是所有类，而不是接口，如果写的是接口，则不会生效，如 within(main.tsmyk.mybeans.impl.* 将会匹配 main.tsmyk.mybeans.impl 包下所有类的所有 Join point；within(main.tsmyk.mybeans.impl..* 两个点将会匹配该包及其子包下的所有类的所有 Join point。

栗子：

@Pointcut("within(main.tsmyk.mybeans.impl.*)")
public void testWithin(){
}

@Before("testWithin()")
public void test_within(){
	System.out.println("test within 在方法执行之前执行.....");
}

执行该包下的类 UserServiceImpl 的 delete 方法，结果如下：

@Test
public void test5(){
	userServiceImpl.delete("zhangsan");
}

// 结果：
// test within 在方法执行之前执行.....
// 根据name删除用户成功, name = zhangsan

2. @within

匹配所有持有指定注解类型的方法，如 @within(Secure)，任何目标对象持有Secure注解的类方法；必须是在目标对象上声明这个注解，在接口上声明的对它不起作用。
3. target

匹配的是一个目标对象，target(main.tsmyk.mybeans.inf.IUserService) 匹配的是该接口下的所有 Join point ：

@Pointcut("target(main.tsmyk.mybeans.inf.IUserService)")
public void anyMethod(){
}

@Before("anyMethod()")
public void beforeAnyMethod(){
	System.out.println("log: ==== 方法执行之前 =====");
}

@After("anyMethod()")
public void afterAnyMethod(){
	System.out.println("log: ==== 方法执行之后 =====");
}

之后，执行该接口下的任意方法，都会被增强。
3. @target

匹配一个目标对象，这个对象必须有特定的注解，如 

@target(org.springframework.transaction.annotation.Transactional) 匹配任何 有 @Transactional 注解的 方法
4. this

匹配当前AOP代理对象类型的执行方法，this(service.IPointcutService)，当前AOP对象实现了 IPointcutService接口的任何方法
5. arg

匹配参数，

    // 匹配只有一个参数 name 的方法
    @Before("execution(* main.tsmyk.mybeans.inf.IUserService.query(String)) && args(name)")
    public void test_arg(){

    }

    // 匹配第一个参数为 name 的方法
    @Before("execution(* main.tsmyk.mybeans.inf.IUserService.query(String)) && args(name, ..)")
    public void test_arg2(){

    }
    
    // 匹配第二个参数为 name 的方法
    @Before("execution(* main.tsmyk.mybeans.inf.IUserService.query(String)) && args(*, name, ..)")
    public void test_arg3(){

    }

6. @arg

匹配参数，参数有特定的注解，@args(Anno))，方法参数标有Anno注解。
7. @annotation

匹配特定注解

@annotation(org.springframework.transaction.annotation.Transactional) 匹配 任何带有 @Transactional 注解的方法。

8. bean 

匹配特定的 bean 名称的方法

    // 匹配 bean 的名称为 userServiceImpl 的所有方法
    @Before("bean(userServiceImpl)")
    public void test_bean(){
        System.out.println("===================");
    }

    // 匹配 bean 名称以 ServiceImpl 结尾的所有方法
    @Before("bean(*ServiceImpl)")
    public void test_bean2(){
        System.out.println("+++++++++++++++++++");
    }


Spring AOP 原理

Spring AOP 的底层使用的使用 动态代理；共有两种方式来实现动态代理，一个是 JDK 的动态代理，一种是 CGLIB 的动态代理，下面使用这两种方式来实现以上面的功能，即在调用 UserServiceImpl 类方法的时候，在方法执行之前和之后加上日志。
JDK 动态代理

实现 JDK 动态代理，必须要实现 InvocationHandler 接口，并重写 invoke 方法：

public class UserServiceInvocationHandler implements InvocationHandler {

    // 代理的目标对象
    private Object target;

    public UserServiceInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("log: 目标方法执行之前, 参数 = " + args);

        // 执行目标方法
        Object retVal = method.invoke(target, args);

        System.out.println("log: 目标方法执行之后.....");

        return retVal;
    }
}

测试：

public static void main(String[] args) throws IOException {

	// 需要代理的对象
	IUserService userService = new UserServiceImpl();
	InvocationHandler handler = new UserServiceInvocationHandler(userService);
	ClassLoader classLoader = userService.getClass().getClassLoader();
	Class[] interfaces = userService.getClass().getInterfaces();

	// 代理对象
	IUserService proxyUserService = (IUserService) Proxy.newProxyInstance(classLoader, interfaces, handler);

	System.out.println("动态代理的类型  = " + proxyUserService.getClass().getName());
	proxyUserService.query("zhangsan");
    
    // 把字节码写到文件
    byte[] bytes = ProxyGenerator.generateProxyClass("$Proxy", new Class[]{UserServiceImpl.class});
    FileOutputStream fos =new FileOutputStream(new File("D:/$Proxy.class"));
    fos.write(bytes);
    fos.flush();

}

结果：

动态代理的类型  = com.sun.proxy.$Proxy0
log: 目标方法执行之前, 参数 = [Ljava.lang.Object;@2ff4acd0
根据name查询用户成功
log: 目标方法执行之后.....

可以看到在执行目标方法的前后已经打印了日志；刚在上面的 main 方法中，我们把代理对象的字节码写到了文件里，现在来分析下：

反编译 &Proxy.class 文件如下：

可以看到它通过实现接口来实现的。

JDK 只能代理那些实现了接口的类，如果一个类没有实现接口，则无法为这些类创建代理。此时可以使用 CGLIB 来进行代理。
CGLIB 动态代理

接下来看下 CGLIB 是如何实现的。

首先新建一个需要代理的类，它没有实现任何接口：

public class UserServiceImplCglib{
    public User query(String name) {
        System.out.println("根据name查询用户成功, name = " + name);
        User user = new User(name, 20, 1, 1000, "java");
        return user;
    }
}

现在需要使用 CGLIB 来实现在方法 query 执行的前后加上日志：

使用 CGLIB 来实现动态代理，也需要实现接口 MethodInterceptor，重写 intercept 方法：

public class CglibMethodInterceptor implements MethodInterceptor {

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {

        System.out.println("log: 目标方法执行之前, 参数 = " + args);

        Object retVal = methodProxy.invokeSuper(obj, args);

        System.out.println("log: 目标方法执行之后, 返回值 = " + retVal);
        return retVal;
    }
}

测试：

public static void main(String[] args) {

	// 把代理类写入到文件
	System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\");

	Enhancer enhancer = new Enhancer();
	enhancer.setSuperclass(UserServiceImplCglib.class);
	enhancer.setCallback(new CglibMethodInterceptor());

	// 创建代理对象
	UserServiceImplCglib userService = (UserServiceImplCglib) enhancer.create();
	System.out.println("动态代理的类型 = " + userService.getClass().getName());

	userService.query("zhangsan");
}

结果：

动态代理的类型 = main.tsmyk.mybeans.impl.UserServiceImplCglib$$EnhancerByCGLIB$$772edd85
log: 目标方法执行之前, 参数 = [Ljava.lang.Object;@77556fd
根据name查询用户成功, name = zhangsan
log: 目标方法执行之后, 返回值 = User{name='zhangsan', age=20, sex=1, money=1000.0, job='java'}

可以看到，结果和使用 JDK 动态代理的一样，此外，可以看到代理类的类型为 main.tsmyk.mybeans.impl.UserServiceImplCglib$$EnhancerByCGLIB$$772edd85，它是 UserServiceImplCglib 的一个子类，即 CGLIB 是通过 继承的方式来实现的。
总结

1. JDK 的动态代理是通过反射和拦截器的机制来实现的，它会为代理的接口生成一个代理类。

2. CGLIB 的动态代理则是通过继承的方式来实现的，把代理类的class文件加载进来，通过修改其字节码生成子类的方式来处理。

3. JDK 动态代理只能对实现了接口的类生成代理，而不能针对类。

4. CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，但是因为采用的是继承， 所以 final 类或方法无法被代理。

5. Spring AOP 中，如果实现了接口，默认使用的是 JDK 代理，也可以强制使用 CGLIB 代理，如果要代理的类没有实现任何接口，则会使用 CGLIB 进行代理，Spring 会进行自动的切换。
   
逻辑运算符

表达式可由多个切点函数通过逻辑运算组成

 &&

与操作，求交集，也可以写成and

例如 execution(* chop(..)) && target(Horseman)  表示Horseman及其子类的chop方法

 ||

或操作，求并集，也可以写成or

例如 execution(* chop(..)) || args(String)  表示名称为chop的方法或者有一个String型参数的方法

!

非操作，求反集，也可以写成not

例如 execution(* chop(..)) and !args(String)  表示名称为chop的方法但是不能是只有一个String型参数的方法

 
execution常用于匹配特定的方法，如update时怎么处理，或者匹配某些类，如所有的controller类，是一种范围较大的切面方式，多用于日志或者事务处理等。

其他的几个用法各有千秋，视情况而选择。


execution切点函数

 

execution函数用于匹配方法执行的连接点，语法为：

execution(方法修饰符(可选)  返回类型  方法名  参数  异常模式(可选)) 

参数部分允许使用通配符：

*  匹配任意字符，但只能匹配一个元素

.. 匹配任意字符，可以匹配任意多个元素，表示类时，必须和*联合使用

+  必须跟在类名后面，如Horseman+，表示类本身和继承或扩展指定类的所有类

 

除了execution()，Spring中还支持其他多个函数，这里列出名称和简单介绍，以方便根据需要进行更详细的查询

 @annotation()

表示标注了指定注解的目标类方法

例如 @annotation(org.springframework.transaction.annotation.Transactional) 表示标注了@Transactional的方法

args()

通过目标类方法的参数类型指定切点

例如 args(String) 表示有且仅有一个String型参数的方法

@args()

通过目标类参数的对象类型是否标注了指定注解指定切点

如 @args(org.springframework.stereotype.Service) 表示有且仅有一个标注了@Service的类参数的方法

within()

通过类名指定切点

如 with(examples.chap03.Horseman) 表示Horseman的所有方法

target()

通过类名指定，同时包含所有子类

如 target(examples.chap03.Horseman)  且Elephantman extends Horseman，则两个类的所有方法都匹配

@within()

匹配标注了指定注解的类及其所有子类

如 @within(org.springframework.stereotype.Service) 给Horseman加上@Service标注，则Horseman和Elephantman 的所有方法都匹配

@target()

所有标注了指定注解的类

如 @target(org.springframework.stereotype.Service) 表示所有标注了@Service的类的所有方法

 this()

大部分时候和target()相同，区别是this是在运行时生成代理类后，才判断代理类与指定的对象类型是否匹配
