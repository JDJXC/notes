## **一、servlet的本质是什么，它是如何工作的？**

Servlet（Server Applet）是Java Web应用程序中的一部分，用于处理来自Web客户端（如浏览器）的请求并生成响应。它是运行在支持Java Servlet规范的服务器上的Java程序，通常是运行在Servlet容器（如Apache Tomcat, Jetty, GlassFish等）中的。

### 1、Servlet的本质

1. **Java类**：Servlet本质上是一个实现了特定接口的Java类，即`javax.servlet.Servlet`接口。但由于直接实现该接口需要编写大量的代码，所以通常我们会选择继承`javax.servlet.http.HttpServlet`类，该类是`Servlet`接口的一个通用实现。
2. **处理请求**：Servlet的主要功能是处理HTTP请求并生成HTTP响应。这包括解析请求参数、访问数据库、执行业务逻辑、设置响应头、写入响应体等。
3. **生命周期管理**：Servlet容器负责Servlet的生命周期管理，包括加载、实例化、初始化、服务（处理请求）和销毁。

### 2、Servlet的工作流程

1. **加载和实例化**：当Servlet容器启动时，或当Servlet容器检测到需要这个Servlet来处理请求时，Servlet容器会加载并实例化Servlet类。这个过程通常只会在第一次请求时发生，因为Servlet容器通常会保持Servlet的实例以便后续请求使用。
2. **初始化**：在Servlet实例化后，Servlet容器会调用Servlet的`init()`方法进行初始化。在`init()`方法中，可以进行一些只需要执行一次的初始化操作，如加载配置文件、建立数据库连接等。
3. **服务（处理请求）**：当客户端发送HTTP请求到Servlet容器时，Servlet容器会解析请求，并根据请求的URL找到对应的Servlet实例。然后，Servlet容器会调用Servlet的`service()`方法来处理请求。但是，实际上`service()`方法通常不会被直接调用，而是由Servlet容器根据请求的类型（GET、POST等）调用相应的`doGet()`、`doPost()`等方法。在`doGet()`或`doPost()`方法中，可以编写处理请求的逻辑，并调用`response`对象的方法来设置响应头、写入响应体等。
4. **销毁**：当Servlet容器关闭或需要释放Servlet实例占用的资源时，会调用Servlet的`destroy()`方法进行销毁。在`destroy()`方法中，可以执行一些清理操作，如关闭数据库连接、释放资源等。

需要注意的是，Servlet是单例的，即对于每个Servlet类，Servlet容器只会创建一个实例。这意味着在Servlet类中定义的实例变量会被多个线程共享，因此在编写Servlet时需要注意线程安全的问题。

## 二、ServletContext以及Servlet映射规则

### 1、ServletContext是什么

ServletContext，直译的话叫做“Servlet上下文”，听着挺别扭。它其实就是个大容器，是个map。服务器会为每个应用创建一个ServletContext对象：

- ServletContext对象的创建是在服务器启动时完成的

- ServletContext对象的销毁是在服务器关闭时完成的

  ![](..\assets\spring\image\ServletContext是什么.png)

ServletContext对象的作用是在整个Web应用的动态资源（Servlet/JSP）之间共享数据。

这种用来装载共享数据的对象，在JavaWeb中共有4个，而且更习惯被成为“域对象”：

- ServletContext域（Servlet间共享数据）
- Session域（一次会话间共享数据，也可以理解为多次请求间共享数据）
- Request域（同一次请求共享数据）
- Page域（JSP页面内共享数据）


它们都可以看做是map，都有getAttribute()/setAttribute()方法。

### 2、如何获取ServletContext

![用域对象获得域对象](..\assets\spring\image\servletContext.jpg)

所以，获取ServletContext的方法共5种（page域这里不考虑，JSP太少用了）：

- ServletConfig#getServletContext();
- GenericServlet#getServletContext();
- HttpSession#getServletContext();
- HttpServletRequest#getServletContext();
- ServletContextEvent#getServletContext();

### 3、Filter拦截方式之：REQUEST/FORWARD/INCLUDE/ERROR

在Java的Servlet规范中，Filter有以下四种拦截方式：

1. REQUEST：每次用户发起一个新的请求，都会调用该Filter。
2. FORWARD：只有当用户请求被转发的时候，才会调用该Filter。
3. INCLUDE：只有当用户请求被包含的时候，才会调用该Filter。
4. ERROR：只有当用户请求因为错误而执行的时候，才会调用该Filter。

### 4、Servlet映射器

在Java的Servlet技术中，Servlet映射器（或称为Servlet映射）是一个将URL模式映射到特定Servlet的机制。这允许开发人员定义哪些URL应该由哪个Servlet处理。这种映射通常在web应用的`web.xml`部署描述符中配置，或者使用Servlet 3.0及以上版本的`@WebServlet`注解来定义。

#### 1）使用`web.xml`进行Servlet映射

在`web.xml`文件中，你可以使用`<servlet-mapping>`元素来定义Servlet映射。以下是一个简单的例子：

