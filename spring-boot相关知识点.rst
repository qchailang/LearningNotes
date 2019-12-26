maven的默认编译使用的jdk版本貌似很低，如果我们不告诉它我们的代码要使用什么样的jdk版本编译的话，它就会用maven-compiler-plugin默认的jdk版本来进行处理，这样就容易出现版本不匹配，以至于可能导致编译不通过的问题。使用maven-compiler-plugin插件可以指定项目源码的jdk版本，编译后的jdk版本，以及编码。

.. code:: java

  <build>
    <plugins>
      <plugin>                                                                                                                                      
          <!-- 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 -->                                                                           
          <groupId>org.apache.maven.plugins</groupId>                                                                                               
          <artifactId>maven-compiler-plugin</artifactId>                                                                                            
          <version>3.1</version>                                                                                                                    
          <configuration>                                                                                                                           
              <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中不能使用低版本jdk中不支持的语法)，会存在target不同于source的情况   -->                    
              <source>1.8</source> <!-- 源代码使用的JDK版本 -->                                                                                             
              <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->                                                                                     
              <encoding>UTF-8</encoding><!-- 字符集编码 -->
              <skipTests>true</skipTests><!-- 跳过测试 -->                                                                             
              <verbose>true</verbose>
              <showWarnings>true</showWarnings>                                                                                                               
              <fork>true</fork><!-- 要使compilerVersion标签生效，还需要将fork设为true，用于明确表示编译版本配置的可用 -->                                                        
              <executable><!-- path-to-javac --></executable><!-- 使用指定的javac命令，例如：<executable>${JAVA_1_4_HOME}/bin/javac</executable> -->           
              <compilerVersion>1.3</compilerVersion><!-- 指定插件将使用的编译器的版本 -->                                                                         
              <meminitial>128m</meminitial><!-- 编译器使用的初始内存 -->                                                                                      
              <maxmem>512m</maxmem><!-- 编译器使用的最大内存 -->                                                                                              
              <compilerArgument>-verbose -bootclasspath ${java.home}\lib\rt.jar</compilerArgument><!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->               
          </configuration>                                                                                                                          
      </plugin> 
    </plugins>
  </build>

POM 文件中添加了“org.springframework.boot:spring-boot-maven-plugin”插件。在添加了该插件之后，当运行“mvn package”进行打包时，会打包成一个可以直接运行的 JAR 文件，使用“Java -jar”命令就可以直接运行。这在很大程度上简化了应用的部署．

可以在POM中，指定生成 的是Jar还是War。

<packaging>jar</packaging>

还可以指定要执行的类，如果不指定的话，Spring会找有这个【public static void main(String[] args)】方法的类，当做可执行的类。

如果要指定的话，可以用下面两个方法：

1，如果POM是继承spring-boot-starter-parent的话，只需要下面的指定就行。

.. code:: java

    <properties>
        <!-- The main class to start by executing java -jar -->
        <start-class>com.mycorp.starter.HelloWorldApplication</start-class>
    </properties>

2，如果POM不是继承spring-boot-starter-parent的话，需要下面的指定。

.. code:: java

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>1.3.5.RELEASE</version>
        <configuration>
          <mainClass>${start-class}</mainClass> <!--指定要执行的类-->
          <layout>ZIP</layout>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>  <!--可以把依赖的包都打包到生成的Jar包中-->
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

Spring Boot+maven打war包的方法
==================================
1.指定war打包方式

.. code:: java

  <packaging>war</packaging>
2.pom.xml添加spring-boot-maven-plugin插件

.. code:: java

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>2.0.1.RELEASE</version>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
3.pom.xml添加spring-boot-starter-tomcat依赖

.. code:: java

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
  </dependency>

4.启动类继承SpringBootServletInitializer并重写configure方法

.. code:: java

  @SpringBootApplication
  public class Application extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
      return application.sources(Application.class);
    }
    public static void main(String[] args) throws Exception {
      SpringApplication.run(Application.class, args);
    }
  }