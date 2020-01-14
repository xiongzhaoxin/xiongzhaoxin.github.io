---
title: java开发保证多次点击的数据一致（如何幂等）
date: 2020-1-14 23:18:45
tags:
---
# java开发保证多次点击的数据一致（如何幂等）

> 在我以前写过的一个管理系统的项目中，因为时间比较赶，所以没人注意到多次点击的问题，后续的修改bug时发现这个问题，于是百度查看了解决方法。使用Token机制，防止多次提交

**首先需要新增一个util工具类，名为TokenUtil：**

~~~java
public class TokenUtil {
	private static String cons = "@_____@token@________@";
 
	public static String setToken(javax.servlet.http.HttpServletRequest request) {
		javax.servlet.http.HttpSession session = request.getSession();
		String token = String.valueOf(java.util.UUID.randomUUID()) + String.valueOf(System.currentTimeMillis());
		session.setAttribute("@_____@token@________@", token);
		String html = "<input type=\"hidden\" name=\"@_____@token@________@\" value='" + token + "'/>";
		return html;
	}
 
	public static boolean isToken(javax.servlet.http.HttpServletRequest request) {
		javax.servlet.http.HttpSession session = request.getSession();
		String token = (String) session.getAttribute(cons);
		if (token == null)
			token = "";
		String tv = request.getParameter(cons);
 
		session.removeAttribute(cons);
		return token.equals(tv);
 
	}
}
~~~

我的情况是在jsp页面发送ajax请求，所以jsp页面也需要几行代码

**在jsp页面引入这个util工具类：（`以下代码是我的路径，复制后需要改成自己本机上这个类的路径`）**

```java
<%@ page import="com.muen.util.TokenUtils" %>
```

**接着需要在form表单中插入以下代码**

```java
<%=TokenUtils.setToken(request) %> 
```

**然后在controller层加入判断**

~~~java
@RequestMapping(value = "insert", method = RequestMethod.POST)
@ResponseBody
	public boolean insert(SysYhzc sysYhzc,HttpServletRequest request) {
		if (TokenUtils.isToken(request)) {
			return creditOrgService.insert(sysYhzc);
		}
		return false;
}
~~~

