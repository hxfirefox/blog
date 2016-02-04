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
Nothing too complex to mention here. The class that we’re going to put under test is the service. You can see that the service is dependant on a UserDao, which is passed into the constructor. This is a good design practice because you’re stating that in order to have a UserService, it must be constructed with a UserDao. This also becomes useful later when using dependency injection frameworks like Spring so you can mark them both as Components and Autowire the constructor arguments in, but alas.

针对UserService编写测试

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

Here we go, right in at the deep end, let me explain what is going on here. Firstly, we’re using groovy, so although it looks like Java (I suppose it is in some respects as it compiles down to Java bytecode anyway) the syntax is a bit lighter, such as no semi-colons to terminate statements, no need for public accessor as everything is public by default, Strings for method names. If you want to learn more about groovy, check out their documentation  here .

As you can see, the test class extends from spock.lang.Specification, this is a Spock base class and allows us to use the given, when and then blocks in our test.

You’ll see the subject of the test then, the service. I prefer to define this as a field and assign it in the setup, but others prefer to instantiate it in the given block of each test, I suppose this is really just a personal preference.

Creating mocks with Spock is easy, just use Mock(Class). I then pass the mocked DAO dependency into the userService in the setup method. Setup runs before each test is executed( likewise, cleanup() is run after each test completes). This is an excellent pattern for testing as you can mock out all dependencies and define their behaviour, so you’re literally just testing the service class.

A great feature of groovy is that you can use String literals to name your methods, this makes tests much easier to read and work out what it is actually testing rather than naming them as “public void testItGetsAUserById()”

Given, when, then
Spock is a  behaviour driven development  (BDD) testing framework, which is where it gets the given, when and then patterns from (amongst others). The easiest way I can explain it as follows:

Given some parts, when you do something, then you expect certain things to happen.

It’s probably easier to explain my test. We’re given an id of 1, you can think of this as a variable for the test. The when block is where the test starts, this is the invocation, we’re saying that when we call findUser() on the service passing in an id, we’ll get something back and assign it to the result.

The then block are your assertions, this is where you check the outcomes. The first line in the then block looks a little scary, but actually it’s very simple, lets dissect it.

1 * dao.get(id) >> new User(id:id, name:"James", age:27)
This line is setting an expectation on the mocked dao. We’re saying that we expect 1 (and only 1) invocation on the dao.get() method, that invocation must be passed id (which we defined as 1 earlier). Still with me? Good, we’re half way.

The double chevron “>>” is a spock feature, it means “then return”. So really this line reads as “we expect 1 hit on the mocked dao get(), and when we do, return a new User object”

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
