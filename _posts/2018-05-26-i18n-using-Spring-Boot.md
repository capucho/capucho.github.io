Nowadays, Spring Boot has achieved what Java developers have been waiting for a long time: a fast and simple way to create a project which is reliable and can go to a production environment in a couple of days or weeks. 

Consequently, one of many concerns when developing a project is how to handle internationalization. The growth of REST API's architecture made this even more necessary. 

Ok, but let's cut to the chase. I'm going to create a simple project just to prove how effortless it is. In the first place, let's open https://start.spring.io/ and create a new project using the Web dependency.

![Sprint Initializr](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Spring-Starter.png)

After the download is complete, let's open it with Eclipse (or the IDE of your choice). You should import as an Existing Maven project. Once you have done this, let's start the project. 

There are two ways to start the project: one is inside Eclipse and the other is using a terminal. Let's use the second option for now because it's easier to read what is going on. So, open your terminal and move to the project folder. Once you're there type the following command: 

```
mvn spring-boot:run
```

What is going on now? Maven will start downloading the dependencies and after that, Spring will boot the server with your application code.

![Spring Boot Running](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Server-Running.png)

The port that will be used by Spring Boot as default is 8080.

Now we can start typing our code. Let's create a Rest Controller that will print a message to us using a parameter.

![Project Structure](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Project-Structure.png)

```Java
package com.joaocapucho.springi18n.controllers;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {

	@GetMapping(path = "/hello-world/{name}")
	public String helloWorld(@PathVariable String name) {
		return "Hello World, " + name;
	}
}
```

So, start your Spring Boot application again and call the endpoint. You will see the result in your browser.

![Hello World](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Hello-World.png)

However, I'm Brazilian and I would like to see the text in Portuguese. What should I do? Let's create a file where all our messages should be inside, using a key=value pair to identify them. So, at the same folder as our application.properties file (src/main/resources) we will create a messages.properties and a messages_pt.properties.

The project structure will look like this:

![Project Structure 2](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Project-Structure-2.png)

The files will have the following code:

 - English
```properties
messages.hello.world=Hello World
```

- Portuguese
```properties
messages.hello.world=Ola Mundo
```

How will Spring know that should look into those files? First, we have to configure which files will be used as message properties. This is the bean that we would use to configure the message source:

```Java
public ResourceBundleMessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasename("messages");
  return messageSource;
}
```
However, Spring Boot makes easier to do it, instead of adding the code above, you'll just need to add one line inside application.properties file, like this:

```
spring.messages.basename=messages
```

Now, we have only 2 more steps to fix the problem. The first one is to create a bean to resolve the `Accept-Language` header for us. Doing that, we will avoid retrieving the information from the request headers in our controller methods.

This is done by creating a bean called `localeResolver`, like this:

```Java
package com.joaocapucho.springi18n;

import java.util.Locale;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

@SpringBootApplication
public class SpringI18nApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringI18nApplication.class, args);
	}
	
	@Bean
	public LocaleResolver localeResolver( ) {
		AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
		localeResolver.setDefaultLocale(Locale.US);
		return localeResolver;
	}
}
```

Now, the last step is to retrieve the message using the information from the LocaleResolver. We will need to add a MessageSource bean in our controller and use it to retrieve the message. 

```Java
package com.joaocapucho.springi18n.controllers;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {
	
	@Autowired
	private MessageSource messageSource;

	@GetMapping(path = "/hello-world/{name}")
	public String helloWorld(@PathVariable String name) {
		return messageSource.getMessage("messages.hello.world", null, LocaleContextHolder.getLocale()) + ", " +  name;
	}
}
```

The LocaleContextHolder has the location information provided by the localeResolver bean. 

To test the implementation, we will use Postman (https://www.getpostman.com/) to send the request with the headers. 

- Without using an Accept-Language header

![NoHeader](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/NoHeader.png)

- Using an Accept-Language header

![NoHeader](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/Ptheader.png)

- Using an Accept-Language header with a language that is not mapped

![NoHeader](/assets/img/posts/2018-05-26-i18n-using-Spring-Boot/WrongHeader.png)

