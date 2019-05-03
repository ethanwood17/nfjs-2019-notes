## NFJS 2019 Notes

### Essential Spring Boot
***Craig Walls***

Newsletter - tinyletter.com/habuma
Github - github.com/habuma

Spring boot CLI provides a way to do command line app development in Groovy, deployed in something like CloudFoundry 
Spring provides 4 main simplifications/advantages
- CLI
- Autoconfiguration
- dependency management (Spring Boot starter dependencies)
- Runtime insight through Actuator

Using a Jar file is amenable to deploying in a cloud context because it doesn't delegate configuration responsibility to the application server. War files delegate to the provided configuration in the application server, making configuration sometimes problematic, with no gaurantee that two application servers are configured the same. 

"Make Jar, not War"

Spring Initializr makes starting a project easy by allowing you to add dependencies and set your packaging scheme, and then creating a project with the appropriate file structure and settings. 

Spring Tool Suite (STS) gives access to the Spring Initializr from your IDE. 

Tweaking the Initializr to add your own internal dependencies is a possibility. Be cool to add the UI dependencies to a Spring Initializr menu in Eclipse/IntelliJ. 

If you already have a parent POM (which we do) and you can't use the Spring starter parent POM, you can just move the Spring Boot parent groupId, artifactId and version down and make them a dependency in your project, giving them a scope of import (the scope is important). Alternatively, make the Spring Boot parent POM the parent POM of your organization's parent POM. Parent POM-ception. 

To override the imported Spring version, add a maven property <spring.version> and specify the version you want. Likewise for any other imported dependency.

If you're going to start using Jar files, it's important to move away from JSPs because they're hard to work with in Jar files. Also, they're just not as good as Thymeleaf or Freemarker or one of the other templating systems.

Spring will automatically detect the appropriate JDBC driver based on the format of the JDBC url. Usually you won't have to specify a driver class unless there's ambiguity. 

Some useful Spring configuration properties are: 
server.port
spring.datasource.url
spring.application.name (does nothing until you deploy Spring app in a container)

YAML for configuration files makes a lot of sense. The structure looks nicer and is easier to read. **Find out how to setup YAML file configuration autocomplete in Intellij**

If you wanted to, you could annotate the Spring main class with every annotation you need for your project, and then include all your code in one file. Bad practice, but a possibility. 

If you're using Lombok, you can delete your args contstructor in your Spring Controller class. Just include your Bean (no @Autowired) and add the @RequiredArgsConstructor annotation from Lombok to the class. HOWEVER, **only works from Spring 4.x**. So probably won't work for UI. 

```
@RestController
@RequiredArgsConstructor
public class BooksController {
	
	//autowiring is implicitly added by Spring
	private BooksRepository repo; 

}
```
checkout `jq` package for formatting JSON on the commandline

### Continuations and Fibers - The New Frontier for Java
***Venkat Subramaniam***

**Benefits of Single-Threaded Applications**
- Easier to work with
- People smile when they come to work

Threads as performance enhancers have some fundamental limitations. Different jobs have different profiles, like whether they're computation intensive or IO intensive, meaning that one needs to adopt a threading strategy unique to the profile of the job. That takes skill and time to tune. 
Also, threads are fundamentally blocking. Resources must be released in order to surrender them to a waiting thread, which creates a lot of context switching. 

Threads have to wait until they have resources available before doing work on a task. Instead, it would be nice if a thread changed what it was working on when it can't work on something. This is the concept of ***coroutines***. 

Coroutines are stateful subroutines. Subroutines are functions, pure and simple. Coroutines are simply subroutines that are aware of context. 

A lambda function is a pure function that is stateless. A closure is a lambda which carries immutable state. Using a function that returns a closure with a partial result, we can implement a coroutine. 

What's the difference between parallel and concurrent. Parallelism is when two processes execute at the same time (two people walking together). Concurrency is when two processes execute together, but pass information back and forth (two people having a conversation). 

You can accomplish this sort of thing already with Kotlin

```
suspend String task1() {
	println("foo")
	yield()
	println("bar")
}

main() {
	launch { task1() }
}
```

Java is implementing this workflow through ***continuations***. 

```
import java.lang.Continuation.*

public class Sample {
	public static void doWork(ContinutationScope scope){

	System.out.println("entering do work");
	yield();
	System.out.println("step 1");
	yield();
	System.out.println("step 2")
}

	public static void main(STring[] args){
	var scope = new ContinuationScope("sample");

	Continuation continuation = new Continuation(scope, () -> doWork());

	while(!continuation.isDone()){
		System.out.println("in the loop");
	}
		System.out.println("OK");
	}
}
```

***Fibers*** are essentially lightweight threads. 
Fibers are managed by the JVM and not by the operating system.
When a fiber blocks (waits for a resource), that specific task waits, but not the underlying thread. 

```
public class Sample {
	public static void doWork(){
		System.out.println("Do work");
	}

	public static void main(String[] args) {
		var fiber = Fiber.schedule(() -> doWork());

		fiber.join();
	}
}
```

According to Venkat, **threads are going to go away**. Let's see if that happens. Pretty big shift. 

For IO operations, fibers are great. For computation intensive processes, threads are the way to go. That brings to mind the JDBC. JDBC is a blocking system. Non blocking implementation with fibers? Big if true. 

This feature is seamless in Java. Venkat thinks that's dangerous. There's nothing restricting you from calling a non-fiber safe function in a fiber context. Rather than throwing an error, the JVM will execute the calls in batches equal to the number of cores on your machine. That being the case, it's important to be careful and not use fibers unless you know you won't be calling blocking, non-fiber safe functions. 

This is great for a cloud application, because this decreases the processing/memory requirements needed to run a multithreaded program on a cloud provider's hardware. For instance, an app running on an EC2 instance will work more efficiently, reducing the cost of operation for you. 

http://www.agiledeveloper.com/downloads.html
