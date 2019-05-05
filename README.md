# NFJS 2019 Notes

## Essential Spring Boot
***Craig Walls***

Newsletter: tinyletter.com/habuma
Github: github.com/habuma

Spring boot CLI provides a way to do command line app development in Groovy, deployed in something like CloudFoundry 
Spring provides 4 main simplifications/advantages
- CLI
- Autoconfiguration
- dependency management (Spring Boot starter dependencies)
- Runtime insight through Actuator

Using a Jar file is amenable to deploying in a cloud context because it doesn't delegate configuration responsibility to the application server. War files delegate to the provided configuration in the application server, making configuration sometimes problematic, with no gaurantee that two application servers are configured the same. 

> Make Jar, not War

Spring Initializr makes starting a project easy by allowing you to add dependencies and set your packaging scheme, and then creating a project with the appropriate file structure and settings. 

Spring Tool Suite (STS) gives access to the Spring Initializr from your IDE. 

Tweaking the Initializr to add your own internal dependencies is a possibility. Be cool to add the UI dependencies to a Spring Initializr menu in Eclipse/IntelliJ. 

If you already have a parent POM (which we do) and you can't use the Spring starter parent POM, you can just move the Spring Boot parent groupId, artifactId and version down and make them a dependency in your project, giving them a scope of import (the scope is important). Alternatively, make the Spring Boot parent POM the parent POM of your organization's parent POM. Parent POM-ception. 

To override the imported Spring version, add a maven property 
<spring.version></spring.version> 
and specify the version you want. Likewise for any other imported dependency.

If you're going to start using Jar files, it's important to move away from JSPs because they're hard to work with in Jar files. Also, they're just not as good as Thymeleaf or Freemarker or one of the other templating systems.

Spring will automatically detect the appropriate JDBC driver based on the format of the JDBC url. Usually you won't have to specify a driver class unless there's ambiguity. 

Some useful Spring configuration properties are: 
```
server.port
spring.datasource.url
spring.application.name 
```
(name does nothing until you deploy Spring app in a container)

YAML for configuration files makes a lot of sense. The structure looks nicer and is easier to read. **Find out how to setup YAML file configuration autocomplete in Intellij**

If you wanted to, you could annotate the Spring main class with every annotation you need for your project, and then include all your code in one file. Bad practice, but a possibility. 

If you're using Lombok, you can delete your args contstructor in your Spring Controller class. Just include your Bean (no @Autowired) and add the @RequiredArgsConstructor annotation from Lombok to the class. However, **only works from Spring 4.x**. So probably won't work for UI. 

```java
@RestController
@RequiredArgsConstructor
public class BooksController {
	
	//autowiring is implicitly added by Spring
	private BooksRepository repo; 

}
```
checkout `jq` package for formatting JSON on the commandline

## Continuations and Fibers - The New Frontier for Java
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

```java
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

```java
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

```java
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

## Migrating to Java Modules
***Venkat Subramaniam***

Introduced in Java 9. 
One person raised their hand when Venkat asked "Who still uses CORBA?" Jesus that poor poor guy. 

The motivation behind modules is make it easier to exclude dependencies I don't need. For instance, Swing and CORBA. Most people don't need them because they'll never, ever use them, but they're still in the JDK. 

> A guy told me the other day that modules weren't necessary. I asked how? He said 
> "Because I'm using Maven." I had to tell him the bad news, that Maven uses you.

To check the modules in a JDK, use `java --list-modules`

CORBA and JavaFX, among other modules, have been removed from Java 10. 9 introduced the module system, 10, 11 and 12 have been removing modules to shrink the JDK size.

Modules improve security because with old Java, it's possible for a class to access another class if it has the same package name. Modules work differently. 

Modules `require` other modules, but they `export` their packages. 

Java 9 is the most significant release of Java, because it is mandatory. If you want to use Java 9, you have to write modular code. It's not opt-in. 

To define a module, add a file called `module-info.java`. This file has to go in the root directory of your project, because when you build it has to be in the root directory of your Jar. 

Java won't make you use different module and package names, but you should

module-info.java in package First
```java 
module com.agiledeveloper.thefirst {
	exports com.agiledeveloper.first;
}
```

module-info.java in package Second
```java
module com.agiledeveloper.thesecond {
	requires com.agiledeveloper.first;
}


