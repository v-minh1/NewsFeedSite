# Upgrade Progress

  ### ✅ Generate Upgrade Plan
  - [[View Log]](logs/1.generatePlan.log)

  ### ✅ Confirm Upgrade Plan
  - [[View Log]](logs/2.confirmPlan.log)

  ### ✅ Setup Development Environment
  - [[View Log]](logs/3.setupEnvironment.log)

  ### ✅ PreCheck
  - [[View Log]](logs/4.precheck.log)
  
  <details>
      <summary>[ click to toggle details ]</summary>
  
  - ###
    ### ✅ Precheck - Build project
    - [[View Log]](logs/4.1.precheck-buildProject.log)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Command
    `mvn clean test-compile -q -B -fn`
    </details>
  
    ### ✅ Precheck - Validate CVEs
    - [[View Log]](logs/4.2.precheck-validateCves.log)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### CVE issues
    - Dependency `junit:junit:4.11` has **1** known CVEs:
      - [CVE-2020-15250](https://github.com/advisories/GHSA-269g-pwp5-87pp): TemporaryFolder on unix-like systems does not limit access to created files
        - **Severity**: **MEDIUM**
        - **Details**: ### Vulnerability
          
          The JUnit4 test rule [TemporaryFolder](https://junit.org/junit4/javadoc/4.13/org/junit/rules/TemporaryFolder.html) contains a local information disclosure vulnerability.
          
          Example of vulnerable code:
          ```java
          public static class HasTempFolder {
              @Rule
              public TemporaryFolder folder = new TemporaryFolder();
          
              @Test
              public void testUsingTempFolder() throws IOException {
                  folder.getRoot(); // Previous file permissions: `drwxr-xr-x`; After fix:`drwx------`
                  File createdFile= folder.newFile("myfile.txt"); // unchanged/irrelevant file permissions
                  File createdFolder= folder.newFolder("subfolder"); // unchanged/irrelevant file permissions
                  // ...
              }
          }
          ```
          
          ### Impact
          
          On Unix like systems, the system's temporary directory is shared between all users on that system. Because of this, when files and directories are written into this directory they are, by default, readable by other users on that same system.
          
          This vulnerability **does not** allow other users to overwrite the contents of these directories or files. This is purely an information disclosure vulnerability.
          
          When analyzing the impact of this vulnerability, here are the important questions to ask:
          
          1. Do the JUnit tests write sensitive information, like API keys or passwords, into the temporary folder?
              - If yes, this vulnerability impacts you, but only if you also answer 'yes' to question 2.
              - If no, this vulnerability does not impact you.
          2. Do the JUnit tests ever execute in an environment where the OS has other untrusted users. 
              _This may apply in CI/CD environments but normally won't be 'yes' for personal developer machines._
              - If yes, and you answered 'yes' to question 1, this vulnerability impacts you.
              - If no, this vulnerability does not impact you.
          
          ### Patches
          
          Because certain JDK file system APIs were only added in JDK 1.7, this this fix is dependent upon the version of the JDK you are using.
           - Java 1.7 and higher users: this vulnerability is fixed in 4.13.1.
           - Java 1.6 and lower users: **no patch is available, you must use the workaround below.**
          
          ### Workarounds
          
          If you are unable to patch, or are stuck running on Java 1.6, specifying the `java.io.tmpdir` system environment variable to a directory that is exclusively owned by the executing user will fix this vulnerability.
          
          ### References
          - [CWE-200: Exposure of Sensitive Information to an Unauthorized Actor](https://cwe.mitre.org/data/definitions/200.html)
          - Fix commit https://github.com/junit-team/junit4/commit/610155b8c22138329f0723eec22521627dbc52ae
          
          #### Similar Vulnerabilities
           - Google Guava - https://github.com/google/guava/issues/4011
           - Apache Ant - https://nvd.nist.gov/vuln/detail/CVE-2020-1945
           - JetBrains Kotlin Compiler - https://nvd.nist.gov/vuln/detail/CVE-2020-15824
          
          ### For more information
          If you have any questions or comments about this advisory, please pen an issue in [junit-team/junit4](https://github.com/junit-team/junit4/issues).
    - Dependency `com.rabbitmq:amqp-client:5.16.0` has **1** known CVEs:
      - [CVE-2023-46120](https://github.com/advisories/GHSA-mm8h-8587-p46h): RabbitMQ Java client's Lack of Message Size Limitation leads to Remote DoS Attack
        - **Severity**: **MEDIUM**
        - **Details**: ### Summary
          `maxBodyLebgth` was not used when receiving Message objects.  Attackers could just send a very large Message causing a memory overflow and triggering an OOM Error.
          
          ### PoC
          #### RbbitMQ
          * Use RabbitMQ 3.11.16 as MQ and specify Message Body size 512M (here it only needs to be larger than the Consumer memory)
          * Start RabbitMQ
          #### Producer
          * Build a String of length 256M and send it to Consumer
          ```
          
          package org.springframework.amqp.helloworld; 
          
          import org.springframework.amqp.core.AmqpTemplate; 
          import org.springframework.context.ApplicationContext; 
          import org.springframework.context.annotation.AnnotationConfigApplicationContext; 
          
          public class Producer {
              public static void main(String[] args) {
                  ApplicationContext context = new AnnotationConfigApplicationContext(HelloWorldConfiguration.class);
                  AmqpTemplate amqpTemplate = context.getBean(AmqpTemplate.class); 
                  String s = "A";
                  for(int i=0;i<28;++i){
                      s = s + s;
                      System.out.println(i);
                  }
                  amqpTemplate.convertAndSend(s);
                  System.out.println("Send Finish");
              }
           }
          ```
          
          #### Consumer
          * First set the heap memory size to 128M
          * Read the message sent by the Producer from the MQ and print the length
          ```
          package org.springframework.amqp.helloworld;
          
          import org.springframework.amqp.core.AmqpTemplate;
          import org.springframework.amqp.core.Message;
          import org.springframework.context.ApplicationContext;
          import org.springframework.context.annotation.AnnotationConfigApplicationContext;
          
          public class Consumer {
              
              public static void main(String[] args) {
                  ApplicationContext context = new AnnotationConfigApplicationContext(HelloWorldConfiguration.class);
                  AmqpTemplate amqpTemplate = context.getBean(AmqpTemplate.class);
                  Object o = amqpTemplate.receiveAndConvert();
                  if(o != null){
                      String s = o.toString();
                      System.out.println("Received Length : " + s.length());
                  }else{
                      System.out.println("null");
                  }
              }
          }
          ```
          #### Results
          * Run the Producer first, then the Consumer
          * Consumer throws OOM Exception
          
          
          ### Impact
          Users of RabbitMQ may suffer from  DoS attacks from RabbitMQ Java client which will ultimately exhaust the memory of the consumer.
    </details>
  
    ### ✅ Precheck - Run tests
    - [[View Log]](logs/4.3.precheck-runTests.log)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Test result
    | Total | Passed | Failed | Skipped | Errors |
    |-------|--------|--------|---------|--------|
    | 0 | 0 | 0 | 0 | 0 |
    </details>
  </details>

  ### ✅ Upgrade project to use `Java 21`
  
  
  <details>
      <summary>[ click to toggle details ]</summary>
  
  - ###
    ### ✅ Upgrade using Agent
    - [[View Log]](logs/5.1.upgradeProjectUsingAgent.log)
    
    - 5 files changed, 25 insertions(+), 26 deletions(-)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Code changes
    - Upgrade Java to 21
      - Updated maven.compiler.source and maven.compiler.target from 17 to 21 in service/pom.xml
    - Migrate javax.* to jakarta.* namespace
      - Updated jakarta.websocket-api from javax.websocket-api:1.1 to jakarta.websocket-api:2.2.0
      - Updated jakarta.servlet-api from javax.servlet-api:3.1.0 to jakarta.servlet-api:6.1.0
      - Updated NewsWebSocket.java imports from javax.websocket.* to jakarta.websocket.*
      - Updated RabbitMQConsumer.java imports from javax.servlet.* to jakarta.servlet.*
      - Updated web.xml namespace from javax.jcp.org to jakarta.ee namespace (version 6.0)
    - Update Maven plugins for Java 21 support
      - Updated maven-surefire-plugin from 2.22.1 to 3.2.5
      - Updated jetty-maven-plugin from org.eclipse.jetty:jetty-maven-plugin:9.4.48 to org.eclipse.jetty.ee10:jetty-ee10-maven-plugin:12.0.21
    </details>
  
    ### ❗ Build Project
    - [[View Log]](logs/5.2.buildProject.log)
    
    - Build result: 0% Java files compiled
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Command
    `mvn clean test-compile -q -B -fn`
    
    #### Errors
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[13,24] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[16,24] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[22,25] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[28,25] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[34,43] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - package jakarta.websocket does not exist
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[3,1] package jakarta.websocket does not exist
      ```
    - cannot find symbol symbol:   class OnOpen location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[15,6] cannot find symbol
      [ERROR]   symbol:   class OnOpen
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class OnClose location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[21,6] cannot find symbol
      [ERROR]   symbol:   class OnClose
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class OnError location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[27,6] cannot find symbol
      [ERROR]   symbol:   class OnError
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class OnMessage location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[33,6] cannot find symbol
      [ERROR]   symbol:   class OnMessage
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    - cannot find symbol symbol:   class Session location: class com.example.websocket.NewsWebSocket
      ```
      [ERROR] /home/runner/work/NewsFeedSite/NewsFeedSite/service/src/main/java/com/example/websocket/NewsWebSocket.java:[43,18] cannot find symbol
      [ERROR]   symbol:   class Session
      [ERROR]   location: class com.example.websocket.NewsWebSocket
      ```
    </details>
  
    ### ✅ Fix Build Errors
    - [[View Log]](logs/5.3.fixBuildErrors.log)
    
    - 2 files changed, 10 insertions(+), 4 deletions(-)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Code changes
    - Fix Jakarta WebSocket API split dependency issue
      - Added jakarta.websocket:jakarta.websocket-client-api:2.2.0 as provided dependency (contains Session, @OnOpen, @OnClose, @OnError, @OnMessage classes which were split from the server API in Jakarta WebSocket 2.x)
    </details>
  
    ### ✅ Build Project
    - [[View Log]](logs/5.4.buildProject.log)
    
    - Build result: 100% Java files compiled
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Command
    `mvn clean test-compile -q -B -fn`
    </details>
  </details>

  ### ✅ Validate & Fix
  
  
  <details>
      <summary>[ click to toggle details ]</summary>
  
  - ###
    ### ✅ Validate CVEs
    - [[View Log]](logs/6.1.validateCves.log)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Checked Dependencies
      - jakarta.websocket:jakarta.websocket-client-api:2.2.0:jar
      - jakarta.websocket:jakarta.websocket-api:2.2.0:jar
      - jakarta.servlet:jakarta.servlet-api:6.1.0:jar
      - java:*:21
    </details>
  
    ### ✅ Validate And Fix Code Behavior Changes
    - [[View Log]](logs/6.2.validateBehaviorChanges.log)
    
    - 3 files changed, 148 insertions(+), 148 deletions(-)
  
    ### ✅ Run Tests
    - [[View Log]](logs/6.3.runTests.log)
    
    <details>
        <summary>[ click to toggle details ]</summary>
    
    #### Test result
    | Total | Passed | Failed | Skipped | Errors |
    |-------|--------|--------|---------|--------|
    | 0 | 0 | 0 | 0 | 0 |
    </details>
  </details>

  ### ✅ Summarize Upgrade
  - [[View Log]](logs/7.summarizeUpgrade.log)