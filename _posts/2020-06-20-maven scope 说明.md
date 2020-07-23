+ ### maven scope 说明

  +  compile（编译范围） 默认。

     表示为当前依赖参与项目的编译、测试和运行阶段，属于强依赖。打包之时，会达到包里去 

    例如：spring-core

  +  test  

  ​        表示被依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。 

  ​    	例如： Junit 测试 

  +  provided（已提供范围）

     被依赖项目理论上可以参与编译、测试、运行等阶段，相当于compile，但是再打包阶段做了exclude的动作 

    例如： 例如， 如果我们在开发一个web 应用，在编译时我们需要依赖 servlet-api.jar，但是在运行时我们不需要该 jar 包，因为这个 jar 包已由应用服务器提供，此时我们需要使用 provided 进行范围修饰 

  + runtime（运行时范围）

     表示被依赖项目无需参与项目的编译，但是会参与到项目的测试和运行。与compile相比，被依赖项目无需参与项目的编译。 

    例如：JDBC 驱动

  + system(系统范围)

     system 元素与 provided 元素类似，但是被依赖项不会从 maven 仓库中查找，而是从本地系统中获取，systemPath 元素用于制定本地系统中 jar 文件的路径 

    例如：

    ```xml
    <dependency>
        <groupId>org.open</groupId>
        <artifactId>open-core</artifactId>
        <version>1.5</version>
        <scope>system</scope>             
        <systemPath>${basedir}/WebContent/WEB-INF/lib/opencore.jar</systemPath>
    </dependency>
    ```

  + import(只能在 dependencyManagement 标签之下使用，目的解决maven parent 单继承的问题，所继承项目必须是pom )

     例如：

    ```xml
    <project>
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.test.sample</groupId>
        <artifactId>base-parent1</artifactId>
        <packaging>pom</packaging>
        <version>1.0.0-SNAPSHOT</version>
        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>junit</groupId>
                    <artifactid>junit</artifactId>
                    <version>4.8.2</version>
                </dependency>
                <dependency>
                    <groupId>log4j</groupId>
                    <artifactid>log4j</artifactId>
                    <version>1.2.16</version>
                </dependency>
            </dependencies>
        </dependencyManagement>
    </project>
    ```

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
    </parent>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.test.sample</groupId>
                <artifactid>base-parent1</artifactId>
                <version>1.0.0-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependency>
        <groupId>junit</groupId>
        <artifactid>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactid>log4j</artifactId>
    </dependency>
    ```

    