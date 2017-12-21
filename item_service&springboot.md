
## 商品服务

#### 为什么要有商品服务？
1. 基于淘宝现有商品接口做服务增强，如区间价格搜索,商家编码搜索, include(numIid,title) exclude(numIid,title)等
2. 基于缓存cache，减少api的调用量。



#### 商品服务的演变
   
>    第一阶段： 满足促销工具的内部商品检索(内置cache for 检索)。
   
>    第二阶段：要满足其他业务(海报 水印之类的)，抽离成专用服务ItemSearcher(定时和淘宝做增量同步对比)。
   
>    第三阶段：接入淘宝商品变动通知更新商品信息。hash一致的优化。

>    第四阶段：接入御膳房统计数据，merge 御膳房商品数据信息。支持商品销量收藏量排序。


#### 接口查询支持

```
    private String sellerCids;//商品分类
	private String title;//商品标题
	private String excludeTitle;//排除商品标题
	private String numIids;//包含商品
	private String excludeNumIids;//排除商品Id
	private String outerId;//商家编码
	private String startPrice;//最低价
	private String endPrice;//最高价
	private Boolean showcase;//是否橱窗推荐
	private String postageId;//邮费id
	private ItemStatus itemStatus//上下状态
	private OrderBy orderBy;//排序字段
	private Page page;//分页信息
```



#### 商品服务的缓存查询流程
   1. 缓存全部商品: 快速返回数据,服务降级，并发控制。
   2. 缓存商品List<Item>, 相关索引的建立。
   3. 查询查询cache，优先索引，剩余的 for;
   
#### 商品服务的优化
   1. hash一致算法(分而治之)
   2. 索引延时建立。
   3. 缓存接入淘宝主动通知。
   4. 时间切分查询。
   


## dubbo 原理剖析
   
   补充
   
   
   
   

# nginx - web 高并发必备


补充

1. If-Modified-Since


# spring boot - 四大神器

> https://projects.spring.io/spring-boot/
> https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

package hello;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@Controller
@EnableAutoConfiguration
public class SampleController {

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleController.class, args);
    }
}
```

1. auto configuration
    * @EnableAutoConfiguration & **@Conditional** 原理
        * @ConditionalOnBean（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
        * @ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean
        * @ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
        * @ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
        * @ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
        * @ConditionalOnNotWebApplication（不是web应用）
    * devtools & jRebel IDEA 插件
    * @ConfigurationProperties vs. @Value 弊端
        * 默认值不明确，可以随意指定
        * 到处引用，没法知道系统究竟有什么可以配置
        * 命名规则不统一
    * 配置文件优先级从高到低
        * 当前目录 config 目录
        * 当前目录
        * classpath config 目录
        * classpath root
    * 配置尽量体现在代码中，通过 -Dspring.profiles.active=staging 指定不同环境下的配置,
        * 开发环境 - application.properties
        * 单元测试 - application-unit.properties
        * 测试环境 - application-staging.properties
        * 生产环境 - application-prod.properties
    * server, jackson, jsr310 ...
    
2. starters
    * spring-boot-starter-tomcat (spring-boot-starter-jetty)
    * spring-boot-starter-web
    * spring-boot-starter-websocket
    * spring-boot-starter-security
    * spring-boot-starter-data-jpa
    * spring-boot-starter-data-redis
    * spring-boot-starter-data-elasticsearch
3. cli - 没用到
    * bin/launch
    * config/application-{profile}.properties, config/launch.conf（ web 和 updater 关注点不一样使用不同垃圾收集器）
    * out.println Main.class.getPackage().getImplementationVersion() - 检查下每个结点部署的版本是否一致


    ```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
            <archive>
                <manifestEntries>
                    <Specification-Version>${project.version}</Specification-Version>
                    <Implementation-Version>${env.GIT_COMMIT}.b${env.BUILD_NUMBER}</Implementation-Version>
                </manifestEntries>
            </archive>
        </configuration>
    </plugin>
    
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
                <mainClass>com.darcytech.poseidon.updater.UpdaterMain</mainClass>
            </configuration>
        </plugin>
    </plugins>    
    ```

4. actuator - 用处不大
    * /autoconfig	查看自动配置的使用情况
    * /configprops	查看配置属性，包括默认配置
    * /beans	    查看bean及其关系列表
    * /dump	        打印线程栈
    * /env	        查看所有环境变量
    * /env/{name}	查看具体变量值
    * /health 	    查看应用健康指标
    * /info	        查看应用信息
    * /mappings	    查看所有url映射
    * /metrics   	查看应用基本指标
    * /metrics/{name}	查看具体指标
    * /shutdown	    关闭应用
    * /trace    	查看基本追踪信息
    
    
    
    
        default Page<Hotel> findOnlineFalse(Pageable pageable, long userId, String name) {
        Specifications<Hotel> spec = Specifications.where((root, query, cb) -> {
            Subquery<UserHotel> sq = query.subquery(UserHotel.class);
            Root<UserHotel> sr = sq.from(UserHotel.class);
            sq.select(sr.get("id"));
            sq.where(cb.and(cb.equal(sr.get(UserHotel_.online), true),
                    cb.equal(sr.get(UserHotel_.userId), userId),
                    cb.equal(root.get(Hotel_.id), sr.get(UserHotel_.hotel))));
            return cb.not(cb.exists(sq));
        });

        if (StringUtils.isNotBlank(name)) {
            spec = spec.and((root, query, cb) -> cb.like(root.get(Hotel_.name), name + "%"));
        }
        return findAll(spec, pageable);
    }

   