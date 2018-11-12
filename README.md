# SpringCloud-Eurka-Provider-Consumer
SpringCloud-Feign-Eurka-MyBatisPlus

如果你有SSM的User,那么你可以把它作为提供者，但是你需要添加如下参数
你需要在application.yml中注册Eurka
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://user:password@localhost:8761/eureka
```
而且你的启动类要有 @EnableEurekaClient
```
@SpringBootApplication
@MapperScan(basePackages="com.spoon.mapper")
@EnableEurekaClient
public class RunAppProviderUser {

	public static void main(String[] args) {
		SpringApplication.run(RunAppProviderUser.class, args);
	}
}
```

那么你使用Feign的时候，消费者的Controller可以这么写
```
package com.spoon.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.spoon.feign.UserFeign;
import com.spoon.pojo.User;

@RestController
@RequestMapping("/user")
public class UserController {
	@Autowired
	private UserFeign userFeign;
	
	@RequestMapping("/findAll")	//必须路径写全
	public List<User> findAll(){
		return userFeign.findAll();
	}
	
	@RequestMapping("/find/{name}")
	public List<User> find(User user){
		return userFeign.find(user);
	}
	
	@RequestMapping("/insert/{name}/{address}/{birthday}")
	public String insert(User user){
		return userFeign.insert(user);
	}
	
	@RequestMapping("/update/{name}/{id}")
	public String update(User user){
		return userFeign.update(user);
	}
	
	@RequestMapping("/delete/{id}")
	public String delete(@PathVariable Integer id){
		return userFeign.delete(id);
	}
}
```
Feign接口就应该这么写
```
package com.spoon.feign;

import java.util.List;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;

import com.spoon.pojo.User;

@FeignClient("user-provider")
public interface UserFeign {
	@RequestMapping("/user/findAll")	//必须路径写全
	public List<User> findAll();		//POJO中的日期的处理
	
	@RequestMapping("/user/find")
	public List<User> find(@RequestBody User user);
	
	@RequestMapping("/user/insert")		//对象传参，内部走的是json提交
	public String insert(@RequestBody User user);
	
	@RequestMapping("/user/update")		//转换成json方式，接参
	public String update(@RequestBody User user);
	
	@RequestMapping("/user/delete/{id}")
	public String delete(@PathVariable("id") Integer id);
}
```
那么，注意点来了，你的生产者接收对象类型的参数的话，必须要有@RequestBody，不然会报404
```
package com.spoon.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.spoon.pojo.User;
import com.spoon.service.UserService;

@RestController
@RequestMapping("/user")
public class UserController {
	@Autowired
	private UserService userService;
	
	@RequestMapping("/findAll")
	public List<User> findAll() {
		return userService.findAll();
	}
	@RequestMapping("/find")
	public List<User> find(@RequestBody User user) {
		return userService.find(user);
	}
	@RequestMapping("/insert")
	public String insert(@RequestBody User user) {
		try {
			System.out.println(user);
			userService.insert(user);
			return "插入成功";
		} catch (Exception e) {
			e.printStackTrace();
			return "插入失败";
		}
	}
	@RequestMapping("/update")
	public String  update(@RequestBody User user) {
		try {
			userService.update(user);
			return "修改成功";
		} catch (Exception e) {
			e.printStackTrace();
			return "修改失败";
		}
	}
	@RequestMapping("/delete/{id}")
	public String delete(@PathVariable Integer id) {
		try {
			userService.delete(id);
			return "删除成功";
		} catch (Exception e) {
			e.printStackTrace();
			return "删除失败";
		}
	}
}
```
使用Feign还需要注意时间的问题，如果你的提供者者有Date日期类型属性的pojo，如下
```
package com.spoon.pojo;

import java.io.Serializable;
import java.util.Date;

import org.springframework.format.annotation.DateTimeFormat;

import com.baomidou.mybatisplus.annotations.TableId;
import com.baomidou.mybatisplus.annotations.TableName;
import com.baomidou.mybatisplus.enums.IdType;
@TableName("user")
public class User implements Serializable{
	/**
	 * 表和类映射
	 * select 字段* from @TableName
	 */
	private static final long serialVersionUID = 1L;
	//主键,自增主键策略
	@TableId(type=IdType.AUTO)
	private Integer id;
	//映射，全局配置驼峰规则，MyBatisPlus自动修改
	private String name;
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date birthday;
	private String address;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	
	
	public Date getBirthday() {
		return birthday;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + ", birthday=" + birthday + ", address=" + address + "]";
	}
	
}
```
那么为了避免Feign在转换时间上的不准确，你消费者的日期类型的属性要使用String字符串类型
```
package com.spoon.pojo;

import java.io.Serializable;

