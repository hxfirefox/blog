Groovy/Spock 测试导论
=====================

>原文 http://java.dzone.com/articles/intro-so-groovyspock-testing

>翻译 hxfirefox

测试对于软件开发者而言至关重要，不过总会有人说：“写代码是我的事，测试那是QA的工作”，这样的想法真是弱爆了。测试驱动编码已经被证明可以有效地帮助开发者提升代码质量。

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
上述测试代码中，首先我们使用了groovy，这是一种非常类似Java的语言，但是它的语法更加轻，例如它不用像Java语言那样，在每句结尾加上分号；它也不需要使用public修饰符，因为默认修饰符就是public的。上述测试类继承自spock.lang.Specification，这是Spock基类，继承该基类后就可以使用given，when，then等代码块。

在Spock中创建mock对象非常容易，只需要使用Mock(Class)这样的语句即可。如上所述，mock后的DAO对象被传入userService中。Setup方法会在每个测试方法运行前被执行。Groovy的一个显著特点是可以使用字符串文本来命名方法，将这个特点应用在测试方法上就能使得测试方法可以更加容易被阅读和理解，如上述代码所示。

**Given, when, then**

Spock是一个BDD测试框架，因此对于Spock中涉及的given，when，then样式最简单的理解就是：
*Given* 给定一些条件，*When* 当执行一些操作时，*Then* 将期望得到某个结果。

如上述测试方法中Given，给定id=1，即测试的变量；而在When中则是被测试方法，如在上述代码中调用findUser()；Then中则是断言，即检查被测试方法的输出结果。

上述Then中的第一句语句虽然看上去可怕，但实际上却非常容易理解：

```
1 * dao.get(id) >> new User(id:id, name:"James", age:27)
```
该行表示了对于mock对象dao的期望值，即期望调用dao.get()方法1次，而“>>”是spock的特色，表示“then return”含义。因此该句翻译过来的意思是：期望调用1次dao.get()方法，当执行该方法后，请返回一个新的User对象。

You can also see that I’m using named parameters in the constructor of the User object, this is another neat little feature of groovy.

The rest of the then block are just assertions on the result object, not really required here as we’re doing a straight passthrough on the dao, but gives an insight as to what you’d normally want to do in more complex examples.

The implementation.
If you run the test, it’ll fail, as we haven’t implemented the service class, so lets go ahead and do that right now, its quite simple, just update the service class to the following:

public class UserService {

  private UserDao userDao;

  public UserService(UserDao userDao) {
    this.userDao = userDao;
  }

  public User findUser(int id){
    return userDao.get(id);
  }
}
Run the test again, it should pass this time.

Stepping it up
That was a reasonably simple example, lets look at creating some users. Add the following into the UserService:

public void createUser(User user){
        // check name

        // if exists, throw exception

        // if !exists, create user
    }
Then add these methods into the UserDao

public User findByName(String name);
    public void createUser(User user);
Then start with this test

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
This time, we’re testing the createUser() method on the service, you’ll notice that there is nothing returned this time.

You may be asking “why are there 2 then blocks?”, if you group everything into a single then block, Spock just asserts that they all happen, it doesn’t care about ordering. If you want ordering on assertions then you need to split into separate then blocks, spock then asserts them in order. In our case, we want to firstly find by user name to see if it exists, THEN we want to create it.

Run the test, it should fail. Implement with the following and it’ll pass

public void createUser(User user){
        User existing = userDao.findByName(user.getName());

        if(existing == null){
            userDao.createUser(user);
        }
    }
Thats great for scenarios where the user doesn’t already exist, but what if it does? Lets write so co…NO! Test first!

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
This time, when we call findByName, we want to return an existing user. Then we want 0 interactions with the createUser() mocked method.

The third then block grabs hold of the thrown exception by calling thrown() and asserts the message. Note that groovy has a neat feature called GStrings that allow you to put arguments inside quoted strings.

Run the test, it will fail. Implement with the following at it’ll pass.

public void createUser(User user){
        User existing = userDao.findByName(user.getName());

        if(existing == null){
            userDao.createUser(user);
        } else{
            throw new RuntimeException(String.format("User with name %s already exists!", user.getName()));
        }
    }
I’ll leave it there, that should give you a brief intro to Spock, there is far more that you can do with it, this is just a basic example.

Snippets of wisdom
Read the  spock documentation!
You can name spock blocks such as given:”Some variables”, this is useful if its not entirely clear what your test is doing.
You can use _ * mock.method() when you don’t care how many times a mock is invoked.
You can use underscores to wildcard methods and classes in the then block, such as 0 * mock._ to indicate you expect no other calls on the mock, or 0 * _._ to indicate no calls on anything.
I often write the given, when and then blocks, but then I start from the when block and work outwards, sounds like an odd approach but I find it easier to work from the invocation then work out what I need (given) and then what happens(then).
The expect block is useful for testing simpler methods that don’t require asserting on mocks.
You can wildcard arguments in the then block if you don’t care what gets passed into mocks.
Embrace  groovy closures!  They can be you’re best friend in assertions!
You can override setupSpec and cleanupSpec if you want things to run only once for the entire spec.
Conclusion
Having used Spock (and groovy for testing) on various work and hobby projects I must admit I’ve become quite a fan. Test code is there to be an aid to the developer, not a hinderance. I find that groovy has many shortcuts (collections API to name but a few!) that make writing test code much nicer.

You can view the full Gist here https://gist.github.com/jameselsey/8096211
