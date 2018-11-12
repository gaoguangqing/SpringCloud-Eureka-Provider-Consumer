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
那么，注意点来了，你的生产者接受对象的话也必须要有@RequestBody，不然会报404
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
