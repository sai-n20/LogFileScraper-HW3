# Homework 3

## Name: Sai Nadkarni
## UIN: 672756678
## NetID: snadka2
## Instructions and documentation:

### Detiled description in docs folder

### Explanation video:

YouTube- https://youtu.be/j89HyGQGWNI

Includes explanation of code as well.

### The goal of this homework is for students to gain experience with solving a distributed computational problem using cloud computing technologies by designing and implementing a RESTful service and a lambda function that are accessed from clients using gRPC.
### Grade: 8%
#### This Git repo contains the description of the third and final homework that uses this implementation of a log file generator in Scala. Students should clone this repo using the command ```git clone git@github.com:0x1DOCD00D/LogFileGenerator.git```.

## Preliminaries
As part of the previous homework assignment you learned to create and manage your Git repository, create your application in Scala, create tests using widely popular Scalatest framework, and to configure your big data processing environments. Doing this homework is essential for successful completion of the rest of this course, since the course project will depend on this homework: create randomly generated log files that represent big data, parse and analyze them, change their content and structure and process them in the cloud using a big data analysis framework called Spark.

First things first, if you haven't done so, you must create your account at either [BitBucket](https://bitbucket.org/) or [Github](https://github.com/), which are Git repo management systems. Please make sure that you write your name in your README.md in your repo as it is specified on the class roster. Since it is a large class, please use your UIC email address for communications and avoid emails from other accounts like funnybunny2000@gmail.com.

Next, if you haven't done so, you will install [IntelliJ](https://www.jetbrains.com/student/) with your academic license, the JDK, the Scala runtime and the IntelliJ Scala plugin and the [Simple Build Toolkit (SBT)](https://www.scala-sbt.org/1.x/docs/index.html) and make sure that you can create, compile, and run Java and Scala programs. Please make sure that you can run [various Java tools from your chosen JDK between versions 8 and 16](https://docs.oracle.com/en/java/javase/index.html).

