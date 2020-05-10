# JUnit And Mockito

Latest version of Junit is 5.

Dependencies required
```java
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.1.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-params</artifactId>
            <version>5.5.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-all</artifactId>
            <version>1.10.19</version>
            <scope>test</scope>
        </dependency>
```

### Difference between JUnit4 and JUnit5

#### Exception expected
In JUnit4 we write code with @Test to expect an exception in test case. but in JUnit5
we have assertThrow method which takes Exception with lambda function.

JUnit4
```java
@Test(expected = NullPointerException.class)
public void checkException() {
    int[] numbers = null;
    Arrays.sort(numbers);
}
```

JUnit5
```java
    @Test
    public void testExceptionScenaario() {
        assertThrows(NullPointerException.class,() -> {
            int[] numbers = null;
            Arrays.sort(numbers);
        });
    }
```    
#### Timeout
In JUnit4 we write timeout time with @Test annotation but in JUnit5 we have assertTimeout method

JUnit4
```java
@Test(timeout="1000")
public void checkPerformance() {
    int[] array = {34,56,1,44};
    for(int i=0;i<1000000;i++) {
        array[0] = i;
        Arrays.sort(array);
    }
}
```

JUnit5
```java
    @Test
    public void testPerformance() {
        assertTimeout(Duration.ofMillis(100), () -> {
            int[] array = {34,5,1,36};
            for(int i=0;i<1000000;i++) {
                array[0] = i;
                Arrays.sort(array);
            }
        });

    }
```    

### Parameterised tests.
Thes tests are used when we have similar kind of multiple test cases with the only difference of 
test inputs like
```java
    @Test
    public void string1IsNotNull() {
       assertNotNull("Hello");
    }

    @Test
    public void string2IsNotNull() {
         assertNotNull("world");
    }
```

So we can write it as :

```java
    @ParameterizedTest
    @ValueSource(strings = {"Hello", "World"})
    void shouldPassNonNullMessageAsMethodParameter(String message) {
        assertNotNull(message);
    }
```


### Stub testing
Stubs are the dummy implementation of our classes.
 For example we have 2 services
 
 TodoBusinessService
 TodoService
 
 and we are calling todoService in todoBusinessService. For writting test cases of 
 todoBusinessService. we can either mock the todoService or can use stub to give a 
 dummy implementation like :
 
 ```java
 TodoService
 
 public interface TodoService {
     public List<String> retrieveTodos(String user);
 }
 ```
and Stub 

```java
public class TodoServiceStub implements TodoService {
    @Override
    public List<String> retrieveTodos(String user) {
        return Arrays.asList("Learn Spring MVC","Learn Spring","Learn Junit");
    }
}
```

test case 
```java
    @Test
    public void testRetreiveTodosRelatedToSpring() {
        TodoService todoServiceStub = new TodoServiceStub();
        TodoBusinessImpl todoBusiness = new TodoBusinessImpl(todoServiceStub);
        assertEquals(2, todoBusiness.retreiveTodosRelatedToSpring("Dummy").size());
        assertTrue(true);
    }
```    

So the problem with Stub is :                                          

Dynamic data : We have to create different stubs each time we need different dynamic data testing

Extra code : There can be more than 1 method in an interface. So to test 1 service method we have to
implement others also in stub.


### Mock testing
In mock testing, the dependencies are replaced with objects that simulate the behaviour of the real ones.

test case :
```java
   @Test
    public void testRetreiveTodosRelatedToSpring() {
        TodoService todoService = mock(TodoService.class);

        List<String> strings = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn Junit");

        when(todoService.retrieveTodos("Dummy")).thenReturn(strings);

        TodoBusinessImpl todoBusiness = new TodoBusinessImpl(todoService);
        assertEquals(2, todoBusiness.retreiveTodosRelatedToSpring("Dummy").size());
    }
```    

### Returning multiple values.
We can return multiple values in mock statement like :

```java
    @Test
    public void multipleReturnValue() {
        List list = mock(List.class);
        when(list.size()).thenReturn(2).thenReturn(3);
        assertEquals(2,list.size());
        assertEquals(3,list.size());
    }
```

### Argument Matcher
When we want to test any method for general input like :

```java
    @Test
    public void listGetTest() {
        List list = mock(List.class);
        //Argument Matcher
        when(list.get(anyInt())).thenReturn("Alpha");
        assertEquals("Alpha",list.get(2));
        assertEquals("Alpha",list.get(10));
    }
```

We have different types of argument Matchers

anyInt()
anyDouble()
anyObject()
anyString()
anyBoolean()
anyChar() and many more..

they all part of Matchers.class


### BDD (Behavioural driven development)
Its a method in which we develop our code based on scenarios.
It can be divided into 3 parts
Given  -  when  -  then

Given some value to the function
when something got executed on that value
Then some output it will return

example :

Given a customer comes and buy a candy from s shop
when we give the candy and take the money
then our money increases and candy decreases

BDD test case :

```java
    @Test
    public void testRetreiveTodosRelatedToSpring() {
        //Given
        TodoService todoService = mock(TodoService.class);
        List<String> strings = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn Junit");
        given(todoService.retrieveTodos("Dummy")).willReturn(strings);
        TodoBusinessImpl todoBusiness = new TodoBusinessImpl(todoService);

        // when
        int todoCount = todoBusiness.retreiveTodosRelatedToSpring("Dummy").size();

        // then
        assertEquals(2, todoCount);

    }
```    

### Checking method calls in a test case
We can check if any other service call is happening in our test code and how
many times it is happening.

It is used to test the functions which are not returning any value. Are of void types.

like : 
```java
    public void deleteTodoNotRelatedToSprinng(String user) {
        List<String> todos = todoService.retrieveTodos(user);
        todos.stream().filter(x -> !x.contains("Spring")).forEach(x ->
                todoService.deleteTodo(x)
        );
    }
```

Now here we can verify when this todoService deleteTodo method gets called and 
with which parameter and how many times

```java
@Test
    public void testDeleteTodosNotRelatedToSpring() {
        //Given
        TodoService todoService = mock(TodoService.class);

        List<String> strings = Arrays.asList("Learn Spring MVC", "Learn Spring", "Learn Junit");

        given(todoService.retrieveTodos("Dummy")).willReturn(strings);
        TodoBusinessImpl todoBusiness = new TodoBusinessImpl(todoService);

        // when
        todoBusiness.deleteTodoNotRelatedToSprinng("Dummy");

        // Just wanter to test is it calling or not
        verify(todoService).deleteTodo("Learn Junit");

        // exactly wanted to test how many times it is calling
        verify(todoService, times(1)).deleteTodo("Learn Junit");

        // minimum 1 time it should get called
        verify(todoService, atLeastOnce()).deleteTodo("Learn Junit");

        // Atleast 5 times
        verify(todoService, atLeast(5)).deleteTodo(anyString());
        
        // Maximium how many times
        verify(todoService, atMost(6)).deleteTodo(anyString());
        
        //  should never be called 
        verify(todoService,never()).deleteTodo("Learn Spring MVC");

    }

```