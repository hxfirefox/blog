Groovy/Spock 测试导论
=====================

>原文 http://java.dzone.com/articles/intro-so-groovyspock-testing

>翻译 hxfirefox

测试对于软件开发者而言至关重要，不过总会有人说：“写代码是我的事，测试那是QA的工作”，这样的想法真是弱爆了，因为大量的业界实践已经证明测试驱动编码可以有效地帮助开发者提升代码质量。

大多数遵循TDD的Java开发者均会使用mockito或powermock，但mockito和powermock均包含了许多样本代码，导致测试代码变得冗长而难以维护。在测试中引入Groovy/Spock后，我完全被它们吸引，并转向使用Groovy/Spock来替代原有的测试框架。

下面将围绕一个简单例子来讲解Groovy/Spock，例子中将包含一个service类，负责处理domain对象，以及一个数据访问层。
首先是domain类：

```
public class User {

    private int id;
    private String name;
    private int age;

    // Accessors omitted
}
```
接下来是DAO接口：

```
public interface UserDao {

    public User get(int id);

}
```
最后是service类：

```
public class UserService {

    private UserDao userDao;
    
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public User findUser(int id){
        return null;
    }
}
```
采用Groovy/Spock针对UserService编写测试

```
class UserServiceTest extends Specification {

    UserService service
    UserDao dao = Mock(UserDao)

    def setup(){
      service = new UserService(dao)
    }

    def "it gets a user by id"(){
      given:
      def id = 1

      when:
      def result = service.findUser(id)

      then:
      1 * dao.get(id) >> new User(id:id, name:"James", age:27)
      result.id == 1
      result.name == "James"
      result.age == 27
    }
}
```
上述测试代码中，首先我们使用了groovy，这是一种非常类似Java的语言，但是它的语法更加轻，例如它不用像Java语言那样，在每句结尾加上分号；它也不需要使用public修饰符，因为public是默认的。上述测试类继承自spock.lang.Specification，这是Spock基类，继承该基类后就可以使用given，when，then等代码块。

在Spock中创建mock对象非常容易，只需要使用Mock(Class)这样的语句即可。如上所述，mock后的DAO对象被传入userService中。Setup方法会在每个测试方法运行前被执行。Groovy的一个显著特点是可以使用字符串文本来命名方法，将这个特点应用在测试方法上就能使得测试方法可以更加容易被阅读和理解，如上述代码所示。

**Given, when, then**

Spock是一个BDD测试框架，因此对于Spock中涉及的given，when，then样式最简单的理解就是：
*Given* 给定一些条件，*When* 当执行一些操作时，*Then* 期望得到某个结果。

如上述测试方法中Given，给定id=1，即测试的变量；而在When中则是被测试方法，如在上述代码中调用findUser()；Then中则是断言，即检查被测试方法的输出结果。

上述Then中的第一句语句虽然看上去可怕，但实际上却非常容易理解：

```
1 * dao.get(id) >> new User(id:id, name:"James", age:27)
```
该行表示了对于mock对象dao的期望值，即期望调用dao.get()方法1次，而“>>”是spock的特色，表示“then return”含义。因此该句翻译过来的意思是：期望调用1次dao.get()方法，当执行该方法后，请返回一个新的User对象。此外在构造方法中使用具名参数也是groovy的另一特点。Then中剩余的代码对result对象进行检查。

由此测试代码驱动产生的产品代码非常简单，如下所示：

```
public class UserService {

    private UserDao userDao;
    
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public User findUser(int id){
        return userDao.get(id);
    }
}
```

接下来实现创建用户功能，在UserService中添加如下代码：

```
public void createUser(User user){
        // check name

        // if exists, throw exception

        // if !exists, create user
    }
```
在UserDao中添加如下方法：

```
public User findByName(String name);
public void createUser(User user);
```
相应的测试方法如下：

