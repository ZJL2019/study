# Spring Boot MVC

1. spring boot 集成了spring mvc框架并实现了自动配置。

   - 只需要在pom中添加对应依赖即可

2. Java包名结构

   - Controller  此包下包含了MVC的Controller
   - Service  此包下游业务处理代码
   - entity  包含了业务实体
   - conf  包含了一些配置类

3. 使用Controller

   1. Spring MVC 框架并不像传统的MVC框架必须继承某个基础类才可以处理http请求。

      - Spring MVC只需要在类上声明@Controller，标明这是一个Controller即可。
      - 对于用户请求使用@RequestMapping映射http请求到特定的方法处理类。

      ```java
      @Controller
      @RequestMapping("/test")
      public class HelloworldController {
      		@RequestMapping("/index.html")
      		public String say(Model model) {
      		model.addAttribute("name","hello,world");
      		return "index.btl";
      		}
      }
      ```
      - @Controller作用在类上，标明这是一个MVC中的Controller。@RequestMapping既可以作用在方法上，也可以作用在类上。如上代码所示，http访问/test/index.html 则会访问HelloworldController.say方法。
   
   		<!--say方法返回类型是String，默认是视图的名称。Spring Boot的视图默认保存在resource/templates下，渲染视图的是resource/templates/index.btl-->
   		
   	 - MVC有时候返回的是JSON字符串，如果想直接返回内容而不是视图名，则需要在方法上使用@ResponseBody
   	
   	```java
   	@RequestMapping("/index.json")
   	public @ResponseBody String say(){
   		return "hello, world";
   	}
   	```
   	
   	- ResponseBody 注解直接将返回的对象输出到客户端，如果是字符串，则直接返回；如果不是则默认使用Jackson序列化成JSON字符串后输出。
   	
   	```java
   	@RequestMapping(path = "all.json",method = RequestMethod.GET)
   	    public @ResponseBody List<User> allUser(){
   	        return userService.allUser();
   	}
   	```
   	
   	 <!--如果期望返回json,使用了ResponseBody注解,但是请求URL以html结尾，这会导致Spring Boot认为请求的是HTML类型的资源,而返回的是json的资源，与期望不一致，而报出相应的错误。建议在Spring Boot应用中，如果期望返回JSON，URL请求资源后缀是josn；如果期望返回视图，URL请求资源后缀是html-->
   	
   2. URL映射到方法
   	
   	Spring MVC提供了各种各样的映射方式
   	
   	1. @RequestMapping
   	
   	   	可以使用@RequestMapping来映射URL，比如/test到某个Controller类，或者是某个具体的方法。通常类上的注解@RequestMapping用来标注请求的路径，方法上的@RequestMapping注解进一步映射特定的URL到某个具体的方法。RequestMapping有多个属性来进一步匹配HTTP请求到Controller方法：
   	
   	   - value，请求的URL路径，支持URL模板、正则表达式。
   	   - method，HTTP请求方法，有GET、POST、PUT等
   	   - consumes，允许的媒体类型(Media Types)，如consumes="application/json"，对应于请求的HTTP的Content-Type。
   	   - produces，相应的媒体类型，如produces="application/json"，对应于HTTP的Accept字段。
   	   - params，请求的参数，如params="action=update"。
   	   - headers，请求的HTTP头的值，如headers=“myHeaders=myValue"。
   	   
   	2. URL路径匹配
   	
   	   属性value用于匹配一个URL映射，value支持简单的表达式来匹配:
   	   
   	   ```java
   	   @RequestMapping(value="/get/{id}.json")
   	   public @ResponseBody User getById(@PathVariable("id") Long id) {
   	   	return userService.getUserById(id);
   	   }
   	   ```
   	   
   	   如上，访问路径是/get/1.json，将调用getById方法，且参数id的值是1.注解PathVariable作用在方法参数上，用来表示参数的值来自URL路径。如果idea启用了debug模式，Spring则会自动识别URL中存在的表达式遇方法参数的对应关系。上述代码可以简化为：
   	   
   	   ```java
   	   @RequestMapping(path="/user/{id}.json",method=RequestMethod.GET)
   	   public @ResponseBody User getById(@PathVariable Long id){
   	   	return userService.getUserById(id);
   	   }
   	   ```
   	   
   	   也可以使用类似Ant的通配符来对URL进行映射。
   	   
   	   ```java
   	   @RequestMapping(path="/user/all/*.json",method=RequestMethod.GET)
   	   @ResponseBody
   	   public List<User> allUser(){
   	   	return userService.allUser();
   	   }
   	   ```
   	   
   	   Ant**路径表达式**
   	   
   	   	Ant用符号”*”来表示匹配任意字符，用“**”来表示统配任意路径，用“?”来匹配单个字符。
   	   
   	   - /user/*.html，匹配/user/1.html、/user/2.html等。
   	   - /**/1.html，匹配/1.html，也匹配/user/1.html，还匹配/user/add/1.html。
   	   - /user/?.html，匹配/user/1.html，但不匹配/user/11.html
   	   
   	   如果一个@RequestMapping能够匹配，通常会匹配更具体的。
   	   
   	   URL映射可以使用${}来获取系统的配置或者环境变量，通常用于Controller路径是通过配置文件设定的情况。
   	   
   	2. HTTP method匹配
   	
   	   @RequestMapping提供method属性来映射对应的HTTP的请求方法。
   	   
   	   - GET
   	   - POST
   	   - HEAD
   	   - PUT
   	   - DELETE
   	   - PATCH
   	   
   	   Spring提供了简化后的@RequestMapping，提供了新的注解来表示HTTP方法:
   	   
   	   - @GetMapping
   	   - @PostMapping
   	   - @PutMapping
   	   - @DeleteMapping
   	   - @PatchMapping
   	   
   	   因此上面的例子可以修改为:
   	   
   	   ```java
   	   @GetMapping("/user/all/*.json")
   	   public @ResponseBody List<User> allUser(){
   	   	return userService.allUser();
   	   }
   	   ```
   	   
   	2. consumes和produces
   	
   	   属性consumes意味着请求的HTTP头部的Content-Type媒体类型与consumes的值匹配，才能调用此方法。
   	   
   	   ```java
   	   @GetMapping(value="/consumes/test.json",consumes="application/json")
   	   @ResponseBody
   	   public User forJson(){
   	   	return userService.getUserById(11);
   	   }
   	   ```
   	   
   	   produces属性对应用HTTP请求的Accept字段，只有匹配得上的方法才能被调用。
   	   
   	5. params和header匹配
   	
   	   可以从请求参数或者HTTP头中提取值来进一步确定调用的方法，有以下三种形式：
   	
   	   - 如果存在参数，则通过；
   	   - 如果不存在参数，则通过；
   	   - 如果参数等于某一个具体的值，则通过；
   	
   	   ```java
   	    @PostMapping(path = "/update.json",params = "action=save")
   	       @ResponseBody
   	       public void test5(){
   	           System.out.println("call save");
   	       }
   	   ```
   	
   	   ```java
   	   @PostMapping(path = "update.json",params = "action=update")
   	       @ResponseBody
   	       public void test6(){
   	           System.out.println("call update");
   	       }
   	   ```
   	
   	   header与params一样。
   	
   	6. 方法参数
   	   
   	   Spring 的Controller方法可以接受多种类型参数，比如path，以及MVC中的Model。
   	   
   	   - @PathVariable，可以将URL中的值映射到方法参数中。
   	   - Model，Spring中通用的MVC模型，也可以使用Map和ModelMap作为渲染视图的模型。
   	   - ModelAndView，包含了模型和视图路径的对象。
   	   - JavaBean，将HTTP参数映射到JavaBean对象。
   	   - MultipartFile，用于处理文件上传。
   	   - @ModelAttribute，使用该注解的变量将作为Model的一个属性。
   	   - WebRequest或者NativeWebRequest，类似Servlet Request，但做了一定的封装。
   	   - java.io.InputStream和java.io.Reader，用来获取Server API中的InputSteam/Reader。
   	   - java.io.OutputStream/java.io.Writer，用来获取Servlet API中的OutputStream/Writer。
   	   - HttpMethod，枚举类型，对应于HTTP Method，如POST、GET。
   	   - @MatrixVariable，矩阵变量。
   	   - @RequestParam，对应于HTTP请求的参数，自动转换为参数对应的类型。
   	   - @RequestHeader，对应于HTTP请求头参数，自动转换为对应的类型。
   	   - @RequestBody，自动将请求内容转换为指定对象，默认使用HttpMessageConverters来转换。
   	   - @RequestPart，用于文件上传，对应HTTP协议的multipart/form-data.
   	   - @SessionAttribute，该方法标注的变量来自于Session的属性。
   	   - @RequestAttribute，该标注的变量来自于request的属性。
   	   - @InitBinder，用在方法上，说明这个方法会注册多个转化器，用来个性化地将HTTP请求参数转化为对应的Java对象，如转化为日期类型、浮点类型、JavaBean等。当然也可以实现WebBindingInitializer接口来用于Spring Boot应用需要的dataBinder。
   	   - BindingResult和Errors，用来处理绑定过程中的错误。
   	
   3. PathVariable
   
   	注解PatchVariable用于从请求URL中获取参数并映射到方法中。
   	
   	```java
   	@Controller
   	@RequestMapping("/user/{id}")
   	public class Sample35Controller {
   	  @Autowired UserService uservice;
   	  @GetMapping(path = "/{type}/get.json")
   	  @ResponseBody
   	  public User getUser(@PathVariable Long id , @PathVariable Integer type){
   	    return userServcie.getUserById(id);
   	  }
   	}
   	```
   	
   4. JavaBean接受HTTP参数
   	
   	HTTP提交的参数可以映射到方法参数上，按照名称来映射。可以通过注解@RequestParam来进一步限定HTTP参数到Controller方法的映射关系，RequestParam支持如下属性：
   	
   	- value，指明HTTP参数的名称。
   	
   	- required。boolean类型，声明此参数是否必须有，如果http参数请求里没有，则会抛出400错误。
   	
   	- defaultValue，字符类型，如果HTTP参数没有提供，可以指定一个默认字符串，Spring类型转化为目标类型。
   	
   5. @RequestBody接受JSON
   	
   	Controller方法带有@RequestBody注解的参数，意味着请求的HTTP消息体的内容是一个JSON，需要转化为注解指定的参数类型。SpringBoot默认使用JackSon类处理反序列化工作。
   	
   	```java
   	@PostMapping("/user.json")
   	    public WebResponse<User> test8(@RequestBody User user){
   	        return WebResponse.buildData(user);
   	    }
   	```
   	
   6. 验证框架
   
      SpringBoot支持JSR-303、Bean验证框架，默认实现用的是Hibernate validator。在Spring MVC中，只需要使用@Valid注解标注在方法参数上，Spring Boot即可对参数对象进行校验，校验结果放在BindingResult对象中。
   
      1. JSR-303
   
         JSR-303是Java标准的验证框架，已有的实现有Hibernate validator。JSR-303定义了一系列注解用来验证Bean的属性，常用的有以下几种：
   
         - @Null
         - @NotNull
         - @NotBlank
         - @NotEmpty
         - @Size(min=,max=)
         - @Length
         - @Min
         - @Max
         - @Digits 验证数字是否符合指定形式，如@Digits(integer=9,fraction=2)
         - @Range 验证数字是否在指定范围，如@Range(min=1,max=1000)
         - @Email 验证是否为邮件格式，为null则不做校验
         - @Pattern 验证String对象是否符合正则表达式的规则
   
         JSR-303定义了group概念，每个校验都必须支持。校验注解作用在字段上的时候，可以指定一个或者多个group，当Spring Boot校验对象时，也可以指定校验的上下文属于那个group。这样只有group匹配的时候，校验注解才能生效。
   
         ```java
         public class WorkInfoForm{
         	public interface Update{}
         	public interface Add{}
         	
         	@NotNull(groups={Update.class})
         	@Null(groups={Add.class})
         	Long id;
         }
         ```
   
         当校验上下文为Add.class的时候，@Null生效，id为空才能校验通过；
   
         当校验上下文为Update.class的时候，@NotNull生效，id不为空。