---
layout: post
title:  "SpringMVC-crud"
date:   2018-4-10
excerpt: "刚刚接触springmvc框架，简单配置"
image: "/images/springmvc.png"
---

## SpringMVC-CRUD

## web.xml
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
 
	<配置：把post请求可以更改为delete和put请求>
	<filter>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	</web-app>

## Spring配置文件
	<context:component-scan base-package="com.chen.springmvc"></context:component-scan>
	< 配置视图解析器>
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<mvc:default-servlet-handler表示如果你访问的是一个没有映射的资源
	首先会按照静态资源处理，如果没找到对应的静态资源
	在找是否映射过，若有则访问若无则404 -->
	<mvc:default-servlet-handler/>
	<mvc:annotation-driven></mvc:annotation-driven>
	
## index页面先超链接到emps对应handler如下
	@RequestMapping("/emps")
		public String list(Map<String,Object> map){	
			map.put("employees", employeeDao.getAll());
			return "list";
		}
	
		通过spring的配置文件以上方法会映射到/WEB-INF/views/list.jsp
	
	
## list.jsp
	<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	<title>Insert title here</title>
	< springmvc处理静态资源 比如jquery的资源文件 springnvc会按照请求处理但是找不到映射>
	<解决方案：在springmvc配置文件中配置mvc:default-servlet-handler>
	并加入<mvc:annotation-driven></mvc:annotation-driven>
	< 因为只有post请求才可以利用web的配置文件将其改变为delete或者put请求
	所以要引入jquery将a标签的get请求转换为post请求-->
	<script type="text/javascript" src="script/jquery-3.3.1.js"></script>
	<script type="text/javascript">
		$(function(){
			$(".delete").click(function(){
				var href=$(this).attr("href");
				$("form").attr("action",href).submit();
			
			return false;
		});
	})
</script>
</head>
<body>

	<form action="" method="post">
		<input type="hidden" name="_method"  value="DELETE"/>
	</form>

	<c:if test="${empty requestScope.employees }">
	没有任何员工信息
	</c:if>
	
	<c:if test="${!empty requestScope.employees }">
		<table border="1" cellpadding="10" cellspacing="0">  
			<tr>
				<th>ID</th>
				<th>LASTNAME</th>
				<th>EMAIL</th>
				<th>GENDER</th>
				<th>DEPARTMENT</th>
				<th>EDIT</th>
				<th>DELETE</th>
			</tr>
			<c:forEach items="${requestScope.employees}" var="emp">
				<tr>
					<td>${emp.id}</td>
					<td>${emp.lastName}</td>
					<td>${emp.email}</td>
					<td>${emp.gender==0?'Female':'Male'}</td>
					<td>${emp.department.departmentName}</td>
					<td><a href="emp/${emp.id }"> Edit</a></td>
					<td><a class="delete" href="emp/${emp.id }"> Delete</a></td>
				</tr>
			</c:forEach>
		</table>
	</c:if>
	<br><br>
	<a href="emp">Add new employee</a>
</body>
</html>

## input.jsp

<!-- 使用springmvc的表单标签 -->
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<!--使用springmvc的表单标签可以更方便的开发页面而且回显更简单
这里必须注意form的表单属性必须设置一个modelAttribute="employee"属性
默认是command,他会自动去ioc中寻找这个bean没有会报错
同时改完之后我们还需要在Handler对应的方法中添加如下：
     map.put("employee", new Employee());
  -->
<form:form action="${pageContext.request.contextPath}/emp" method="post"  modelAttribute="employee">
	<!-- path属性对应Html表单的name属性 -->
	
	<c:if test="${employee.id==null}">
		LastName:<form:input path="lastName"/><br>
	</c:if>
	<c:if test="${employee.id!=null}">
		<form:hidden path="id"/>
		<input type="hidden" name="_method"  value="PUT"/>
	</c:if>
	
	Email:<form:input path="email"/><br>
	<%
		Map<String,String> genders= new HashMap();
		genders.put("1", "Male");
		genders.put("0", "Female");
		request.setAttribute("genders", genders);
	%>
	Gender:<form:radiobuttons path="gender" items="${genders}" /><br>
	Department:<form:select path="department.id" 
				items="${departments}" itemLabel="departmentName" itemValue="id"></form:select>
	<input type="submit" value="Submit"/>
</form:form>

## handler

package com.chen.springmvc.handlers;

import java.util.Map;

import javax.security.auth.message.callback.PrivateKeyCallback.Request;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

import com.chen.springmvc.dao.DepartmentDao;
import com.chen.springmvc.dao.EmployeeDao;
import com.chen.springmvc.entities.Employee;

@Controller
public class EmployeeHandler {
	
	@Autowired
	private EmployeeDao employeeDao;
	
	@Autowired
	private DepartmentDao departmentDao;
	
	//解决lastName为空
	@ModelAttribute
	public void getEmployee(@RequestParam(value="id",required=false) Integer id,
			Map<String,Object>map){
		if(id!=null){
			map.put("employee", employeeDao.get(id));
		}
	}
	
	
	@RequestMapping(value="/emp",method=RequestMethod.PUT)
	public String update(Employee employee){
		employeeDao.save(employee);
		
		return "redirect:/emps";
	}
	
	
	@RequestMapping(value="/emp/{id}",method=RequestMethod.GET)
	public String input(@PathVariable("id") Integer id,Map<String,Object> map){
		
		map.put("employee", employeeDao.get(id));
		map.put("departments", departmentDao.getDepartments());
		return "input";
	}
	
	@RequestMapping(value="/emp/{id}" ,method=RequestMethod.DELETE)
	public String delete(@PathVariable("id") Integer id){
		employeeDao.delete(id);
		return "redirect:/emps";
	}
	
	@RequestMapping(value="/emp" ,method=RequestMethod.POST)
	public String sava(Employee employee){
		employeeDao.save(employee);
		return "redirect:/emps";
	}
	
	@RequestMapping(value="/emp",method=RequestMethod.GET)
	public String input(Map<String,Object> map){
		map.put("departments",departmentDao.getDepartments());
		map.put("employee", new Employee());
		return "input";
	}
	@RequestMapping("/emps")
	public String list(Map<String,Object> map){	
		map.put("employees", employeeDao.getAll());
		return "list";
	}
}

















	


