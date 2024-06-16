How to Dockerize a simple Spring Boot REST application.


Let’s see how to create the simplest Spring Boot REST application to make it work in a container. This may not look very useful but, this application might come in handy whenever you are trying out a something related to Docker.

If you aready know how to Dockerize an Spring Boot app and all you need is an example code, jump to the github repo — https://github.com/chameerar/spring-rest-dockerized

There are some prerequisites to this operation. You should have,

Java 17
Maven 3.6 or above
Docker
installed in your computer. Also you need to have some context on how a Java code works, how Maven is used as a build tool and a package manager for Java, and how does Docker works. If you are completely new to this I’ll mention some links in the Additional Reads section below for you to refer before getting in.

Create the Spring Boot project
I’m going to use the easiest way to create an SpringBoot application. All you need to do is to visit the Spring Initializr from your web browser. There you can select the Project type, Language, Spring Boot version, Project Metadata, Packaging type, etc. Here I’m going to go with a Maven project with Java as the language using Java 17. Fill up the form with the values given below.


Project definitions for creating the project
You can name your project as you wish. Therefore, change the Project Metadata to your preference.

Since we are going to develop a REST API here, we are going to need the Spring Web dependency. Let’s add the dependency to the project by clicking the ADD DEPENDENCIES button. Select Spring Web from the list. Now your setup should look like this.


Setup for the Spring Boot REST project
Click the GENERATE button to download the generated Spring Boot project. The project will be downloaded as .zip file. Extract it to a desired location. Open the project using a Java-friendly IDE. I’ll use Intellij IDEA for this.

Explore the Project
Let’s explore the project we just generated. The project hierarchy will be something like this.


Project structure
Let’s get rid of some elements in the project which are not needed for our case. Delete the .mvn directory along with the mnvw, mvnw.cmd, and HELP.md files.

Now the project is ready to proceed.

Implement the REST API
Let’s implement the REST API to return a response upon. I’m going to create a new java class HelloController with the @RestController annotation which will be considered as a controller. For the sake of clarity I will create this java class under a new package named controller.


First look of HelloController
Next, let’s add @RequestMapping annotation with “hello” as the value to denote the base path of this controller. The class will look like this.

package io.github.chameerar.SpringHello.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("hello")
public class HelloController {

}
Let’s add a GET method to return “Hello World” message. This method will have path mapping “/message”.

The final outcome will look like this.

package io.github.chameerar.SpringHello.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("hello")
public class HelloController {

    @GetMapping("/message")
    public String HelloWorld(){
        return "Hello World";
    }
}
Now you have a REST API with a simple GET method.

Test before Dockerizing (Optional)
Let’s test the REST API we created before moving forward with the Dockerization. If you are confidant enough, you can skip this step.

Use Maven to build the project. Navigate to the root directory of the project using terminal/cmd and execute the maven install command.

mvn clean package
This will build the project and create SpringHello-0.0.1-SNAPSHOT.jar inside the target directory. Now let’s run the project using the java -jar command. Navigate to the target directory and execute the command below.

java -jar SpringHello-0.0.1-SNAPSHOT.jar
This will start the Spring Boot server and you can send requests to the server to localhost:8080/hello/message URL. Let’s open a web browser and check whether this works.


Sending request to test the REST API
If you can complete every step upto now successfully, congrats you are good to proceed. If not please check what has gone wrong. Specially check the java and maven versions.

Dockerize the Spring Boot Application
Let’s dockerize our Spring Boot application. We can start this by creating a file named Dockerfile in our project root directory. Now your project structure will look like this.


Dockerfile created in the root of the Spring Boot project
Add the below content to the Dockerfile.

#
# Build stage
#
FROM maven:3.8.3-openjdk-17 AS build
COPY src /home/app/src
COPY pom.xml /home/app
RUN mvn -f /home/app/pom.xml clean package

#
# Run stage
#
FROM openjdk:17
COPY --from=build /home/app/target/SpringHello-0.0.1-SNAPSHOT.jar /usr/local/lib/SpringHello-0.0.1-SNAPSHOT.jar
EXPOSE 8080
USER 10014
ENTRYPOINT ["java","-jar","/usr/local/lib/SpringHello-0.0.1-SNAPSHOT.jar"]
This Dockerfile contains two stages. The Build Stage builds the project and creates the .jar and the Run Stage copies the .jar into the container and runs it.

Now the project Dockerization is complete. Let’s test this using Docker CLI commands. Open a terminal/cmd in project root and execute the below command to build the Docker image.

docker build -t spring-rest:1.0 .
Successful completion of Docker image build will look like this.

 => [stage-1 2/2] COPY --from=build /home/app/target/SpringHello-0.0.1-SNAPSHOT.jar /usr/local/lib/SpringHello-0.0.1-SNAPSHOT.jar                                                                                                  0.1s
 => exporting to image                                                                                                                                                                                                             0.1s
 => => exporting layers                                                                                                                                                                                                            0.1s
 => => writing image sha256:8a0cfddcb58e3efb6c64b35506aa24ea28b3ac5bce5e70632876859e9f276203                                                                                                                                       0.0s
 => => naming to docker.io/library/spring-rest:1.0
Let’s see the newly buit image by listing the Docker images using the below command.

docker images
You’ll see the newly created spring-rest image in the list.


Newly created spring-rest image
Let’s run the image using docker run command. You can specify a port to be exposed for this container. I’ll go with port 4000 for this instance.

docker run -p 4000:8080 --name spring-rest-container spring-rest:1.0
With this the Docker container will start and we can send request to it through address localhost:4000 . Let’s check this in browser by visiting localhost:4000/hello/message


Response from Docker container
As you can see this works as expected. If you are done with testing this you can shutdown the container using the command below.

docker stop spring-rest-container
Conclusion
We can easily dockerize any Spring REST API project following this approach. Even though this example is one of the simplest projects you can find, the same can be applied to a complex project as well.

You can find this project in github — https://github.com/chameerar/spring-rest-dockerized. Feel free to fork this and submit any changes.