```
def "it saves a new user"(){
    given:
        def user = new User(id: 1, name: 'James', age:27)
    
    when:
        service.createUser(user)
    
    then:
        1 * dao.findByName(user.name) >> null
    
    then:
        1 * dao.createUser(user)
}
```
在上述代码中出现了两处Then，这是因为当所有断言放在一个then块中，Spock会认为这些断言是同时发生的。如果期望断言按顺序执行，则需要将断言分割到多个then块中，spock会按顺序执行断言。如上述所示，首先需要判断用户是否存在，然后再去创建用户。产品代码实现如下：

```
public void createUser(User user){
    User existing = userDao.findByName(user.getName());
    
    if(existing == null){
        userDao.createUser(user);
    }
}
```

上述代码针对用户不存在场景，而对于用户存在的场景，测试代码如下：

```
def "it fails to create a user because one already exists with that name"(){
    given:
        def user = new User(id: 1, name: 'James', age:27)
    
    when:
        service.createUser(user)
    
    then:
        1 * dao.findByName(user.name) >> user
    
    then:
        0 * dao.createUser(user)
    
    then:
        def exception = thrown(RuntimeException)
        exception.message == "User with name ${user.name} already exists!"
}
```
上述代码当调用findByName时，返回一个存在的用户，然后不调用createUser()，第三个Then块捕获方法抛出的异常。注意groovy拥有一个称之为GStrings的特征，该特征可以在引用的字符串中插入参数，如${user.name}。相应产品代码如下：

```
public void createUser(User user){
    User existing = userDao.findByName(user.getName());
    
    if(existing == null){
        userDao.createUser(user);
    } else{
        throw new RuntimeException(String.format("User with name %s already exists!", user.getName()));
    }
}
```

**提示**

- 最重要也是最容易被遗忘的提示，阅读spock文档！
- 可以命名spock块，例如将given命名为“Some variables”，有助于开发者在测试代码中更加清楚的表达含义
- 当对mock对象方法调用次数不关心时，可以使用_ * mock.method()
- 在then块中可使用下划线来通配方法及类，例如，0 * mock._ 表示期望mock对象的任何方法都未被调用，或0 * _._ 表示期望任何对象的任何方法都未被调用
- 通常按given，when，then编写测试，但实际上从when开始编写测试会更加容易发现测试需要的given和测试的输出结果(then) 
- expect块对于测试不需要对mock对象进行断言的简单方法更加有效
- 当对于传递给mock对象的参数不关注时，可以使用通配符参数
- 拥抱groovy闭包Embrace  groovy closures!  They can be you’re best friend in assertions!
- 当希望在整个测试类中只运行一次，可以复写setupSpec和cleanupSpec

**结论**

测试代码是为了协助开发者的，而不是起相反作用，groovy在这方面提供了很多快捷方式来帮助开发者写出更加优雅的测试代码。完整代码可参考https://gist.github.com/jameselsey/8096211

**思考**

翻译这篇文章是受到了《[使用 Groovy 语言替代 JUnit 来为 Java 程序编写单元测试](https://codingstyle.cn/topics/45)》和《[The Coding Kata: FizzBuzzWhizz in Modern Java](https://codingstyle.cn/topics/100)》两篇文章的启示。除了赞叹两篇文章中采用的测试框架的易用，也深深地被groovy所吸引，其作为DSL的特质不论是对于追求编写更好测试用例的精益开发者还是对于刚入门测试用例的新手开发者来说都是容易掌握和使用的。我们期望测试用例的目标就是能够作为产品代码的 ***living docs***，最佳的效果就是完全摆脱编程语言的语法束缚，成为纯粹的书写或口头表达方式，这样就能“望文生义”。Groovy在这方面确实对于Java测试用例编写起到了促进作用，再加上groovy与Java的无缝融合，及自身拥有的语法特性，在团队中推广groovy替代传统Java测试框架的唯一阻力就剩下大多数开发者是否愿意学习一门新的编程语言。
