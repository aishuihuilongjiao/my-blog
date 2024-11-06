### 快速入门

#### 引入依赖

~~~~xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>

~~~~



#### 启用Swagger2

​	在启动类上或者配置类加 @EnableSwagger2 注解

~~~~java
@SpringBootApplication
@MapperScan("com.sangeng.mapper")
@EnableScheduling
@EnableSwagger2
public class SanGengBlogApplication {
    public static void main(String[] args) {
        SpringApplication.run(SanGengBlogApplication.class,args);
    }
}
~~~~



#### 测试

​	访问：http://localhost:7777/swagger-ui.html  注意其中localhost和7777要调整成实际项目的域名和端口号。

#### Controller配置

##### @Api 注解 

属性介绍：

tags  设置标签

description 设置描述信息

~~~~java
@RestController
@RequestMapping("/comment")
@Api(tags = "评论",description = "评论相关接口")
public class CommentController {
}
~~~~






####  接口配置

#####  接口描述配置@ApiOperation

~~~~java
    @GetMapping("/linkCommentList")
    @ApiOperation(value = "友链评论列表",notes = "获取一页友链评论")
    public ResponseResult linkCommentList(Integer pageNum,Integer pageSize){
        return commentService.commentList(SystemConstants.LINK_COMMENT,null,pageNum,pageSize);
    }
~~~~



##### 接口参数描述

 @ApiImplicitParam 用于描述接口的参数，但是一个接口可能有多个参数，所以一般与 @ApiImplicitParams 组合使用。

~~~~java
    @GetMapping("/linkCommentList")
    @ApiOperation(value = "友链评论列表",notes = "获取一页友链评论")
    @ApiImplicitParams({
           @ApiImplicitParam(name = "pageNum",value = "页号"),
           @ApiImplicitParam(name = "pageSize",value = "每页大小")
    }
    )
    public ResponseResult linkCommentList(Integer pageNum,Integer pageSize){
        return commentService.commentList(SystemConstants.LINK_COMMENT,null,pageNum,pageSize);
    }
~~~~



####  实体类配置

#####  实体的描述配置@ApiModel 

@ApiModel用于描述实体类。在这里引入dto，因为实体类已经跟mybatisplus绑定了，若在绑定，则维护成本会很高

~~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ApiModel(description = "添加评论dto")
public class AddCommentDto{
    //..
}
~~~~

 

##### 实体的属性的描述配置@ApiModelProperty

@ApiModelProperty用于描述实体的属性

~~~~java
    @ApiModelProperty(notes = "评论类型（0代表文章评论，1代表友链评论）")
    private String type;
~~~~
####  文档信息配置

~~~~java
@Configuration
public class SwaggerConfig {
    @Bean
    public Docket customDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.sangeng.controller"))
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("团队名", "http://www.my.com", "my@my.com");
        return new ApiInfoBuilder()
                .title("文档标题")
                .description("文档描述")
                .contact(contact)  
                .version("1.1.0")  
                .build();
    }
}
~~~~