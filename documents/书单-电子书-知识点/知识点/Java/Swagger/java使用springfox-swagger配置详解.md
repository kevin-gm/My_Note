### 一、引用依赖

#### Maven

	<dependency>
	    <groupId>io.springfox</groupId>
	    <artifactId>springfox-swagger2</artifactId>
	    <version>2.8.0</version>
	</dependency>

#### gradle

	dependencies {
	    compile "io.springfox:springfox-swagger2:2.8.0"
	}

### 二、基本配置
一般项目不需要这么多配置，为了后面说明配置属性，此处多列了很多属性。

	@SpringBootApplication
	@EnableSwagger2
	@ComponentScan(basePackageClasses = {
	    PetController.class
	})
	public class Swagger2SpringBoot {
	
	  public static void main(String[] args) {
	    ApplicationContext ctx = SpringApplication.run(Swagger2SpringBoot.class, args);
	  }
	
	
	  @Bean
	  public Docket petApi() {
	    return new Docket(DocumentationType.SWAGGER_2)
	        .select()
	          .apis(RequestHandlerSelectors.any())
	          .paths(PathSelectors.any())
	          .build()
	        .pathMapping("/")
	        .directModelSubstitute(LocalDate.class,
	            String.class)
	        .genericModelSubstitutes(ResponseEntity.class)
	        .alternateTypeRules(
	            newRule(typeResolver.resolve(DeferredResult.class,
	                typeResolver.resolve(ResponseEntity.class, WildcardType.class)),
	                typeResolver.resolve(WildcardType.class)))
	        .useDefaultResponseMessages(false)
	        .globalResponseMessage(RequestMethod.GET,
	            newArrayList(new ResponseMessageBuilder()
	                .code(500)
	                .message("500 message")
	                .responseModel(new ModelRef("Error"))
	                .build()))
	        .securitySchemes(newArrayList(apiKey()))
	        .securityContexts(newArrayList(securityContext()))
	        .enableUrlTemplating(true)
	        .globalOperationParameters(
	            newArrayList(new ParameterBuilder()
	                .name("someGlobalParameter")
	                .description("Description of someGlobalParameter")
	                .modelRef(new ModelRef("string"))
	                .parameterType("query")
	                .required(true)
	                .build()))
	        .tags(new Tag("Pet Service", "All apis relating to pets")) 
	        .additionalModels(typeResolver.resolve(AdditionalModel.class)) 
	        ;
	  }
	
	  @Autowired
	  private TypeResolver typeResolver;
	
	  private ApiKey apiKey() {
	    return new ApiKey("mykey", "api_key", "header");
	  }
	
	  private SecurityContext securityContext() {
	    return SecurityContext.builder()
	        .securityReferences(defaultAuth())
	        .forPaths(PathSelectors.regex("/anyPath.*"))
	        .build();
	  }
	
	  List<SecurityReference> defaultAuth() {
	    AuthorizationScope authorizationScope
	        = new AuthorizationScope("global", "accessEverything");
	    AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
	    authorizationScopes[0] = authorizationScope;
	    return newArrayList(
	        new SecurityReference("mykey", authorizationScopes));
	  }
	
	  @Bean
	  SecurityConfiguration security() {
	    return SecurityConfigurationBuilder.builder()
	        .clientId("test-app-client-id")
	        .clientSecret("test-app-client-secret")
	        .realm("test-app-realm")
	        .appName("test-app")
	        .scopeSeparator(",")
	        .additionalQueryStringParams(null)
	        .useBasicAuthenticationWithAccessCodeGrant(false)
	        .build();
	  }
	
	  @Bean
	  UiConfiguration uiConfig() {
	    return UiConfigurationBuilder.builder()
	        .deepLinking(true)
	        .displayOperationId(false)
	        .defaultModelsExpandDepth(1)
	        .defaultModelExpandDepth(1)
	        .defaultModelRendering(ModelRendering.EXAMPLE)
	        .displayRequestDuration(false)
	        .docExpansion(DocExpansion.NONE)
	        .filter(false)
	        .maxDisplayedTags(null)
	        .operationsSorter(OperationsSorter.ALPHA)
	        .showExtensions(false)
	        .tagsSorter(TagsSorter.ALPHA)
	        .validatorUrl(null)
	        .build();
	  }
	
	}

### 三、配置属性说明

#### 1）注解

**@EnableSwagger2**
> 启用Springfox Swagger 2

**@ComponentScan**
> 指定需要扫描的包

#### 2）Docket

**DocumentationType.SWAGGER_2**
> Docket是Springfox主要的api配置机制，其初始化是为了swagger2的规范定义

**select()**
> select方法返回一个ApiSelectorBuilder实例，对通过swagger公开的端点进行细粒度的控制。

**apis()**
> apis()方法需要一个类型为Predicate的参数，使用谓词来对请求进行处理，来选择需要控制的接口。示例使用的ANY，其余的还有any, none, withClassAnnotation, withMethodAnnotation 和 basePackage可选

**paths()**
> paths()方法需要一个类型为Predicate的参数，使用谓词选择路径。示例使用的any，其余的还有regex, ant, any, none可选。

**build()**
> build()方法是，在进行了api的配置，以及路径选择配置后，需要实际构建selector。实际这里是使用建造者模式，创建selector对象实例。

**pathMapping()**
> pathMapping()方法，当servlet有路径映射时，添加一个servlet路径映射。

**directModelSubstitute()**