```xml
<web-app ...>  
    ...  
    <servlet>  
        <servlet-name>MyServlet</servlet-name>  
        <servlet-class>com.example.MyServlet</servlet-class>  
    </servlet>  
  
    <servlet-mapping>  
        <servlet-name>MyServlet</servlet-name>  
        <url-pattern>/myPath/*</url-pattern>  
    </servlet-mapping>  
    ...  
</web-app>
```

在这个例子中，`MyServlet`类将处理所有以`/myPath/`开头的URL。

#### 2）使用`@WebServlet`注解进行Servlet映射

如果你使用的是Servlet 3.0或更高版本，你可以使用`@WebServlet`注解直接在Servlet类上定义映射，而无需在`web.xml`中配置。以下是一个例子：

```java
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;  
  
@WebServlet("/myPath/*")  
public class MyServlet extends HttpServlet {  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {  
        // 处理GET请求  
    }  
  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {  
        // 处理POST请求  
    }  
}
```

在这个例子中，`MyServlet`类将处理所有以`/myPath/`开头的URL，与`web.xml`中的配置效果相同。

#### 3）URL模式

在定义URL模式时，你可以使用通配符来匹配多个URL。例如：

- `/myPath/*`：匹配所有以`/myPath/`开头的URL。
- `*.do`：匹配所有以`.do`结尾的URL。
- `/`：匹配根URL。

但是，请注意不要将`/*`用作URL模式，因为它会捕获所有请求，这可能会导致意外的行为。如果你想要捕获所有请求并做进一步处理（例如，通过前端控制器模式），应该谨慎使用这种模式。

### 5、DispatcherServlet与SpringMVC

`DispatcherServlet` 是 Spring MVC 框架的核心组件，它负责接收所有的 HTTP 请求，并根据配置将请求分发给合适的控制器（通常是基于注解的 `@Controller` 类或者实现了 `Controller` 接口的类）。Spring MVC 是一个基于 Java 的实现 MVC 设计模式的请求驱动类型的轻量级 Web 框架，它依赖于 Servlet API，并利用了 Spring 框架的功能。

#### 1） DispatcherServlet 的作用

1. **接收请求**：`DispatcherServlet` 作为前端控制器（Front Controller），接收来自客户端的所有 HTTP 请求。
2. **查找处理器**：根据请求 URL 和配置的映射规则（通常是基于注解或者 XML 配置），找到处理该请求的控制器。
3. **调用处理器**：将请求分发给对应的控制器，并等待控制器的处理结果。
4. **视图解析**：控制器处理完请求后，返回一个视图名称或视图对象，`DispatcherServlet` 根据配置的视图解析器（ViewResolver）解析视图名称，得到真正的视图对象（如 JSP 页面）。
5. **渲染视图**：将模型数据（Model）和视图对象合并，生成最终的响应，并返回给客户端。

#### 2）Spring MVC 的组成

Spring MVC 主要由以下几个部分组成：

1. **前端控制器（DispatcherServlet）**：负责接收请求，并分发到相应的处理器。
2. **处理器映射（Handler Mapping）**：根据请求的 URL 查找匹配的处理器（Controller）。
3. **处理器（Controller）**：处理具体的业务逻辑，并返回视图名称或视图对象。
4. **模型与视图（ModelAndView）**：Controller 处理完请求后，返回一个 ModelAndView 对象，包含了模型数据和视图名称。
5. **视图解析器（ViewResolver）**：根据视图名称解析为真正的视图对象（如 JSP 页面）。
6. **视图（View）**：负责渲染模型数据，生成最终的响应结果。

#### 3） 使用 Spring MVC

在 Spring MVC 中，你通常会配置一个或多个 `DispatcherServlet` 来处理你的 Web 请求。你可以使用 XML 配置或者基于 Java 的配置（如 `@Configuration` 和 `@Bean` 注解）来定义你的控制器、处理器映射、视图解析器等组件。同时，你也可以使用 Spring 提供的各种功能，如数据绑定、验证、国际化、异常处理等，来增强你的 Web 应用的功能。

#### 4） 总结

`DispatcherServlet` 是 Spring MVC 的核心组件，它负责接收请求、分发请求、解析视图和渲染响应。而 Spring MVC 是一个基于 Java 的 Web 框架，它提供了完整的 MVC 实现，包括前端控制器、处理器映射、控制器、模型与视图、视图解析器等组件，以及数据绑定、验证、国际化等丰富的功能。

### 6、conf/web.xml与应用的web.xml

- `WEB-INF/web.xml`是Java Web应用的标准配置文件，用于配置Servlet、过滤器、监听器等。
- `conf/web.xml`可能是一个特定服务器或框架使用的全局配置文件，或者是项目中自定义的配置文件路径。它不是Java Web应用的标准配置文件路径。

### 7、Tomcat和Spring的关系

Tomcat是一个开源的Web服务器和Servlet容器，主要用于提供HTTP服务并处理Servlet请求。Spring是一个开源的Java框架，用于简化企业级应用开发。

