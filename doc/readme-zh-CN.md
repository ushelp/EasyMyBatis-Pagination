## EasyMyBatis Pagination

EasyMyBatis Pagination 是一个针对 MyBaits 框架的通用分页插件。提供 PageBean 自动分页数据封装, EasyCriteria 分页条件对象，支持基于` Mappers 接口`和 `SQLID` 两种方式的数据库的自动化分页 SQL。

最新版本: `1.9.1-RELEASE`

MyBatis: `3.4.0+`

## Maven

```XML
<dependency>
	<groupId>cn.easyproject</groupId>
	<artifactId>easymybatis-pagination</artifactId>
	<version>1.9.1-RELEASE</version>
</dependency>
```

## 使用步骤

1. 在 `mybatis-config.xml` 配置 **EasyMybatisPaginationPlugin** 插件

 ```XML
 <plugins>
	<plugin interceptor="cn.easyproject.easymybatis.pagination.EasyMybatisPaginationPlugin">
		<!-- 默认方言; 可选 -->
		<!-- 也可以通过 'pageBean.setDialect(PageBean.XXXX_DIALECT)' 指定 -->
		<!-- Value: ORACLE, ORACLE_12C, SQLSERVER, SQLSERVER_2012, MYSQL -->
		<property name="dialect" value="MYSQL" />
	</plugin>
 </plugins>
 ```
 
2. DAO

 - **DAO interface:**
 
	 ```JAVA
	 package com.company.project.dao;
	
	 import cn.easyproject.easymybatis.pagination.PageBean;
	
	 public class AccountDAO{
	 	public List pagination(PageBean pageBean);
	 	// ...
	 }
	 ```
	 
 - **SQL Mapper:**
 	
 	```XML
 	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<mapper namespace="com.company.project.dao.AccountDAO">
 	
	 	<select id="pagination" resultType="Account">
	 		${autoSQL}
	 	</select>
 		...
	</mapper>
 	```

## PageBean 分页

- 示例一
 ```JAVA
 SqlSession session=MyBatisSessionFactory.getSession();

 PageBean pb=new PageBean();
 // SELECT 语句; 可选; 默认为 *
 pb.setSelect("*"); 
 // From 语句; 必须
 pb.setFrom("Account account");
 // WHERE 语句; 可选; 默认为 ''
 pb.setCondition(" and account.qxid>=10");
 // 追加 WHERE 条件; 可选; 默认为 ''
 //pb.addCondition(""); 
 // 排序列名; 可选; 默认为 ''
 pb.setSort("account.accountid");
 // 排序方式; 可选; 默认为 'asc'
 pb.setSortOrder("desc");
 // 当前页数; 可选; 默认为 1
 pb.setPageNo(1);
 // 每页条数; 可选; 默认为 10
 pb.setRowsPerPage(4);

 // 查询方式一: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // 查询方式二: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);
 
 // Pagination data
 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount());
 ```

- 示例二
 ```JAVA
 PageBean<Qx> pb=new PageBean<Qx>();
 pb.setPageNo(2);
 pb.setRowsPerPage(5);
 // data sql
 pb.setSql("select * from Account where accountId<=80 and accountName=? limit 5,5"); 
 // total sql
 pb.setCountSQL("select count(*) from Account where accountId<=80 and accoutName=?"); 

 // Set parameter values
 Map<String, Object> values=new HashMap<String,Object>();
 values.put("accoutName", "%a%");
 pb.setSqlParameterValues(values);

 // 查询方式一: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // 查询方式二: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);

 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount());
 ```
 
- 示例三

  多表查询必须指定事实表别名: `pageBean.setPrimaryTable();`

 ```JAVA
 PageBean pb=new PageBean();
 // SELECT 语句; 可选; 默认为 *
 pb.setSelect("*"); 
 // From 语句; 必须
 pb.setFrom("Account account, AccountType type");
 // 多表查询必须指定事实表别名
 pageBean.setPrimaryTable("account");
 // WHERE 语句; 可选; 默认为 ''
 pb.setCondition(" and account.typeId=type.id and account.qxid>=10");
 // 排序列名; 可选; 默认为 ''
 pb.setSort("account.accountid");
 // 排序方式; 可选; 默认为 'asc'
 pb.setSortOrder("desc");
 // 当前页数; 可选; 默认为 1
 pb.setPageNo(1);
 // 每页条数; 可选; 默认为 10
 pb.setRowsPerPage(4);

 // 查询方式一: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // 查询方式二: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);

 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount());
 ```
  
 