```

`jar -f output/mlib/first.jar -d`
Lists module contents as required, exported or contained. 

You can't access classes that are not exported by their module. You can get them as a Class object by using reflection, however, you cannot invoke them. 

However, you can add another command in the module-info.java file to allow reflective access to a class from a different module that doesn't export it, likeso: 

```java
module com.agiledeveloper.thesecond {
	requires com.agiledeveloper.first;
	opens com.agiledeveloper.stuff.MyHelper;
}
```

Any traditional jar running in the classpath is an ***unnamed module***. Any traditional jar running in the modulepath is an ***automatic module***. If you run a modular jar on the modulepath, it is an ***explicitely named module***. 

Split packages are a problem. That's where you have classes for the same package in different jars. A reasonable use for this is where you have tests and library classes in different jars, but call them the same package name. In that case, use `patch-modules.` Don't know how to do that right now, have to look it up later. 

Automatic modules can access unnamed modules, but unnamed modules cannot access automatic modules. If you're in the classpath, you have to stay there. If you're in the module path, you can access stuff in the classpath. It's a security feature. 

**Question to check at home:** if I want to upgrade to Java 9, can I add `module-info.java` files to my packages running in Java 8 without breaking anything? Will the `module-info.java` file break anything in Java 8? I doubt it would, but it would be good to get confirmation. In that case, one could upgrade one's packages and then upgrade to Java 9. 

If you want to do some work towards upgrading without actually upgrading yet, the bare minimum is to write a manifest and claim a name for your project. 

For instance, in the package `mypackage`, create a file `mypackage/whatever.txt` and add this property to it.

`automatic-module-name=com.agiledeveloper.mypackage;`

If you compile Java code that includes your old package, as long as it has a .txt or any other kind of file that specifies an automatic file name, Java will generate a MANIFEST.mf for you in your META-INF folder. 

Don't upgrade from Java 8 to 9 if you can help it. Upgrade all the way from 8 to 12, if possible. It'll be better in the long run. You may have to do more work right away, but it'll reduce the work overall. 

Upgrade from the top down because unnamed modules can't call explicit modules. So, if you upgrade some base library, **every** library or project that uses that also has to have an explicit module name. So, upgrade each of your top level projects first, then work your way down through the dependencies. Practically, that means UI should upgrade each of our web apps first, then start upgrading our internal libraries like account and all the others. 

Venkat advises skipping over LTS versions if you have the time and feel like it. The reason is that the LTS actually works differently than a lot of people think. Apparently, Oracle will support the LTS version AND all version above it up until the next LTS version for 3 years. So, 11 is an LTS version. You upgrade to 12, you get LTS for 3 years, at which point you have to upgrade to the next LTS version (14 or whatever). So really, you lose nothing by passing the LTS version. 

## Domain Driven Serverless Design
***Matt Stine*** 

**Note: This talk was somewhat interesting, but ultimately not very helpful.**

Heroku was one of the first places to fully flesh out the idea of an application platform, creating something called the 12 factor application (don't know what that is). 

Dotcloud was a company focused on PaaS, but they didn't takeoff so they pivoted and developed tooling to run in a containerized environment, and eventually rebranded to Docker. 

|Container Orchestrator| Application Platform | Serverless Functions |
|----------------------|----------------------|--------------------- |
|	Kubernets, etc	   |	Heroku, etc		  |		Lambda 			 |


As you go left to right, lower complexity, higher efficiency
As you go right to left, fewer constraints and more flexibility

Start out serverless, but if you need lower latency or high throughput (i.e. your function will be constantly executing), then migrate to a container. Otherwise, your serverless function will suffer from startup time cost and automatic shutoffs. 

A problem one can run into in the serverless environment is decomposing services into functions too eagerly, which is an antipattern. Not everything needs to be its own independent function. Each layer of decomposition increases latency and maintenance cost, so be mindful in your decomposition. If a function always executes relative to another, they should ***most likely*** be a single function. 

At some point on a cost graph, the serverless and PaaS costs meet and invert. So it's important to know the uptime and invocation of your serverless function, because if it's always on, you're probably better off putting your function in a container on a server. 

A natural candidate for a serverless method is a batch job. It needs to perform at high efficiency, but only rarely. 

For the future: how about systems that are able to connect, deploy, monitor and scale themselves? Basically, a system that monitors its own efficiency graphs, and self-allocates resources as needed to improve those metrics. Essentially, an EBS sort of system, but for everything. 

If you operate based on an event-driven architecture, your operations may be good candidates for serverless operations. If you don't operate based on events, maybe if you're doing constant data manipulation or controlling some piece of hardware, then serverless won't work well for you. 

Domain Driven Design seeks to separate different interpretations of entities or processes across different domains. In other words, DDD seeks to decouple the Shipping implementation of Order from the Payments implementation. A change to the way an Order is payed for should not affect how the payment is shipped. 

Look up **context mapping** and **bounded contexts**. 

If you directly aggregate multiple objects, you may inadvertently create very large aggregates that have very large bounded contexts, meaning a change to any of the items contained in that context could break something. 

An example: If you change the ID of Product, you've not only broken Line Item but also all Orders that include that. Instead, do the right side architecture. Product Id is an immutable data object, and other items reference it. If they need to change their value, they point to a different Product Id object, so they only change their reference, creating a bounded domain around themselves Product Id, excluding everything else. 

	Domain 								    		Domain 1		Domain 2

	Order 			  				 		Order             			
	  ^				  				 	  	  ^		  
	  |				  				 	      |		   	 					  
	Line <-- Product						 Line 		<--   Product   --> 	Product 
	Item 			  				 	     Item 	   	  		ID 				  


So he extolls the virtues of Hexagonal Architecture, which I don't know anything about. Seems kind of culty. 

Tons of graphics of architectures using AWS based on your domain requirements. For instance, if you need to be able to publish events based on input to your system, you'll follow one architecture. But if you don't need to do that, you'll follow a different one. 

## Understanding Kubernetes
***Jonathan Johnson***

The guy giving this talk look like Wallace.

It may not be very safe to base things on "functions" if you're worried about vendor lock in. At least with AWS, you have to set up serverless functions and configure them to the AWS platform. They are not mobile, whereas Kubernetes or some other containerization schema is mobile and largely vendor independent. 

Maybe functions are fine, but sparingly, and not for your entire set of processes. 

There are still things to worry about with serverless architectures, like efficiency, performance, cost, etc. They don't solve everything. 

Serverless functions have their background in the mobile development community. Because some mobile developers have to write small amounts of server based code that serve a resource, or respond to a request, they wanted to accomplish that with the minimal infrastructure footprint. So, cloud providers began developing the idea of serverless functions. 

Cloud Native Landscape Foundation
- https://landscape.cncf.io
- https://s.cncf.io

Good resource to find cloud resource providers. 

Serverless
- http://serverless.com

They're developing a CLI tool to deploy serverless resources, provider independent. 

He has tutorials on KataKoda about Kubernetes. ***Definitely want to give them a look.***
We did the one about Kubeless in the session today, followed by the one about OpenFaaS. 

Really the biggest takeaway from this talk is to go to KataKoda and do this guy's tutorials. They seem pretty solid. 

Kubernetes seems like a very extensible technology to control Docker instances. 

https://cd.foundation

Good place to see continuous deployment tools for Kubernetes 

**What is JenkinsX?**

KataKoda JavaJon for Kubernetes + Java tutorials

## Machine Learning: Overview
***Brian Sletten***

It's important to know what your data's like, and it's important to have well-formed data. In the 1980s, the highest starting salary by major was in Geography, because Michael Jordan studied that in college before joining the NBA. Not removing him as an outlier from the dataset, he skews the entire set. 

Prior to starting a machine learning program, it's important to perform some exploratory data analysis. The idea is to get the warp and warf of the data in order to match your analysis correctly. 

You don't want to just memorize data about objects. It's important to know what data points are important. "Years of experience" is probably a more important data point for predicting your salary than "Name" is. Collecting and analyzing too many data points is called **overfitting**. 

Brian doesn't worry about the existential threat machine learning and data aggregation pose. Rather, he worries that the disruption of established fields will create loss of jobs and problems similar to that. 

Brian says in Watson there's no sentience, nothing fancy. It's just pattern matching. But isn't that all we are? Highly evolved pattern matchers. 

9/11 and the application of data analysis to terror prediction and security concerns were drivers of the current data analysis boom. Government funding for data analysis and statistical research produced a burst of activity. Also, the 2008 financial crisis caused some Wall Street analysis and data experts to jump ship and go to other companies. 

Really nice analogy between data aggregators today and Tycho Brahe/Johannes Kepler/Isaac Newton. Brahe never developed a solid explanatory theory around planetary motion, but his data was integral to Kepler's development of the elliptical theory of motion, which was essential to Newton's development of the general theory of motion. Right now, data is being aggregated but not explained adequately. We need Keplers and Newtons. 

**The Curse of Dimensionality**

http://cleverowl.uk/2016/02/06/curse-of-dimensionality-explained/

Some useful algorithms 

- linear regression
- k-nearest neighbor
- Naive Bayes
- Decision trees (random forest)
- k-means clustering

Brian Sletten ML gists, ML recs

## Machine Learning: Natural Language Processing
***Brian Sletten***

Natural Language Processing is highly context dependent. Asking Alexa or Siri to do something at home may be different than asking the same thing at work. So context is important and boundaries need to be established. 

A new field that hasn't been explored completely yet is generative NLP. Rather than answering questions, generative NLP produces language content. For instance, a system that produces documentation for a process or product, or a system that writes journlism articles. 

Stemming and lemmatization, look up. 

Different ways to parse documents

- Bag of words
	- Easy and quick, but loses context and word order
- N-gram model
	- splites documents into chunks of N words, which helps to preserve context
- Term frequency
	- frequency of words, often needs to be normalized to account for common words like prepositions, article adjectives, particles, etc. 
- Inverse Document frequency (look up)

Lonely words: Words are often treated as isolated indices. In other words, dog, cat and animal are all treated as distinct words, ignoring the relationships between them. 

t-SNE (t-stochastic network embedding) visualizations of word embeddings

In a graph of words by location in relation to other words, words will cluster in similar areas based on meaning. For instance, words about days of the week will often cluster together, but weekend and weekday words will be slightly separated. 

Vector space model, latent semantic analysis. 

Word2Vec

http://www.mccormickml.com

https://deeplearning4j.org/ 

seems like a good place to do some research and learning. 

https://wiki.dbpedia.org/

Using NLP and neural networks, it's possible to find connections between words. Training a model against a small set of sentences can find connections between the word "day" and the words "night", "week" and "year".

Book recommendation

> The Theory That Would Not Die

Look into Jupyter notebooks, pretty nice for doing coding examples. 

Some of the most salient uses of NLP would be for a company with inadequate resources, but lots of data. For instance, if your company has a small customer engagement team but many customers, and you get lots of comments or posts on social media, use NLP to analyse the most commonly used words in social media posts that are either compliments or complaints. That could help you find the most pressing problems and needs for improvement, as well as the parts that you're doing the best. 

## Get Go-ing
***Raju Gandhi***

main.go

```go
package main