**Tomcat和Spring的关系主要体现在以下两个方面**：

1. **运行环境**：Spring框架编写的Web应用可以运行在Tomcat这样的Servlet容器中。Tomcat提供了Java Web应用运行所需的环境，包括Servlet API和JSP API等。
2. **集成**：虽然Tomcat和Spring是两个独立的框架，但它们可以很好地集成在一起。Spring提供了多种方式来集成Tomcat，例如通过Maven或Gradle等构建工具将Spring Web应用打包成WAR文件，然后部署到Tomcat中。此外，Spring Boot还提供了内嵌的Tomcat服务器，使得开发者可以在本地直接运行Spring应用，而无需单独安装和配置Tomcat。

### 8、SpringMVC和Servlet的关系

SpringMVC是Spring框架的一个模块，用于构建Web应用。Servlet是Java Web应用的核心技术之一，用于处理HTTP请求并生成响应。

**SpringMVC和Servlet的关系主要体现在以下两个方面**：

1. **基于Servlet**：SpringMVC是建立在Servlet技术之上的。它使用Servlet API来处理HTTP请求，但提供了更加简洁、灵活的方式来编写Web应用。例如，SpringMVC通过控制器（Controller）来处理请求，并通过视图（View）来渲染响应。这使得开发者可以更加关注业务逻辑的实现，而不是底层的技术细节。
2. **扩展和简化**：虽然SpringMVC建立在Servlet之上，但它对Servlet API进行了扩展和简化。SpringMVC提供了许多注解和配置选项，使得开发者可以更加容易地定义请求映射、参数绑定、异常处理等功能。此外，SpringMVC还提供了与其他Spring模块的集成，例如Spring Security用于安全控制、Spring Data用于数据访问等。这使得开发者可以更加容易地构建功能强大的Web应用。

总结来说，Tomcat是Web服务器和Servlet容器，用于运行Java Web应用；Spring是一个全面的Java框架，用于简化企业级应用开发；SpringMVC是Spring框架的一个模块，用于构建Web应用，并建立在Servlet技术之上。这些技术可以相互集成和协作，以构建高效、可维护的Java Web应用。

## 三、补充知识

### 1、MVC 设计模式是什么

MVC（Model-View-Controller）是一种软件设计模式，用于将应用程序的逻辑、数据和用户界面（UI）分离。这种分离使得应用程序更易于管理、维护和扩展。MVC 设计模式最初是由 Trygve Reenskaug 在 1978 年为 Smalltalk 语言发明的，但后来被广泛应用于许多其他的编程语言和环境，包括 Java、C#、PHP 和 JavaScript（特别是在前端开发框架中，如 React、Vue 和 Angular）。

MVC 设计模式包含以下三个核心组件：

1. 模型（Model）
   - 模型是应用程序中用于处理数据逻辑的部分。
   - 它包含了应用程序的核心数据和业务逻辑。
   - 模型与数据库进行交互，以存储和检索数据。
   - 当数据发生变化时，模型会通知视图进行更新。
2. 视图（View）
   - 视图是应用程序的用户界面部分。
   - 它负责显示模型中的数据。
   - 视图通常不处理业务逻辑，它只是简单地展示数据给用户。
   - 当模型中的数据发生变化时，视图应该自动更新以反映这些变化。
3. 控制器（Controller）
   - 控制器是模型和视图之间的协调者。
   - 它接收用户的输入（如点击按钮或提交表单），并决定如何处理这些输入。
   - 控制器可能会更新模型中的数据，并指示视图进行更新。
   - 控制器还负责确保模型和视图之间的数据同步。

MVC 设计模式的好处包括：

- **低耦合**：模型、视图和控制器之间保持独立，这意味着更改一个组件的代码不太可能影响到其他组件。
- **高内聚**：每个组件都专注于其特定的任务（模型处理数据，视图显示数据，控制器协调两者）。
- **可重用性**：由于模型和视图是分离的，因此可以轻松地重用相同的模型来处理不同的视图，或者将相同的视图与不同的模型一起使用。
- **易于维护和扩展**：由于每个组件都是独立的，因此可以单独测试和维护它们。此外，当需要添加新功能时，只需更改相应的组件即可，而无需更改整个应用程序。
- **清晰的职责划分**：MVC 模式清晰地定义了每个组件的职责，这使得开发人员更容易理解代码并协作开发。

## 参考资料

[servlet的本质是什么，它是如何工作的？4732 赞同 · 182 评论回答](https://www.zhihu.com/question/21416727/answer/690289895)

[Servlet（下）658 赞同 · 53 评论文章](https://zhuanlan.zhihu.com/p/65658315)

[bravo1988：Java程序员练级路线23 赞同 · 0 评论文章](https://zhuanlan.zhihu.com/p/686288683)

[bravo1988：Servlet（下）658 赞同 · 53 评论文章](https://zhuanlan.zhihu.com/p/65658315)