public class User implements Serializable{
	private static final long serialVersionUID = 1L;
	private Integer id;
	//映射，全局配置驼峰规则，MybatisPlus自动修改
	private String name;
	
	private String birthday;
	
	private String address;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}

	public String getBirthday() {
		return birthday;
	}
	public void setBirthday(String birthday) {
		this.birthday = birthday;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	
}

```
## 运行结果
http://localhost:9001/user/findAll
```
2018-11-13 02:07:29.607 DEBUG 11200 --- [nio-7900-exec-3] com.spoon.mapper.UserMapper.selectList   : ==>  Preparing: SELECT id AS id,`name`,birthday,address FROM user 
2018-11-13 02:07:29.608 DEBUG 11200 --- [nio-7900-exec-3] com.spoon.mapper.UserMapper.selectList   : ==> Parameters: 
2018-11-13 02:07:29.612 DEBUG 11200 --- [nio-7900-exec-3] com.spoon.mapper.UserMapper.selectList   : <==      Total: 5
```
```
[
  {
    "id": 1,
    "name": "张三",
    "birthday": -12527251200000,
    "address": "北京"
  },
  {
    "id": 2,
    "name": "李四",
    "birthday": -12086352000000,
    "address": "上海"
  },
  {
    "id": 3,
    "name": "王五",
    "birthday": -12306412800000,
    "address": "广州"
  },
  {
    "id": 4,
    "name": "赵六",
    "birthday": -12748176000000,
    "address": "深圳"
  },
  {
    "id": 5,
    "name": "钱七",
    "birthday": -13000636800000,
    "address": "厦门"
  }
]
```
http://localhost:9001/user/find/赵
```
2018-11-13 02:08:41.785 DEBUG 11200 --- [nio-7900-exec-5] com.spoon.mapper.UserMapper.selectList   : ==>  Preparing: SELECT id AS id,`name`,birthday,address FROM user WHERE (name LIKE ?) 
2018-11-13 02:08:41.787 DEBUG 11200 --- [nio-7900-exec-5] com.spoon.mapper.UserMapper.selectList   : ==> Parameters: %赵%(String)
2018-11-13 02:08:41.790 DEBUG 11200 --- [nio-7900-exec-5] com.spoon.mapper.UserMapper.selectList   : <==      Total: 1
```
```
[
  {
    "id": 4,
    "name": "赵六",
    "birthday": "-12748176000000",
    "address": "深圳"
  }
]
```
http://localhost:9001/user/insert/小美/东莞/1994-05-07
```
User [id=null, name=小美, birthday=Sat May 07 08:00:00 CST 1994, address=东莞]
2018-11-13 02:10:38.077 DEBUG 11200 --- [nio-7900-exec-7] com.spoon.mapper.UserMapper.insert       : ==>  Preparing: INSERT INTO user ( `name`, birthday, address ) VALUES ( ?, ?, ? ) 
2018-11-13 02:10:38.079 DEBUG 11200 --- [nio-7900-exec-7] com.spoon.mapper.UserMapper.insert       : ==> Parameters: 小美(String), 1994-05-07 08:00:00.0(Timestamp), 东莞(String)
2018-11-13 02:10:38.086 DEBUG 11200 --- [nio-7900-exec-7] com.spoon.mapper.UserMapper.insert       : <==    Updates: 1
2018-11-13 02:10:54.357  INFO 11200 --- [trap-executor-0] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
```
```
插入成功
```
http://localhost:9001/user/update/张三丰/1
```
2018-11-13 02:13:17.220 DEBUG 11200 --- [nio-7900-exec-9] com.spoon.mapper.UserMapper.updateById   : ==>  Preparing: UPDATE user SET `name`=? WHERE id=? 
2018-11-13 02:13:17.223 DEBUG 11200 --- [nio-7900-exec-9] com.spoon.mapper.UserMapper.updateById   : ==> Parameters: 张三丰(String), 1(Integer)
2018-11-13 02:13:17.231 DEBUG 11200 --- [nio-7900-exec-9] com.spoon.mapper.UserMapper.updateById   : <==    Updates: 1
```
```
修改成功
```
http://localhost:9001/user/delete/6
```
2018-11-13 02:15:15.096 DEBUG 11200 --- [nio-7900-exec-1] c.s.mapper.UserMapper.deleteBatchIds     : ==>  Preparing: DELETE FROM user WHERE id IN ( ? ) 
2018-11-13 02:15:15.096 DEBUG 11200 --- [nio-7900-exec-1] c.s.mapper.UserMapper.deleteBatchIds     : ==> Parameters: 6(Integer)
2018-11-13 02:15:15.101 DEBUG 11200 --- [nio-7900-exec-1] c.s.mapper.UserMapper.deleteBatchIds     : <==    Updates: 1
```
```
删除成功
```