import "fmt"

var name string
var names = [..]string {
	"Raju", 
	"Barrett"
}

func main() {
	//type inference through :=
	name := "raju"

	fmt.Println("Hello %s\n", name)

	//constant declaration, can't use type inference with const
	const name = "raju"
}
```

Go was built by Google to solve problems at Google. One of the big things they wanted to solve was ever increasing build times due to a huge codebase. Go seeks unchanging build times over arbitrarily large surfaces. 

The tooling surrounding Go is very mature. Go is mainly targeted at devops and systems development.

```go
var names = [5]string

var names = [..]string {
	"Raju", 
	"Barrett"
}

var names (
	var names = [5]string
	var names = [..]string {
		"Raju", 
		"Barrett"
	}
)
```

Every higher order Go type is eventually distilled down to a primitive, meaning null pointers are impossible unless you explicitely set a reference to `nil`; 

Go ships with a formatting tool, `go-fmt`, that automatically enforces proper Go syntax and formatting, such as whitespace (thus killing the inane tabs vs spaces debate), trailing commas in array declarations, etc.

Go distinguishes between public and private variables by using Uppercase or lowercase variable names. 

```go
var Name string //public
var name string //private
```

You have to declare the return type of a function, but it comes after the method signature. 

```go
func fullName(s string) string {
	return fmt.Sprintf("hello %s", s)
}
```

How to return multiple types from a function. If you return multiple types, you have to explicitely handle each type in your consumer code. I.E., if you return an error type, you have to decide how to consume the error in your code. 

```go
func fullName(s string) (string, error) {
	if len(s) == 0 {
		return "", errors.New("length == 0")
	}
	return fmt.Sprintf("hello %s", s), nil
}
```

Go also prohibits declaring unused variables. If a variable is not used in your code, the compiler will produce an error. 

If you declare return values with names in the signature of a function, it implicitely declares variables of those names and types in your code. 

```go
func fullName(s string) (msg string, e error) {
	msg = "thing" //don't have to use := because msg is already in scope as string type
}
```

Unused imports are a compiler error. Go automatically updates the import to the most recent HEAD version unless you've explicitely pinned it to a particular version. 

Only has one loop construct, the `for` loop, but you can wrangle these into while or forever loops if you want. The `while` keyword doesn't exist because it's silly. 

Arrays are value types in Go. Meaning, if I create an array and then assign its value to another array, I haven't changed the reference, I've created 2 copies of the array, doubling my memory. So, don't do that. 

Instead, Go has slices. Slices look very similar to arrays, but are slightly different. Note that the declaration below has no size (`[5]`) or the variable array declaration syntax (`[..]`)

```go
//slice declaration
languages := []string (
		"Go", 
		"Pascal",
	)