In this and all consecutive homeworks and in the course project you will use logging and configuration management frameworks. You will comment your code extensively and supply logging statements at different logging levels (e.g., TRACE, INFO, WARN, ERROR) to record information at some salient points in the executions of your programs. All input and configuration variables must be supplied through configuration files -- hardcoding these values in the source code is prohibited and will be punished by taking a large percentage of points from your total grade! You are expected to use [Logback](https://logback.qos.ch/) and [SLFL4J](https://www.slf4j.org/) for logging and [Typesafe Conguration Library](https://github.com/lightbend/config) for managing configuration files. These and other libraries should be imported into your project using your script [build.sbt](https://www.scala-sbt.org). These libraries and frameworks are widely used in the industry, so learning them is the time well spent to improve your resumes. Preferably, you should create your developer account for $30 per month to enjoy the full range of AWS services.

From the implementation of the log generator you can see how to use Scala to create a fully pure functional (not imperative) implementation. As you see from the StackOverflow survey, knowledge of Scala is highly paid and in great demand, and it is expected that you pick it relatively fast, especially since it is tightly integrated with Java. I recommend using the book on Programming in Scala Fourth and Fifth Editions by Martin Odersky et al. You can obtain this book using the academic subscription on Safari Books Online. There are many other books and resources available on the Internet to learn Scala. Those who know more about functional programming can use the book on Functional Programming in Scala published on Sep 14, 2014 by Paul Chiusano and Runar Bjarnason.

When creating your RESTful/gRPC in Scala, you should avoid using **var**s and while/for loops that iterate over collections using [induction variables](https://en.wikipedia.org/wiki/Induction_variable). Instead, you should learn to use collection methods **map**, **flatMap**, **foreach**, **filter** and many others with lambda functions, which make your code linear and easy to understand. Also, avoid mutable variables that expose the internal states of your modules at all cost. Points will be deducted for having unreasonable **var**s and inductive variable loops without explanation why mutation is needed in your code unless it is confined to method scopes - you can always do without it.

## Overview of the Log File Generator
In this homework, you will create a distributed program for locating requested records in the log files that are generated using this project that you cloned. Once you cd into the cloned project directory you can build using ```sbt clean compile``` then run tests with ```sbt test``` and then run the project with ```sbt run```. Currently, the settings in ```application.conf``` allow you to generate a random log file with 100 entries that will be put in a directory named ***log*** under the root project directory. Alternatively, you can import this project into IntelliJ and run it from within the IDE. 

Each entry in the log dataset describes a fictitios log message, which contains the time of the entry, the logging context name, the message level (i.e., INFO, WARN, DEBUG or ERROR), the name of the logging module and the message itself. The size of the log file can be controlled by setting the maximum number of log messages or the duration of the log generator run in ```application.conf```. Students can experiment with smaller log files when debugging their programs, but they should create large enough log files for this homework assignment. Each log entry is independent from the other one in that it can be processed without synchronizing with processing some other entries. Using configuration parameter TimePeriod you can specify a range of timeout values, e.g., ```[1000, 10000]``` so that each log message will be created with some random delay within one to ten seconds.

Consider the following entries in the dataset. The first entry is generated at time 09:58:55.569 followed by the second entry generated at time 09:58:55.881. The sixth entry is of type ERROR that has a smaller likelihood range in ```application.conf```. Depending on the value of the configuration parameter ```Frequency``` the regular expression specified in configuration parameter ```Pattern``` is used to create instances of this pattern which are inserted into the generated log messages. It is imperative that students read comments in this configuration file and experiment with different configuration parameter settings to see how the content of the generated log file changes.   
```
09:58:55.569 [main] INFO  GenerateLogData$ - Log data generator started...
09:58:55.881 [scala-execution-context-global-17] INFO  GenerateLogData$ - NL8Q%rvl,RBHq@|XR2U&k>"SXwcyB#iv
09:58:55.928 [scala-execution-context-global-17] WARN  Generation.Parameters$ - =5$YcP!s@h
09:59:30.849 [scala-execution-context-global-17] INFO  Generation.Parameters$ - V<Z~#Ws"WNJ:[d?+dRpaIFp23"1_oKn;Qd,>
09:59:30.867 [scala-execution-context-global-17] INFO  Generation.Parameters$ - 3FNgL<)k7+c+8yQ"3m*e#!)HK[['z+-an/Uw?J'|[<w&kbtM
09:59:30.876 [scala-execution-context-global-17] ERROR  Generation.Parameters$ - +5l}CAK:}q])
09:59:30.891 [scala-execution-context-global-17] INFO  Generation.Parameters$ - Mv8)!{uuaD3%<m.VO/[pfHLS&eIBmKx~(6
```

Your job is to create an algorithm of at most O(log N) complexity for locating messages from a set of N messages that contain some designated pattern-matching string that are timestamped within some delta time interval from the requested time. That is, the input to your client program includes two parameters: the time, T and some time interval, dT, e.g., T = 9:58:55 and dT = 00:01:00. Your algorithm will determine if messages exist in the log that are timestamped within the time interval 9:57:55 and 9:59:55. Next, your algorithm will determine which of these messages contain strings that match the pattern specified in configuration parameter ```Pattern```. Since log files can be very large iterating through messages one by one has O(N) complexity and it is not acceptable. You will need to use a cleverer techniques like binary search or you can modify the code of the log generator to create some kind of a hash table to speed up the search. 

Furthermore, there is no point to search a large file for log messages if the desired time entry is not in this file. For that purpose you will create a lambda function that will check to see if a log file contains messages within a given time interval and it will return  a boolean status to the client to specify the presence of the desired time interval in the log file. We assume that all messages in a log file are created in strictly sequential time order, i.e., the timestamp for the first message is always smaller than the one for the following message. 

As before, this homework script is written using a retroscripting technique, in which the homework outlines are generally and loosely drawn, and the individual students improvise to create the implementation that fits their refined objectives. In doing so, students are expected to stay within the basic requirements of the homework and they are free to experiments. Asking questions is important, so please ask away at MS Teams!

In this homework, you will create and deploy your lambda functions on [AWS Lambda](https://aws.amazon.com/lambda/) and you will write a client program to invoke these lambda functions using [gRPC](https://grpc.io/) and independently a different client program to invoke the same lambda functions by using the [AWS API Gateway](https://aws.amazon.com/api-gateway/) with the requests GET and POST and DELETE.

Doing this homework enables students to put their theoretical knowledge about gRPC and REST architectural style on a firm footing in the context of creating and using lambda functions deployed in the cloud.

## Functionality
Your homework assignment consists of two interlocked parts: first, create a client program that uses gRPC to invoke a lambda function deployed on AWS to determine if the desired timestamp is in the log file and second, to  a client program and the corresponding lambda function that use the REST methods (e.g., GET or POST) to interact. While you are free to determine how your lambda function works, there is a small subset of mandatory functionality that you must implement: given the input of time stamp and time interval determine if the log files in some bucket that are being constantly updated with new log messages contain messages in the given time interval from the designated input time stamp and to return a MD5-generated hash code from these messages or some 400-level HTTP client response to designate that log files do not contain any messages in the given time interval.

You will deploy an instance of the log file generation program on EC2 and configure it to run for some period of time producing and storing log messages into log file in some S3-allocated storage. If you need to modify the generator for this purpose please go ahead and fork the repo and make appropriate changes. You can create additional projects for the RESTful service and its client and extend the sbt script to include these subprojects as dependencies.

The starting point is to follow the guide on [AWS Serverless Application Model (SAM)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-quick-start.html). Once you follow the steps of the tutorial, you will be able to invoke a lambda function via the AWS API Gateway.

Next, you will learn how to create a [gRPC](https://grpc.io/) client program. I find this [tutorial on gRPC on HTTP/2](https://www.cncf.io/blog/2018/08/31/grpc-on-http-2-engineering-a-robust-high-performance-protocol/) very well written by Jean de Klerk, Developer Program Engineer at Google.

After that you will learn about [AWS API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) and determine how to use it to create RESTful API for your implementation of the lambda function.

A guide to keep you on the right path is the [blog entry](https://blog.coinbase.com/grpc-to-aws-lambda-is-it-possible-4b29a9171d7f) that describes the process of using gRPC for invoking AWS lambda function in Go.

Excellent [guide how to create a REST service with AWS Lambda](https://blog.sourcerer.io/full-guide-to-developing-rest-apis-with-aws-api-gateway-and-aws-lambda-d254729d6992) includes instructions on how to set up and configure AWS Lambda.

To implement a RESTful service for retrieving log messages you can use one of the popular frameworks: [Play](https://www.baeldung.com/scala/play-rest-api) or [Finch/Finagle](https://www.baeldung.com/scala/finch-rest-apis) or [Akka HTTP](https://vikasontech.github.io/post/scala-rest-api-with-akka-http/) or [Scalatra](http://honstain.com/rest-in-a-scalatra-service/). There is a [discussion thread on Reddit](https://www.reddit.com/r/scala/comments/ifz8ji/what_framework_should_i_use_to_build_a_rest_api/) about which framework software engineers prefer to create and maintain RESTful services. Personally, I like Akka HTTP but students are free to experiment with more than one framework. On a side note it would be very suspicious for your TA and me to see very similar implementations of the service using the same framework that came from different submissions by different students.

Regarding ***testing client programs to test your RESTful services*** you can implement them as a [Postman](https://www.postman.com/) project or as a [curl](https://curl.se/) command in a shell script or you can write a Scala program that uses [Apache HTTP client library](https://hc.apache.org/httpcomponents-client-5.1.x/). 

Next, after creating and testing your programs locally, you will deploy it and run it on the AWS. You will produce a short movie that documents all steps of the deployment and execution of your program with your narration and you will upload this movie to [youtube](http://www.youtube.com) and as before you will submit a link to your movie as part of your submission in the README.md file. To produce a movie, you may use an academic version of [Camtasia](https://www.techsmith.com/video-editor.html) or some other cheap/free screen capture technology from the UIC webstore or an application for a movie capture of your choice. The captured web browser content should show your login name in the upper right corner of the AWS application and you should introduce yourself in the beginning of the movie speaking into the camera.

## Baseline Submission
Your baseline project submission should include your implementation, a conceptual explanation in the document or in the comments in the source code of how your algorithm and its implementation work to solve the problem, and the documentation that describe the build and runtime process, to be considered for grading. Your project submission should include all your source code as well as non-code artifacts (e.g., configuration files), your project should be buildable using the SBT, and your documentation must specify how you partitioned the data and what input/outputs are.

## Collaboration
You can post questions and replies, statements, comments, discussion, etc. on Teams using the corresponding channel. For this homework, feel free to share your ideas, mistakes, code fragments, commands from scripts, and some of your technical solutions with the rest of the class, and you can ask and advise others using Teams on where resources and sample programs can be found on the Internet, how to resolve dependencies and configuration issues. When posting question and answers on Teams, please make sure that you selected the appropriate channel, to ensure that all discussion threads can be easily located. Active participants and problem solvers will receive bonuses from the big brother :-) who is watching your exchanges (i.e., your class instructor and your TA). However, *you must not describe your algorithm or specific details related to how you constructed your system!*

## Git logistics
**This is an individual homework.** If you read this description it means that you located the [Github repo for this homework](https://github.com/0x1DOCD00D/LogFileGenerator). Please remember to grant a read access to your repository to your TA and your instructor. In future, for the team homeworks and the course project, you should grant the write access to your forkmates, but NOT for this homework. You can commit and push your code as many times as you want. Your code will not be visible and it should not be visible to other students (except for your forkmates for a team project, but not for this homework). Announcing a link to your public repo for this homework or inviting other students to join your fork for an individual homework before the submission deadline will result in losing your grade. For grading, only the latest commit timed before the deadline will be considered. **If your first commit will be pushed after the deadline, your grade for the homework will be zero**. For those of you who struggle with the Git, I recommend a book by Ryan Hodson on Ry's Git Tutorial. The other book called Pro Git is written by Scott Chacon and Ben Straub and published by Apress and it is [freely available](https://git-scm.com/book/en/v2/). There are multiple videos on youtube that go into details of the Git organization and use.

Please follow this naming convention to designate your authorship while submitting your work in README.md: "Firstname Lastname" without quotes, where you specify your first and last names **exactly as you are registered with the University system**, so that we can easily recognize your submission. I repeat, make sure that you will give both your TA and the course instructor the read/write access to your *private forked repository* so that we can leave the file feedback.txt with the explanation of the grade assigned to your homework.

## Discussions and submission
As it is mentioned above, you can post questions and replies, statements, comments, discussion, etc. on Teams. Remember that you cannot share your code and your solutions privately, but you can ask and advise others using Teams and StackOverflow or some other developer networks where resources and sample programs can be found on the Internet, how to resolve dependencies and configuration issues. Yet, your implementation should be your own and you cannot share it. Alternatively, you cannot copy and paste someone else's implementation and put your name on it. Your submissions will be checked for plagiarism. **Copying code from your classmates or from some sites on the Internet will result in severe academic penalties up to the termination of your enrollment in the University**.


## Submission deadline and logistics
Wednesday, November 3, 2021 at 11:59PM CST via email to the instructor and your TA that lists your name and the link to your repository. Your submission repo will include the code for the program, your documentation with instructions and detailed explanations on how to assemble and deploy your program along with the results of your program execution, the link to the video and a document that explains these results based on the characteristics and the configuration parameters of your log generator, and what the limitations of your implementation are. Again, do not forget, please make sure that you will give both your TAs and your instructor the read access to your private repository. Your code should compile and run from the command line using the commands **sbt clean compile test** and **sbt clean compile run**. Also, you project should be IntelliJ friendly, i.e., your graders should be able to import your code into IntelliJ and run from there. Use .gitignore to exlude files that should not be pushed into the repo.


## Evaluation criteria
- the maximum grade for this homework is 8%. Points are subtracted from this maximum grade: for example, saying that 2% is lost if some requirement is not completed means that the resulting grade will be 8%-2% => 6%; if the core homework functionality does not work or it is not implemented as specified in your documentation, your grade will be zero;
- only some basic RESTful service examples from some repos are given and nothing else is done: zero grade;
- having less than five unit and/or integration scalatests: up to 5% lost;
- missing comments and explanations from your program: up to 5% lost;
- logging is not used in your programs: up to 3% lost;
- hardcoding the input values in the source code instead of using the suggested configuration libraries: up to 4% lost;
- for each used *var* for heap-based shared variables or mutable collections: 0.2% lost;
- for each used *while* or *for* or other loops with induction variables to iterate over a collection: 0.2% lost;
- no instructions in README.md on how to install and run your program: up to 5% lost;
- the program crashes without completing the core functionality: up to 6% lost;
- the documentation exists but it is insufficient to understand your program design and models and how you assembled and deployed all components of your solution: up to 5% lost;
- the minimum grade for this homework cannot be less than zero.

That's it, folks!
