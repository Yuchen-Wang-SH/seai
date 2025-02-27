---
author: Christian Kaestner
title: "17-445: Infrastructure Quality"
semester: Fall 2019
footer: "17-445 Software Engineering for AI-Enabled Systems, Christian Kaestner"
license: Creative Commons Attribution 4.0 International (CC BY 4.0)
---

# Infrastructure Quality

Christian Kaestner

<!-- references -->

Required reading: Eric Breck, Shanqing Cai, Eric Nielsen, Michael Salib, D. Sculley. [The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction](https://research.google.com/pubs/archive/46555.pdf). Proceedings of IEEE Big Data (2017)

---

# Learning Goals

* Implement and automate tests for all parts of the ML pipeline
* Test for robustness
* Understand testing opportunities beyond functional correctness
* Automate test execution with continuous integration
* Understand the idea of chaos engineering



---
# Beyond Model and Data Quality
----
## Possible Mistakes in ML Pipelines

Danger of "silent" mistakes in many phases

<!-- discussion -->
----
![](mltestingandmonitoring.png)

<!-- references -->
Source: Eric Breck, Shanqing Cai, Eric Nielsen, Michael Salib, D. Sculley. [The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction](https://research.google.com/pubs/archive/46555.pdf). Proceedings of IEEE Big Data (2017)
----
## Possible Mistakes in ML Pipelines

Danger of "silent" mistakes in many phases:

* Dropped data after format changes
* Failure to push updated model into production
* Incorrect feature extraction
* Use of stale dataset, wrong data source
* Data source no longer available (e.g web API)
* Telemetry server overloaded
* Negative feedback (telemtr.) no longer sent from app
* Use of old model learning code, stale hyperparameter
* Data format changes between ML pipeline steps


----
## Everything can be tested?

<!-- discussion -->

Notes: Many qualities can be tested beyond just functional correctness (for a specification). Examples: Performance, model quality, data quality, usability, robustness, ... not all tests are equality easy to automate
----
## Testing Strategies

* Performance
* Scalability
* Robustness
* Safety
* Security
* Extensibility
* Maintainability
* Usability

**How to test for these? How automatable?**













---
# Test Automation

----
## From Manual Testing to Continuous Integration

<!-- colstart -->
![Manual Testing](manualtesting.jpg)
<!-- col -->
![Continuous Integration](ci.png)
<!-- colend -->
----
## Unit Test, Integration Tests, System Tests

![Testing levels](testinglevels.png)

Notes:

Software is developed in units that are later assembled. Accordingly we can distinguish different levels of testing.

Unit Testing - A unit is the "smallest" piece of software that a developer creates. It is typically the work of one programmer and is stored in a single file. Different programming languages have different units: In C++ and Java the unit is the class; in C the unit is the function; in less structured languages like Basic and COBOL the unit may be the entire program.

Integration Testing - In integration we assemble units together into subsystems and finally into systems. It is possible for units to function perfectly in isolation but to fail when integrated. For example because they share an area of the computer memory or because the order of invocation of the different methods is not the one anticipated by the different programmers or because there is a mismatch in the data types. Etc.

System Testing - A system consists of all of the software (and possibly hardware, user manuals, training materials, etc.) that make up the product delivered to the customer. System testing focuses on defects that arise at this highest level of integration. Typically system testing includes many types of testing: functionality, usability, security, internationalization and localization, reliability and availability, capacity, performance, backup and recovery, portability, and many more. 

Acceptance Testing - Acceptance testing is defined as that testing, which when completed successfully, will result in the customer accepting the software and giving us their money. From the customer's point of view, they would generally like the most exhaustive acceptance testing possible (equivalent to the level of system testing). From the vendor's point of view, we would generally like the minimum level of testing possible that would result in money changing hands.
Typical strategic questions that should be addressed before acceptance testing are: Who defines the level of the acceptance testing? Who creates the test scripts? Who executes the tests? What is the pass/fail criteria for the acceptance test? When and how do we get paid?

----
## What's a good Unit in Unit Testing?

<!-- discussion -->

----

## What's a good Unit in Unit Testing?

* Smallest programming abstraction that can be tested independently 
* Smallest programming abstraction that can be assigned to a single developer 
  * OO languages: classes or methods
  * Procedural languages: procedures, subroutines 
  * Functional languages: functions 
*
* Test against unit's contract (interface)

----
## Recall: Who is to blame?

```java
class Algorithms {
    /**
     * This method finds the shortest distance between to 
     * verticies. It returns -1 if the two nodes are not 
     * connected. 
    */
    int shortestDistance(…) {…}
}
```
```java
class Algorithms {
    /**
     * This method finds the shortest distance between to 
     * verticies. Method is only supported 
     * for connected verticies.
    */
    int shortestDistance(…) {…}
}
```

----
## Anatomy of a Unit Test

```java
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class AdjacencyListTest {
    @Test
    public void testSanityTest(){
        // set up
        Graph g1 = new AdjacencyListGraph(10);
        Vertex s1 = new Vertex("A");
        Vertex s2 = new Vertex("B");
        // check expected results (oracle)
        assertEquals(true, g1.addVertex(s1));
        assertEquals(true, g1.addVertex(s2));
        assertEquals(true, g1.addEdge(s1, s2));
        assertEquals(s2, g1.getNeighbors(s1)[0]);
    }

    // use abstraction, e.g. common setups
    private int helperMethod…
}
```

----
## Unit tests should... (Best Practices)


* be independent -- can execute in any order and any subsets
* be repeatable -- no special knowledge needed to execute tests
* lend well to automation
* test organization mirrors code organization
* have meaningful names -- `testShouldReturnLargestNumberInArrayWithoutRepeats()`
* be able to fail
* be comprehensive -- happy and sad paths, corner cases, boundaries
* be timely -- execute quickly

----
## Behavior-Driven Testing


```text
A Stack
- should pop values in last-in-first-out-order
  + Given a non-empty stack 
  + When pop is invoked on the stack 
  + Then the most recently pushed element should be returned 
  + And the stack should have one less item than before 
- should throw NoSuchElementException if an empty stack is popped
  + Given an empty stack 
  + When pop is invoked on the stack 
  + Then NoSuchElementException should be thrown 
  + And the stack should still be empty 
```

----
## Behavior-Driven Testing
```scala
import org.scalatest.FunSpec
import scala.collection.mutable.Stack
class StackSpec extends FunSpec {
  describe("A Stack") {
    it("should pop values in last-in-first-out order") {
      val stack = new Stack[Int]
      stack.push(1); stack.push(2)
      assert(stack.pop() === 2); assert(stack.pop() === 1)
    }
    it("should throw NoSuchElementException if an empty stack is popped") {
      val emptyStack = new Stack[String]
      intercept[NoSuchElementException] {
        emptyStack.pop()
      }
    }
  }
}
```


----
## Unit Testing Pitfalls

* Working code, failing tests
* Smoke tests pass
* Works on my (some) machine(s)
* Tests break frequently

**How to avoid?**

----
## How to unit test component with dependency on other code?

<!-- discussion -->

----
## Example: Using Test Data

```java
DataTable getData(Stream stream, DataCleaner cleaner) { ... }

@Test void test() {
    Stream stream = openKafkaStream(...)
    DataTable output = getData(testStream, new DefaultCleaner());
    assert(output.length==10)
}
```

----
## Example: Using Test Data

```java
DataTable getData(Stream stream, DataCleaner cleaner) { ... }

@Test void test() {
    Stream testStream = new Stream() {
        int idx = 0;
        // hardcoded or read from test file
        String[] data = [ ... ] 
        public void connect() { }
        public String getNext() {
            return data[++idx];
        }
    }
    DataTable output = getData(testStream, new DefaultCleaner());
    assert(output.length==10)
}
```

----
## Example: Mocking a DataCleaner Object

```java
DataTable getData(KafkaStream stream, DataCleaner cleaner) { ... }

@Test void test() {
    DataCleaner dummyCleaner = new DataCleaner() {
        boolean isValid(String row) { return true; }
        ...
    }
    DataTable output = getData(testStream, dummyCleaner);
    assert(output.length==10)
}
```
----
## Example: Mocking a DataCleaner Object

```java
DataTable getData(KafkaStream stream, DataCleaner cleaner) { ... }

@Test void test() {
    DataCleaner dummyCleaner = new DataCleaner() {
        int counter = 0;
        boolean isValid(String row) { 
            counter++;
            return counter!=3; 
        }
        ...
    }
    DataTable output = getData(testStream, dummyCleaner);
    assert(output.length==9)
}
```

Mocking frameworks provide infrastructure for expressing such tests compactly.

----
## How to select test cases?

Think like an attacker:
     The tester’s goal is to find bugs!

<!-- colstart -->
**Black box**

* Read specification
* Write tests for
    * Representative case
    * Invalid cases
    * Boundary conditions
* Are there difficult cases? (error guessing)
* How many test should you write?
    * Aim to cover the specification
    * Work within time/money constraints
<!-- col -->

**White-box**

* Analyze code paths
* Write tests to cover
    * all statements
    * all branches
    * all important paths
* Identify inputs that may lead to exception paths
<!-- colend -->

----
![Coverage report](coverage.png)

----
## Testable Code

* Think about testing when writing code
* Unit testing encourages you to write testable code
* Separate parts of the code to make them independently testable
* Abstract functionality behind interface, make it replaceable
*
* Test-Driven Development: A design and development method in which you write tests before you write the code


----
## Integration and system tests

![Testing levels](testinglevels.png)

----
## Integration and system tests

Test larger units of behavior

Often based on use cases or user stories -- customer perspective

```java
@Test void gameTest() {
    Poker game = new Poker();
    Player p = new Player();
    Player q = new Player();
    game.shuffle(seed)
    game.add(p);
    game.add(q);
    game.deal();
    p.bet(100);
    q.bet(100);
    p.call();
    q.fold();
    assert(game.winner() == p);
}

```
----
## Test Adequacy?

* How good is the test suite?
* When have we tested enough?

<!-- discussion -->


----
## Build systems & Continuous Integration

* Automate all build, analysis, test, and deployment steps from a command line call
* Ensure all dependencies and configurations are defined
* Ideally reproducible and incremental
* Distribute work for large jobs
* Track results
*
* Key CI benefit: Tests are regularly executed, part of process

----
![Continuous Integration example](ci.png)

----
## Tracking Build Quality

Track quality indicators over time, e.g.,
* Build time
* Test coverage
* Static analysis warnings
* Performance results
* Model quality measures
* Number of TODOs in source code


----
[![Jenkins Dashboard with Metrics](https://blog.octo.com/wp-content/uploads/2012/08/screenshot-dashboard-jenkins1.png)](https://blog.octo.com/en/jenkins-quality-dashboard-ios-development/)

<!-- references -->

Source: https://blog.octo.com/en/jenkins-quality-dashboard-ios-development/








---
# Local Improvements vs Overall Quality


* Ideally unit tests catch bugs locally
* Some bugs emerge from interactions among system components
    - Missed local specifications -> more unit tests
    - Nonlocal effects, interactions -> integration & system tests
    

Known as **emergent properties** and **feature interactions**

----
## Feature Interaction Examples

![Flood and Fire Control](interactionflood.png)

Notes: Flood control and fire control work independently, but interact on the same resource (water supply), where flood control may deactivate the water supply to the sprinkler system in case of a fire

----
## Feature Interaction Examples

![Parking break and AC](interactionparkingbreak.png)

Notes: Electronic parking brake and AC are interacting via the engine. Electronic parking brake gets released over a certain engine speed and AC may trigger that engine speed (depending on temperature and AC settings).
----
## Feature Interaction Examples

![Wordpress Plugins for Smiley and Weather](interactionwordpress.png)


Notes: Weather and smiley plugins in WordPress may work on the same tokens in a blog post (overlapping preconditions)

----
## Feature Interaction Examples

![Call forwarding and call waiting](interactiontelecom.png)


Notes: Call forwarding and call waiting in a telecom system react to the same event and may result in a race condition. This is typically a distributed system with features implemented by different providers.

----
## Feature Interactions

Failure in compositionality: Components developed and tested indepently, but they are not fully independent

Detection and resolution challenging: 
* Analysis of requirements (formal methods or inspection), e.g., overlapping preconditions, shared resources
* Enforcing isolation (often not feasible)
* Testing, testing, testing at the system level

<!-- references -->
Recommended reading: Nhlabatsi, Armstrong, Robin Laney, and Bashar Nuseibeh. [Feature interaction: The security threat from within software systems](https://www.nii.ac.jp/pi/n5/5_75.pdf). Progress in Informatics 5 (2008): 75-89.

----
## Nonlocal effects in ML systems?

<!-- discussion -->

Notes: Improvement in prediction quality in one component does not always increase overall system performance. Have both local model quality tests and
global system performance measures.

Examples: Slower but more accurate face recognition not improving user experience for unlocking smart phone.

Example: Chaining of models -- second model (language interpretation) trained on output of the first (parts of speech tagging) depends on specific artifacts and biases

Example: More accurate model for common use cases, but more susceptable to gaming of the model (adversarial learning)

----
## Recall: Sensitivity Analysis

* How sensitive is a model to its inputs, especially inputs that come from other models

![](intrude-sa.png)

----
## Recall: Beta Tests and Testing in Production

* Test the full system in a realistic setting
* Collect telemetry to identify bugs

![](ab-button.png)

----
## Excursion: Feedback Loops

```mermaid
graph TD
a[Model] -->|Influences| b[User behavior]
b -->|Learned from| a
```

**Examples?**

----
## Feedback Loop Examples

* Users adjust how they speak to Alexa, Alexa trained on ...
* Model to suggest popular products makes products popular
* Model to predict crime influences where crime is enforced, influencing crime statistics
* Criminals adjust to fraud detection model, model adjusts to recent crime

----
## How to detect feedback loops?

<!-- discussion -->

More in a later lecture





---
# Testing for Robustness

----
## Test Error Handling


```java
@Test void test() {
    DataTable data = new DataTable();
    try {
        Model m = learn(data);
        Assert.fail();
    } catch (NoDataException e) { /* correctly thrown */ }
}
```

Notes: Code to test that the right exception is thrown


----
## Test "Faulty" Environment


```java
DataTable getData(Stream stream, DataCleaner cleaner) { ... }

@Test void test() {
    Stream testStream = new Stream() {
        public void connect() { 
            throw new IOException("cannot establish connection")
        }
        public String getNext() {
            throw new IOException("connection dropped")
        }
    }
    try {
        DataTable output = getData(testStream, new DefaultCleaner());
        Assert.fail();
    } catch (DataTableException e) { /* correctly handled */ }
}
```

Notes: Use stubs for testing


----
## Test Local Error Handling (Modular Protection)

```java
@Test void test() {
    Stream testStream = new Stream() {
        int idx = 0;
        public void connect() { 
            if (++idx < 3)
                throw new IOException("cannot establish connection")
        }
        public String getNext() { ... }
    }
    DataLoader loader = new DataLoader(testStream, new DefaultCleaner());
    ModelBuilder model = new ModelBuilder(loader, ...);
    // assume all exceptions are handled correctly internally
    assert(model.accuracy > .91)
}
```

Notes: Test that errors are correctly handled within a module and do not leak


----
## Test Monitoring

* Inject/simulate faulty behavior
* Mock out notification service used by monitoring
* Assert notification

```java
class MyNotificationService extends NotificationService {
    public boolean receivedNotification = false;
    public void sendNotification(String msg) { receivedNotification = true; }
}
@Test void test() {
    Server s = getServer();
    MyNotificationService n = new MyNotificationService();
    Monitor m = new Monitor(s, n);
    s.stop();
    s.request();
    s.request();
    wait();
    assert(n.receivedNotification);
}
```

----
## Test Monitoring in Production

* Like fire drills
* Manual tests in production, repeat regularly
* Actually take down service or trigger wrong signal to monitor

----
## Chaos Testing

![Chaos Monkey](simiamarmy.jpg)


<!-- references -->

http://principlesofchaos.org

Notes: Chaos Engineering is the discipline of experimenting on a distributed system in order to build confidence in the system’s capability to withstand turbulent conditions in production. Pioneered at Netflix

----
## Chaos Testing Argument

* Distributed systems are simply too complex to comprehensively predict
* -> experiment on our systems to learn how they will behave in the presence of faults
* Base corrective actions on experimental results because they reflect real risks and actual events
*
* Experimentation != testing -- Observe behavior rather then expect specific results
* Simulate real-world problem in production (e.g., take down server, inject latency)
* *Minimize blast radius:* Contain experiment scope

----
## Netflix's Simian Army

Chaos Monkey: randomly disable production instances

Latency Monkey: induces artificial delays in our RESTful client-server communication layer

Conformity Monkey: finds instances that don’t adhere to best-practices and shuts them down

Doctor Monkey: monitors other external signs of health to detect unhealthy instances

Janitor Monkey: ensures that our cloud environment is running free of clutter and waste

Security Monkey: finds security violations or vulnerabilities, and terminates the offending instances

10–18 Monkey: detects problems in instances serving customers in multiple geographic regions

Chaos Gorilla is similar to Chaos Monkey, but simulates an outage of an entire Amazon availability zone.




----
## Chaos Toolkit

* Infrastructure for chaos experiments
* Driver for various infrastructure and failure cases
* Domain specific language for experiment definitions

```js
{
    "version": "1.0.0",
    "title": "What is the impact of an expired certificate on our application chain?",
    "description": "If a certificate expires, we should gracefully deal with the issue.",
    "tags": ["tls"],
    "steady-state-hypothesis": {
        "title": "Application responds",
        "probes": [
            {
                "type": "probe",
                "name": "the-astre-service-must-be-running",
                "tolerance": true,
                "provider": {
                    "type": "python",
                    "module": "os.path",
                    "func": "exists",
                    "arguments": {
                        "path": "astre.pid"
                    }
                }
            },
            {
                "type": "probe",
                "name": "the-sunset-service-must-be-running",
                "tolerance": true,
                "provider": {
                    "type": "python",
                    "module": "os.path",
                    "func": "exists",
                    "arguments": {
                        "path": "sunset.pid"
                    }
                }
            },
            {
                "type": "probe",
                "name": "we-can-request-sunset",
                "tolerance": 200,
                "provider": {
                    "type": "http",
                    "timeout": 3,
                    "verify_tls": false,
                    "url": "https://localhost:8443/city/Paris"
                }
            }
        ]
    },
    "method": [
        {
            "type": "action",
            "name": "swap-to-expired-cert",
            "provider": {
                "type": "process",
                "path": "cp",
                "arguments": "expired-cert.pem cert.pem"
            }
        },
        {
            "type": "probe",
            "name": "read-tls-cert-expiry-date",
            "provider": {
                "type": "process",
                "path": "openssl",
                "arguments": "x509 -enddate -noout -in cert.pem"
            }
        },
        {
            "type": "action",
            "name": "restart-astre-service-to-pick-up-certificate",
            "provider": {
                "type": "process",
                "path": "pkill",
                "arguments": "--echo -HUP -F astre.pid"
            }
        },
        {
            "type": "action",
            "name": "restart-sunset-service-to-pick-up-certificate",
            "provider": {
                "type": "process",
                "path": "pkill",
                "arguments": "--echo -HUP -F sunset.pid"
            },
            "pauses": {
                "after": 1
            }
        }
    ],
    "rollbacks": [
        {
            "type": "action",
            "name": "swap-to-vald-cert",
            "provider": {
                "type": "process",
                "path": "cp",
                "arguments": "valid-cert.pem cert.pem"
            }
        },
        {
            "ref": "restart-astre-service-to-pick-up-certificate"
        },
        {
            "ref": "restart-sunset-service-to-pick-up-certificate"
        }
    ]
}
```


<!-- references -->
http://principlesofchaos.org, https://github.com/chaostoolkit, https://github.com/Netflix/SimianArmy

----
## Chaos Experiments in ML?

<!-- discussion -->



 
---
# Summary

* Unit testing vs integration testing vs system testing
* Testing all parts of the ML-pipleline
* Test automation with *Continuous Integration* tools
* Interactions: local improvements may degrade overall system performance
* Testing for robustness and chaos engineering