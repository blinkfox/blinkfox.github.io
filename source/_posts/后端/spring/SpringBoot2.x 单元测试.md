---
title: SpringBoot2.x 单元测试
date: 2019-03-02 17:20:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2019/03/02/spring.png
top: true
categories: 后端
tags:
  - 单元测试
  - Java
  - Spring
  - SpringBoot
---

> 一个 bug 被隐藏的时间越长，修复这个 bug 的代价就越大。

我曾经在 [单元测试指南](https://blinkfox.github.io/2018/11/15/hou-duan/java/dan-yuan-ce-shi-zhi-nan/) 一文中写到过单元测试的必要性和 Java 单元测试相关的工具及方法。单元测试能帮助我们在早期就规避、发现和修复很多不易察觉的 bug 和漏洞，而且更能保障后期的需求变动和代码重构时所带来的隐患，减少测试成本和维护成本。在 SpringBoot2.x 集成和写单元测试更加容易了。

## 创建 SpringBoot2.x 项目

在 [start.spring.io](https://start.spring.io/) 中创建一个自己的 SpringBoot2.x 项目，目前版本`2.1.3`。选出自己需要的一些组件生成项目即可，我这里选了如下几个：

- `Web`: Web项目
- `JPA`: 数据库持久层采用[Spring Data JPA](https://spring.io/guides/gs/accessing-data-jpa/)，方便实用
- `Lombok`: 可以通过注解大量减少Java中重复代码的书写
- `HSQLDB`: 内存数据库，用来对 `Repository` 层做单元测试

生成之后可以在 `pom.xml` 中看到 SpringBoot2.x 项目中已经引入了`spring-boot-starter-test`这个启动组件，包含了几乎绝大多数测试场景需要的组件。然后通过`mvn clean install`来构建本项目或者直接导入 IDE 开发工具即可。

下面将以对博客信息做简单修改和查询为示例来说明在 Spring Boot 中如何分别对 `DAO`，`Service`，`Controller` 做单元测试。

## DAO 层的单元测试

### 新建数据库脚本

DAO 层的测试我这里采用的是 `HSQLDB` 的内存数据库，最好准备一些初始化的数据表结构和脚本，当然也可用直接通过官方示例的 `JPA`特性和 API 代码来初始化数据。这里我还是通过脚本的方式来做，便于统一管理和维护表结构和数据。

在 `src/test` 目录下新建 `resources` 资源目录，并在 `resources` 目录下新建 `db` 目录，在 `db` 目录下分别，新建用于管理的表结构文件(`schema.sql`)和初始化数据文件(`data.sql`)的 SQL 脚本。

`schema.sql` 文件中的内容如下：

```sql
-- 创建数据库表所在的模式 schema.
CREATE SCHEMA test;
commit;

-- 在 test 模式下创建数据库表.
DROP TABLE IF EXISTS test.t_test_blog;
CREATE TABLE test.t_test_blog (
    c_id varchar(32) NOT NULL,
    c_author varchar(255),
    c_content varchar(255),
    dt_publish_time timestamp(6) NULL,
    c_title varchar(255),
    c_url varchar(255),
    n_status int,
    c_create_user varchar(255),
    dt_create_time timestamp(6) NULL,
    dt_update_time timestamp(6) NULL,
    constraint pk_test_blog primary key(c_id)
);
commit;
```

`data.sql` 文件中的内容如下：

```sql
-- 初始化插入一些博客信息数据.
INSERT INTO test.t_test_blog VALUES ('1', '张三', '这是内容', '2019-03-01 00:41:01', 'Spring从入门到精通', 'https://baidu.com', '1', 'tom', '2019-03-01 00:41:33', '2019-03-01 00:41:36');
INSERT INTO test.t_test_blog VALUES ('2', '李四', '这是Mybatis的内容', '2019-03-01 00:41:01', 'Mybatis基础', 'https://qq.com', '2', 'jack', '2019-03-01 00:41:33', '2019-03-01 00:41:36');
commit;
```

### 增加 yaml 配置文件及内容

在 `resources` 目录下新建 `application-hsqldb.yml` 配置文件，用于存放 HSQLDB 及 JPA 相关的配置信息，主要配置内容如下：

```yaml
spring:
  datasource:
    url: jdbc:hsqldb:mem:db_test # 以内存数据库的方式来运行.
    username: root
    password: 123456
    driver-class-name: org.hsqldb.jdbc.JDBCDriver
    platform: hsqldb
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: none # 这里没用 JPA 的自动生成表结构等功能，你可以视自己的具体情况来开启.
    generate-ddl: false # 启动时是否初始化数据库.
```

### 准备实体 POJO 和 DAO 层 Repository 类

博客信息的实体 POJO 类如下：

```java
package com.blinkfox.springbootsample.pojo;

import java.util.Date;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.Getter;
import lombok.Setter;
import lombok.experimental.Accessors;

/**
 * 博客实体.
 *
 * @author blinkfox on 2019-2-26.
 */
@Getter
@Setter
@Accessors(chain = true)
@Entity
@Table(name = "t_test_blog", schema = "test")
public class Blog {

    /**
     * ID.
     */
    @Id
    @Column(name = "c_id")
    private String id;

    /**
     * 作者.
     */
    @Column(name = "c_author")
    private String author;

    /**
     * 标题.
     */
    @Column(name = "c_title")
    private String title;

    /**
     * 内容.
     */
    @Column(name = "c_content")
    private String content;

    /**
     * 发布时间.
     */
    @Column(name = "dt_publish_time")
    private Date publishTime;

    /**
     * 链接地址.
     */
    @Column(name = "c_url")
    private String url;

    /**
     * 状态.
     */
    @Column(name = "n_status")
    private Integer status;

    /**
     * 创建用户.
     */
    @Column(name = "c_create_user")
    private String createUser;

    /**
     * 创建时间.
     */
    @Column(name = "dt_create_time")
    private Date createTime;

    /**
     * 最后更新时间.
     */
    @Column(name = "dt_update_time")
    private Date updateTime;

}
```

下面是 `BlogRepository` 中的一个简单的自定义 `@Query` 查询，当然你也可以采用名称的规则来写本查询，我这里为了做示例，使用了 `@Query` 查询。

```java
package com.blinkfox.springbootsample.repository;

import com.blinkfox.springbootsample.pojo.Blog;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

/**
 * BlogRepository.
 *
 * @author blinkfox on 2019-02-27.
 */
@Repository
public interface BlogRepository extends JpaRepository<Blog, String> {

    @Query("SELECT b FROM Blog AS b WHERE b.title like 'Spring%'")
    List<Blog> querySpringBlogs();

}
```

### BlogRepository 的单元测试

然后在 Intellij IDEA 中通过 `Ctrl + Shift + T` 来为 `BlogRepository` 生成它对应的单元测试类 `BlogRepositoryTest`。

```java
package com.blinkfox.springbootsample.repository;

import com.blinkfox.springbootsample.pojo.Blog;

import java.util.List;
import java.util.Optional;
import javax.annotation.Resource;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * BlogRepositoryTest.
 *
 * @author blinkfox on 2019-03-01.
 */
@RunWith(SpringRunner.class)
@ActiveProfiles("hsqldb")
@DataJpaTest
public class BlogRepositoryTest {

    @Resource
    private BlogRepository blogRepository;

    /**
     * 测试新增博客的情况.
     */
    @Test
    public void save() {
        String id = "newblogId";
        String title = "Java 从入门到放弃";
        blogRepository.save(new Blog().setId(id).setTitle(title));

        Optional<Blog> blogOptional = blogRepository.findById(id);
        Assert.assertTrue(blogOptional.isPresent() && title.equals(blogOptional.get().getTitle()));
    }

    /**
     * 测试查询所有 Spring 相关的博客信息.
     */
    @Test
    public void querySpringBlogs() {
        List<Blog> blogs = blogRepository.querySpringBlogs();
        Assert.assertEquals(1, blogs.size());
        Assert.assertEquals("Spring从入门到精通", blogs.get(0).getTitle());
    }

}
```

这样就完成了 DAO 层代码的测试，以上程序主要依托于内存数据库 HSQLDB 和 Spring Data JPA。

## Service 层的单元测试

实际开发过程中，Service 层中的类依赖了 DAO 层中的类或其他 Service 类。为了隔离对其他 Service 类或 DAO 层中的类的依赖，只测试本 Service 类中的方法逻辑，就需要 Mock 数据和做打桩等操作。Spring Boot 中原生集成了 [Mockito](https://site.mockito.org/)，可以非常方便我们对 Java 代码做单元测试。

### 新建 BlogService 类

```java
package com.blinkfox.springbootsample.service;

import com.blinkfox.springbootsample.pojo.Blog;
import com.blinkfox.springbootsample.repository.BlogRepository;

import java.util.List;
import java.util.Optional;
import javax.annotation.Resource;

import lombok.extern.slf4j.Slf4j;

import org.springframework.stereotype.Service;

/**
 * BlogService.
 *
 * @author blinkfox on 2019-03-01.
 */
@Slf4j
@Service
public class BlogService {

    @Resource
    private BlogRepository blogRepository;

    /**
     * 查询所有 Spring 相关的博客信息.
     *
     * @return 博客信息
     */
    public List<Blog> getSpringBlogs() {
        log.info("进入了获取 Spring 相关博客的 Service 方法.");
        return blogRepository.querySpringBlogs();
    }

    /**
     * 根据博客ID来修改该博客的名称.
     *
     * @param id 博客ID
     * @param title 博客标题
     */
    public void modifyTitileById(String id, String title) {
        Optional<Blog> blogOptional = blogRepository.findById(id);
        if (!blogOptional.isPresent()) {
            log.warn("需要修改名称的博客不存在，id为【{}】请检查！", id);
            return;
        }

        blogRepository.save(blogOptional.get().setTitle(title));
    }

}
```

### BlogService 的单元测试

通过 `BlogService` 可以生成和书写出其对应的单元测试类和测试方法，代码如下：

```java
package com.blinkfox.springbootsample.service;

import com.blinkfox.springbootsample.pojo.Blog;
import com.blinkfox.springbootsample.repository.BlogRepository;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.MockitoJUnitRunner;

/**
 * BlogServiceTest.
 *
 * @author blinkfox on 2019-03-01.
 */
@RunWith(MockitoJUnitRunner.class)
public class BlogServiceTest {

    @Mock
    private BlogRepository blogRepository;

    @InjectMocks
    private BlogService blogService;

    /**
     * 测试service层中获取Spring相关博客的方法.
     */
    @Test
    public void getSpringBlogs() {
        // 构造需要返回的博客信息集合数据.
        Blog blog = new Blog()
                .setId("1")
                .setTitle("Spring Action");
        List<Blog> blogList = new ArrayList<>();
        blogList.add(blog);

        Mockito.when(blogRepository.querySpringBlogs())
                .thenReturn(blogList);
        List<Blog> blogs = blogService.getSpringBlogs();

        // 断言验证查询到的数据.
        Assert.assertEquals(1, blogs.size());
        Assert.assertEquals("Spring Action", blog.getTitle());
    }

    /**
     * 测试根据博客ID来修改该博客的名称成功时的情况.
     */
    @Test
    public void modifyTitileById() {
        // Mock 相关数据和类方法的行为.
        String id = "1";
        Mockito.when(blogRepository.findById(id))
                .thenReturn(Optional.of(new Blog()));
        Mockito.when(blogRepository.save(Mockito.any()))
                .thenReturn(new Blog());

        blogService.modifyTitileById(id, "算法导论");

        // 验证 blogRepository.save(s) 方法被调用过一次.
        Mockito.verify(blogRepository).save(Mockito.any());
    }

    /**
     * 测试根据博客ID来修改该博客的名称失败时的情况.
     */
    @Test
    public void modifyTitileByIdWithFailure() {
        // Mock 未根据 ID 找到对应的博客信息的情况.
        String id = "1";
        Mockito.when(blogRepository.findById(id))
                .thenReturn(Optional.ofNullable(null));

        blogService.modifyTitileById(id, "算法导论");

        // 验证 blogRepository.save(s) 方法并没有被调用过.
        Mockito.verify(blogRepository, Mockito.never()).save(Mockito.any());
    }

}
```

> **注意**：这里的 `@RunWith` 采用的是 Mocktio 提供的 `MockitoJUnitRunner`。

这样就完成了 Service 层的单元测试，也是我们业务开发中需要重点关注和测试业务逻辑的一层。

## Controller 层的单元测试

Controller 层测试的重点是测试接口是否能正常工作。可以用到 Spring Boot 中提供的 `@WebMvcTest` 注解来模拟 Web 层的单元测试。当然，也需要通过 Mock 的方式类隔离对 Service 层各个类的依赖影响。

### 新建 BlogController 类

```java
package com.blinkfox.springbootsample.controller;

import com.blinkfox.springbootsample.pojo.Blog;
import com.blinkfox.springbootsample.service.BlogService;

import java.util.List;
import javax.annotation.Resource;

import lombok.extern.slf4j.Slf4j;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PatchMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * BlogController.
 *
 * @author blinkfox on 2019-02-28.
 */
@Slf4j
@RequestMapping("/blogs")
@RestController
public class BlogController {

    @Resource
    private BlogService blogService;

    /**
     * 获取所有 Spring 相关的博客信息.
     *
     * @return Spring相关的博客信息
     */
    @GetMapping
    public ResponseEntity<List<Blog>> getSpringBlogs() {
        return ResponseEntity.ok(blogService.getSpringBlogs());
    }

    /**
     * 根据博客ID修改博客名称.
     *
     * @param id 博客ID
     * @param title 博客标题
     * @return 空
     */
    @PatchMapping("/{id}")
    public ResponseEntity<Void> modifyTitileById(@PathVariable("id") String id,
            @RequestParam("title") String title) {
        try {
            blogService.modifyTitileById(id, title);
            log.info("修改博客名称成功.");
            return new ResponseEntity<>(HttpStatus.OK);
        } catch (Exception e) {
            log.error("修改博客名称出错，id为【{}】.", id);
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

}
```

### BlogController 的单元测试

通过 `BlogController` 可以生成和书写出其对应的单元测试类和测试方法，代码如下：

```java
package com.blinkfox.springbootsample.controller;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.patch;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import com.blinkfox.springbootsample.service.BlogService;

import java.util.ArrayList;
import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

/**
 * BlogControllerTest.
 *
 * @author blinkfox on 2019-03-02.
 */
@RunWith(SpringRunner.class)
@WebMvcTest(BlogController.class)
public class BlogControllerTest {

    @Resource
    private MockMvc mockMvc;

    @MockBean
    private BlogService blogService;

    /**
     * 测试获取所有 Spring 相关的博客信息.
     *
     * @throws Exception 异常
     */
    @Test
    public void getSpringBlogs() throws Exception {
        Mockito.when(blogService.getSpringBlogs())
                .thenReturn(new ArrayList<>());

        this.mockMvc.perform(get("/blogs"))
                .andExpect(status().isOk());
    }

    /**
     * 测试修改博客标题成功时的情况.
     *
     * @throws Exception 异常
     */
    @Test
    public void modifyTitileById() throws Exception {
        Mockito.doNothing()
                .when(blogService).modifyTitileById(Mockito.anyString(), Mockito.anyString());
        this.mockMvc.perform(patch("/blogs/1?title=Spring实战"))
                .andExpect(status().isOk());
    }

    /**
     * 测试修改博客标题失败时的情况.
     *
     * @throws Exception 异常
     */
    @Test
    public void modifyTitileByIdWithException() throws Exception {
        Mockito.doThrow(RuntimeException.class)
                .when(blogService).modifyTitileById(Mockito.anyString(), Mockito.anyString());
        this.mockMvc.perform(patch("/blogs/1?title=Spring实战"))
                .andExpect(status().is5xxServerError());
    }

}
```

以上就完成了对 Controller 层的单元测试。

## 总结

在 Spring Boot 中做单元测试的将会非常容易。上面只是 Spring Boot 中提供的部分方式，[Spring Boot 文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing) 中还有其他更多的测试场景和测试方法供你去参考和使用。