```

Go has maps, but has no sets. If you want to use sets, use maps with value keys. 

Go has structs rather than objects. Everything is statically typed and 0-valued, meaning no null reference, and no type inference or mismatch. 

```go
type Person struct {
	FirstName, LastName string
}
```

No inheritance in Go, which I would say is ***mostly*** nice, but maybe a little problematic at times. 

```go
import "fmt"

type Person struct {
	FirstName, LastName string
}

//function
func fullName(p Person) string {
	return fmt.Sprintf("%s %s", p.FirstName, p.LastName)
}

//method
func fullName(p Person) fullName() string {
	return fmt.Sprintf("%s %s", p.FirstName, p.LastName)
}

func main() {
	barrett := Person{"Barrett", "Ford"}
	fmt.Println(fullName(barrett))
	fmt.Println(barrett.fullName())
}
```

Go prevents the problem of Monkey-patching (adding bad functions onto objects or types) by restricting you to only adding methods to structs that you own. In other words, you can't take the `string` type and add your own methods to it, you can only consume it in a function. 

Interface declaration is clean and simple in Go. Also, as long as a struct "looks" like an interface, in other words it has the same values or functions, that struct fulfills the interface contract for that interface and doesn't have to explicitely declare that it implements it. 

```go
type Named interface {
	fullName() string
}
```

Go has pointers, and the syntax is basically the same as in C-family languages. 

Passing a pointer to a function
```go
func (p *Person) byRef() {
	
}
```
assigning a reference to a variable
```
ptr := &raju
```

Goroutines are kind of like threads. At least, they serve a similar purpose. The `go` keyword creates a goroutine.

```go
func main() {
	 simple()

	 wg.Add(1)
	 //using go keyword essentially creates a thread, but it's different
	 go routine()
	 wg.wait()
}
```

A few useful Go tools: 
- `Godoc`
- `Go fmt`
- `Golint`
- `Go get`
- `Goimports`

## Graal the Magnificent
***Daniel Hinojosa***

Graal is a universal virtual VM. It can run multiple languages in a VM context. Built towards Linux and MacOSX, no Windows version. Oracle based project. 

Supports all JVM languages naturally, also Rust, C/C++, Swift, R, Python, Ruby, and a few others. 

Graal has LLVM support for some langauges including Rust and C/C++

http://llvm.org

GraalVM is available in Dockerhub. Meant to be included in a Docker Compose file. 

To set up Graal, just make sure your `$JAVAHOME` variable is set then download and install the `tar.gz` file.

Graal can convert executable packages or files in other langauges (like JARs) into native images. 

```bash
clang -c -O1 -emit-llvm hello.c
```
The command above emits a file called `hello.bc`, which can be run with `lli hello.bc`

The only advantage of GraalVM over other Java VMs (past JDK8) is the ability to produce the native image. 

The real performance boost speedwise that Graal brings is replacing the JVM C2 JIT compiler written in C++ with a Java implementation, due to changes made in JDK 9 that allow swapping those out (JVMCI). 

The polyglot benefits of GraalVM is you can write code in multiple langauges and run them on the same JVM. For instance, hiring a Python ML engineer or data scientist, and having them write and run their Python code on the same VM as the Java code the rest of your shop writes. 

```java
Context context = Context.newBuilder().allowAllAccess(true).build();
String input = "{\"artist\" : \"Alexandra Savior\"}";
context.eval("ruby", "require json");
Value function = context.eval("ruby", "Proc.new{|x| JSON.parse(x)}");
```
Essentially, you can run Ruby in Java, or any of the other languages in any of the other languages. So, if you want to use some library in some other language (like numpy in Java), you just import and run it in Java. 

https://quarkus.io

The JDK version 9 improves performance substantially. It's roughly 50% faster. 

The Graal native image is really fast. To print a thousand primes in Scala, it cuts the time down from 1.3 seconds to 0.00 seconds. 

## WebAssembly
***Brian Slettern***

In the primordial soup of the early internet, there were several possible languages which web applications could be written in to run in a browser. However, it was impossible for web browsers to keep up with the changing features of every language. So, the pruning began. 

Nowadays, is a single language that most web development is centered around: JavaScript. However, JS is not an optimal language for most kinds of development, including web development. How to solve the issue of having only one reasonable language for web development, one that's suboptimal? 

**Enter WebAssembly.**

http://webassembly.org/demo

http://webassembly.org/docs/mvp


WebAssembly makes it possible to run desktop applications in the browser. Companies are interested in this because it allows them to program, test and build their application once and target all platforms, as long as those platforms are capable of running a WebAssembly enabled browser. That is significantly easier than writing multiple versions of the app targetting each system. 

JavaScript has certain limitations that makes it suboptimal. For instance, the lack of pointers. In order to mimic the functionality of pointers, some JavaScript engines use arrays of addresses that approximate the behavior of pointers. These limitations make JavaScript slower than it could be. 

So, the answer is to compile the actual JavaScript to a language that does have the functionality we need. 

WAST is a standard for WebAssembly files

WebAssembly works using a stack machine, just like classical assembly. Some current limitations: no threading, you can only pass numbers around (no higher order entities). However, none of this is a real problem because you probably won't ever be writing code in WebAssembly. Rather, you'll be ***compiling*** your code to WebAssembly. So who cares? 

IPFS (Interplanetary File System)

WebAssembly means whatever program you have compiled towards a single OS can run in a web browser. 

Blazor is a Microsoft thing that allows you to build .NET applications that can run in the browser. 

So the MVP for WebAssembly is already complete. Some of the things coming later are streaming compilation, a better compiler, 64 bit integers, etc. 

https://hacks.mozilla.org/2018/10/webassemblys-post-mvp-future/

The promise of WebAssembly is absolutely phenomenal. Running native applications in the browser. That's something that hasn't even really been anticipated or sought since the 90s with Java and other technologies that aimed for ubiquity. 