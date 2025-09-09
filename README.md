# JUnit Cheat Sheet

JUnit is the most popular unit testing framework for Java applications. JUnit 5 is the current generation, providing a modern foundation for testing with support for new programming models and features.

---

## Table of Contents
- [Installation & Setup](#installation--setup)
- [Basic Test Structure](#basic-test-structure)
- [Annotations](#annotations)
- [Assertions](#assertions)
- [Parameterized Tests](#parameterized-tests)
- [Test Lifecycle](#test-lifecycle)
- [Exception Testing](#exception-testing)
- [Assumptions](#assumptions)
- [Configuration](#configuration)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)

---

## Installation & Setup

### Maven Configuration
```xml
<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- For parameterized tests -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito for mocking -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>4.11.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0-M9</version>
        </plugin>
    </plugins>
</build>
```

### Gradle Configuration
```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    testImplementation 'org.junit.jupiter:junit-jupiter-params:5.9.2'
    testImplementation 'org.mockito:mockito-core:4.11.0'
}

test {
    useJUnitPlatform()
}
```

## Basic Test Structure

### Project Structure
```
src/
├── main/java/
│   └── com/example/
│       ├── Calculator.java
│       └── UserService.java
└── test/java/
    └── com/example/
        ├── CalculatorTest.java
        └── UserServiceTest.java
```

### Simple Test Example
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    
    @Test
    void testAddition() {
        Calculator calculator = new Calculator();
        int result = calculator.add(2, 3);
        assertEquals(5, result);
    }
    
    @Test
    void testDivisionByZero() {
        Calculator calculator = new Calculator();
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
}
```

## Annotations

| Annotation | Description | Usage |
|------------|-------------|-------|
| `@Test` | Marks a method as test | `@Test void testMethod() {}` |
| `@BeforeEach` | Runs before each test | `@BeforeEach void setUp() {}` |
| `@AfterEach` | Runs after each test | `@AfterEach void tearDown() {}` |
| `@BeforeAll` | Runs once before all tests | `@BeforeAll static void init() {}` |
| `@AfterAll` | Runs once after all tests | `@AfterAll static void cleanup() {}` |
| `@DisplayName` | Custom test display name | `@DisplayName("Addition test")` |
| `@Disabled` | Disable test | `@Disabled("Not implemented")` |
| `@RepeatedTest` | Repeat test | `@RepeatedTest(5)` |
| `@Tag` | Tag tests for grouping | `@Tag("unit")` |
| `@Timeout` | Test timeout | `@Timeout(value = 5, unit = SECONDS)` |

### Annotation Examples
```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
import java.time.Duration;

@DisplayName("Calculator Tests")
class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeAll
    static void initAll() {
        System.out.println("Starting calculator tests");
    }
    
    @AfterAll
    static void tearDownAll() {
        System.out.println("Finished calculator tests");
    }
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @AfterEach
    void tearDown() {
        calculator = null;
    }
    
    @Test
    @DisplayName("Adding two positive numbers")
    @Tag("arithmetic")
    void testAddPositiveNumbers() {
        assertEquals(5, calculator.add(2, 3));
    }
    
    @RepeatedTest(value = 5, name = "Repetition {currentRepetition} of {totalRepetitions}")
    void testRandomNumberGeneration() {
        int random = calculator.generateRandom();
        assertTrue(random >= 0 && random <= 100);
    }
    
    @Test
    @Timeout(value = 2, unit = TimeUnit.SECONDS)
    void testSlowOperation() {
        calculator.performSlowCalculation();
    }
    
    @Test
    @Disabled("Feature not yet implemented")
    void testNotImplementedFeature() {
        // This test will be skipped
    }
}
```

## Assertions

### Basic Assertions
```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void testBasicAssertions() {
    // Equality
    assertEquals(4, 2 + 2);
    assertEquals("Hello", "Hello");
    
    // Boolean
    assertTrue(5 > 3);
    assertFalse(5 < 3);
    
    // Null checks
    assertNull(null);
    assertNotNull("Not null");
    
    // Object identity
    String str1 = "Hello";
    String str2 = str1;
    assertSame(str1, str2);
    
    String str3 = new String("Hello");
    assertNotSame(str1, str3);
}
```

### Collection Assertions
```java
@Test
void testCollectionAssertions() {
    List<String> list = Arrays.asList("Java", "Python", "JavaScript");
    
    // Size
    assertEquals(3, list.size());
    
    // Contains
    assertTrue(list.contains("Java"));
    
    // Array equality
    String[] expected = {"Java", "Python", "JavaScript"};
    String[] actual = list.toArray(new String[0]);
    assertArrayEquals(expected, actual);
    
    // Iterable assertions (JUnit 5.8+)
    assertIterableEquals(Arrays.asList("a", "b"), Arrays.asList("a", "b"));
}
```

### Custom Assertions with Messages
```java
@Test
void testCustomMessages() {
    int actual = calculator.multiply(3, 4);
    assertEquals(12, actual, "3 * 4 should equal 12");
    
    // Lazy message evaluation
    assertEquals(12, actual, () -> "Expected 12 but got " + actual);
}
```

### Grouped Assertions
```java
@Test
void testGroupedAssertions() {
    Address address = new Address("123 Main St", "Springfield", "12345");
    
    // All assertions will be evaluated even if some fail
    assertAll("address",
        () -> assertEquals("123 Main St", address.getStreet()),
        () -> assertEquals("Springfield", address.getCity()),
        () -> assertEquals("12345", address.getZipCode())
    );
}
```

## Parameterized Tests

### Basic Parameterized Tests
```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 5, 8, 13})
void testIsPositive(int number) {
    assertTrue(number > 0);
}

@ParameterizedTest
@ValueSource(strings = {"racecar", "radar", "level"})
void testPalindromes(String word) {
    assertTrue(isPalindrome(word));
}
```

### Multiple Parameter Sources
```java
@ParameterizedTest
@CsvSource({
    "1, 1, 2",
    "2, 3, 5",
    "3, 5, 8",
    "5, 8, 13"
})
void testAddition(int first, int second, int expectedResult) {
    assertEquals(expectedResult, calculator.add(first, second));
}

@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testWithCsvFile(String name, int age, String city) {
    // Test with data from CSV file
    assertNotNull(name);
    assertTrue(age > 0);
    assertNotNull(city);
}

@ParameterizedTest
@EnumSource(value = TimeUnit.class, names = {"DAYS", "HOURS"})
void testWithEnumSource(TimeUnit timeUnit) {
    assertNotNull(timeUnit);
}
```

### Method Source
```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana", "orange");
}

@ParameterizedTest
@MethodSource("argumentProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
    assertEquals(5, str.length());
    assertTrue(num >= 1 && num <= 2);
    assertEquals(2, list.size());
}

static Stream<Arguments> argumentProvider() {
    return Stream.of(
        arguments("apple", 1, Arrays.asList("a", "b")),
        arguments("lemon", 2, Arrays.asList("x", "y"))
    );
}
```

## Test Lifecycle

### Lifecycle Methods
```java
class TestLifecycleExample {
    
    @BeforeAll
    static void initAll() {
        // Executed once before all test methods
        // Must be static unless using @TestInstance(Lifecycle.PER_CLASS)
        System.out.println("Before all tests");
    }
    
    @BeforeEach
    void init() {
        // Executed before each test method
        System.out.println("Before each test");
    }
    
    @Test
    void testOne() {
        System.out.println("Test 1");
    }
    
    @Test
    void testTwo() {
        System.out.println("Test 2");
    }
    
    @AfterEach
    void tearDown() {
        // Executed after each test method
        System.out.println("After each test");
    }
    
    @AfterAll
    static void tearDownAll() {
        // Executed once after all test methods
        System.out.println("After all tests");
    }
}
```

### Test Instance Lifecycle
```java
// Default: PER_METHOD (new instance for each test)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class SharedStateTest {
    
    private int counter = 0;
    
    @BeforeAll
    void init() {
        // No need for static when using PER_CLASS
        System.out.println("Initializing shared state");
    }
    
    @Test
    void testIncrement() {
        counter++;
        assertEquals(1, counter);
    }
    
    @Test
    void testIncrementAgain() {
        counter++;
        assertEquals(2, counter); // This will work with PER_CLASS
    }
}
```

## Exception Testing

### Testing Exceptions
```java
@Test
void testExceptionThrown() {
    Calculator calculator = new Calculator();
    
    // Test that exception is thrown
    assertThrows(ArithmeticException.class, () -> {
        calculator.divide(1, 0);
    });
}

@Test
void testExceptionMessage() {
    Calculator calculator = new Calculator();
    
    ArithmeticException exception = assertThrows(ArithmeticException.class, () -> {
        calculator.divide(1, 0);
    });
    
    assertEquals("Division by zero", exception.getMessage());
}

@Test
void testMultipleExceptions() {
    Calculator calculator = new Calculator();
    
    assertAll(
        () -> assertThrows(ArithmeticException.class, () -> calculator.divide(1, 0)),
        () -> assertThrows(IllegalArgumentException.class, () -> calculator.sqrt(-1))
    );
}
```

## Assumptions

### Basic Assumptions
```java
import static org.junit.jupiter.api.Assumptions.*;

@Test
void testOnlyOnLinux() {
    assumeTrue("Linux".equals(System.getProperty("os.name")));
    // Test will only run on Linux
    // rest of test
}

@Test
void testNotOnWindows() {
    assumeFalse(System.getProperty("os.name").contains("Windows"));
    // Test will not run on Windows
    // rest of test
}

@Test
void testConditionally() {
    String env = System.getenv("ENV");
    assumingThat("development".equals(env), () -> {
        // Execute only in development environment
        assertEquals("dev-value", getConfigValue());
    });
    
    // This part always executes
    assertTrue(true);
}
```

## Configuration

### JUnit Configuration File (junit-platform.properties)
```properties
# Configuration parameters
junit.jupiter.conditions.deactivate=org.junit.*DisabledCondition
junit.jupiter.extensions.autodetection.enabled=true
junit.jupiter.testmethod.order.default=org.junit.jupiter.api.MethodOrderer$OrderAnnotation

# Parallel execution
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
junit.jupiter.execution.parallel.config.strategy=dynamic

# Display names
junit.jupiter.displayname.generator.default=org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

### Maven Surefire Configuration
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M9</version>
    <configuration>
        <groups>unit</groups>
        <excludedGroups>integration</excludedGroups>
        <properties>
            <configurationParameters>
                junit.jupiter.execution.parallel.enabled=true
                junit.jupiter.execution.parallel.mode.default=concurrent
            </configurationParameters>
        </properties>
    </configuration>
</plugin>
```

### Test Ordering
```java
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTests {
    
    @Test
    @Order(1)
    void firstTest() {
        // This runs first
    }
    
    @Test
    @Order(2)
    void secondTest() {
        // This runs second
    }
    
    @Test
    @Order(3)
    void thirdTest() {
        // This runs third
    }
}
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Java CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java: [8, 11, 17]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK ${{ matrix.java }}
      uses: actions/setup-java@v3
      with:
        java-version: ${{ matrix.java }}
        distribution: 'temurin'
    
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    
    - name: Run tests
      run: mvn clean test
    
    - name: Generate test report
      run: mvn surefire-report:report-only surefire-report:failsafe-report-only site:site -DgenerateReports=false
    
    - name: Publish test results
      uses: dorny/test-reporter@v1
      if: success() || failure()
      with:
        name: Maven Tests
        path: target/surefire-reports/*.xml
        reporter: java-junit
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.8'
        jdk 'JDK-17'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/your-project.git'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target/site',
                        reportFiles: 'surefire-report.html',
                        reportName: 'Surefire Report'
                    ])
                }
            }
        }
        
        stage('Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Coverage Report'
                    ])
                }
            }
        }
    }
}
```

## Best Practices

### Test Naming
```java
// Good: Descriptive test names
@Test
void shouldReturnTrueWhenNumberIsPositive() {
    assertTrue(NumberUtils.isPositive(5));
}

@Test
void shouldThrowExceptionWhenDividingByZero() {
    assertThrows(ArithmeticException.class, () -> calculator.divide(10, 0));
}

// Use display names for better readability
@DisplayName("When calculating tax for different income brackets")
class TaxCalculatorTest {
    
    @Test
    @DisplayName("should return 0% tax for income below threshold")
    void testLowIncomeNoTax() {
        assertEquals(0, taxCalculator.calculate(10000));
    }
}
```

### Test Structure (AAA Pattern)
```java
@Test
void shouldCalculateCorrectTotal() {
    // Arrange
    Order order = new Order();
    order.addItem(new Item("Book", 25.00));
    order.addItem(new Item("Pen", 2.50));
    
    // Act
    double total = order.calculateTotal();
    
    // Assert
    assertEquals(27.50, total, 0.01);
}
```

### Use of Test Fixtures
```java
class OrderTest {
    
    private Order order;
    private Item book;
    private Item pen;
    
    @BeforeEach
    void setUp() {
        order = new Order();
        book = new Item("Book", 25.00);
        pen = new Item("Pen", 2.50);
    }
    
    @Test
    void shouldCalculateCorrectTotal() {
        order.addItem(book);
        order.addItem(pen);
        
        assertEquals(27.50, order.calculateTotal(), 0.01);
    }
}
```

### Mocking with Mockito
```java
import static org.mockito.Mockito.*;

class UserServiceTest {
    
    @Test
    void shouldReturnUserWhenFound() {
        // Arrange
        UserRepository mockRepository = mock(UserRepository.class);
        User expectedUser = new User("John", "john@example.com");
        when(mockRepository.findById(1L)).thenReturn(expectedUser);
        
        UserService userService = new UserService(mockRepository);
        
        // Act
        User actualUser = userService.getUser(1L);
        
        // Assert
        assertEquals(expectedUser, actualUser);
        verify(mockRepository).findById(1L);
    }
}
```

---

## Resources
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [JUnit 5 Samples](https://github.com/junit-team/junit5-samples)
- [Testing Best Practices](https://phauer.com/2019/modern-best-practices-testing-java/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)

---
*Originally compiled from various sources. Contributions welcome!*