## EasyCriteria 条件查询

1. 创建 EasyCriteria 类, 必须 `extends EasyCriteria implements Serializable`

2. 编写条件方法 `getCondition()`


 ```JAVA
 public class SysUserCriteria extends EasyCriteria implements java.io.Serializable {
 	// 1. Criteria field
 	private String name;
 	private String realName;
 	private Integer status; // 0 is ON; 1 is OFF; 2 is REMOVED
 
 	// 2. Constructor
 	public SysUserCriteria() {
 	}
 
 	public SysUserCriteria(String name, String realName, Integer status) {
 		super();
 		this.name = name;
 		this.realName = realName;
 		this.status = status;
 	}
 
 	// 3. Condition genertator abstract method implements
 	public String getCondition() {
 		values.clear(); //*Must clear old values**
 		
 		StringBuffer condition = new StringBuffer();
 		if (StringUtils.isNotNullAndEmpty(this.getName())) {
 			condition.append(" and name like #{name}");
 			values.put("name", "%" + this.getName() + "%");
 		}
 		if (StringUtils.isNotNullAndEmpty(this.getRealName())) {
 			condition.append(" and realName like #{realName}");
 			values.put("realName", "%" + this.getRealName() + "%");
 		}
 		if (StringUtils.isNotNullAndEmpty(this.getStatus())) {
 			condition.append(" and status=#{status}");
 			values.put("status" ,this.getStatus());
 		}
 		return condition.toString();
 	}
 
 	// 4. Setters&amp;Getters...
 
 }
 ```

3. 在 PageBean 中使用 EasyCriteria

 ```JAVA
 // EasyCriteria
 SysUserCriteria sysUserCriteria=new SysUserCriteria();
 sysUserCriteria.setName("A");
 sysUserCriteria.setStatus(0);
 
 PageBean pb=new PageBean();
 // Set EasyCriteria
pb.setEasyCriteria(sysUserCriteria);

 pb.setSelect("*");
 pb.setFrom("SysUser sysUser");
 pb.setSort("sysUser.userid");
 pb.setSortOrder("desc");
 pb.setCondition(" and sysUser.userid>=10");
 //	pb.addCondition(""); // Append condition

 pb.setPageNo(1);
 pb.setRowsPerPage(3);
 
 // Use someway query by PageBean...
 // ...
 ```




## END
### [官方主页](http://www.easyproject.cn/easymybatispagination/zh-cn/index.jsp '官方主页')

[留言评论](http://www.easyproject.cn/easymybatispagination/zh-cn/index.jsp#donation '留言评论')

如果您有更好意见，建议或想法，请联系我。

### [The official home page](http://www.easyproject.cn/easymybatispagination/en/index.jsp 'The official home page')

[Comments](http://www.easyproject.cn/easymybatispagination/en/index.jsp#donation 'Comments')

If you have more comments, suggestions or ideas, please contact me.



Email：<inthinkcolor@gmail.com>

[http://www.easyproject.cn](http://www.easyproject.cn "EasyProject Home")



**支付宝钱包扫一扫捐助：**

我们相信，每个人的点滴贡献，都将是推动产生更多、更好免费开源产品的一大步。

**感谢慷慨捐助，以支持服务器运行和鼓励更多社区成员。**

<img alt="支付宝钱包扫一扫捐助" src="http://www.easyproject.cn/images/s.png"  title="支付宝钱包扫一扫捐助"  height="256" width="256"></img>



We believe that the contribution of each bit by bit, will be driven to produce more and better free and open source products a big step.

**Thank you donation to support the server running and encourage more community members.**

[![PayPal](http://www.easyproject.cn/images/paypaldonation5.jpg)](https://www.paypal.me/easyproject/10 "Make payments with PayPal - it's fast, free and secure!")

