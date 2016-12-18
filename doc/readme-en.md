## EasyMyBatis Pagination

EasyMyBatis Pagination is a generic paging plugin for the MyBaits framework. Provides the PageBean automatic paging data encapsulation, the EasyCriteria paging condition object, and based `Mappers Interface` or `SQLID` the automated paging SQL that supports common databases.

Least version: `1.9.0-RELEASE`

MyBatis: `3.4.0+`


## Maven

```XML
<dependency>
	<groupId>cn.easyproject</groupId>
	<artifactId>easymybatis-pagination</artifactId>
	<version>1.9.0-RELEASE</version>
</dependency>
```


## Steps for usage

1. Configure **EasyMybatisPaginationPlugin** in  `mybatis-config.xml` 

 ```XML
 <plugins>
	<plugin interceptor="cn.easyproject.easymybatis.pagination.EasyMybatisPaginationPlugin">
		<!-- Default dialect; Optional -->
		<!-- Also you can set or change with 'pageBean.setDialect(PageBean.XXXX_DIALECT)' code. -->
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

## PageBean Use

- Example
 ```JAVA
 SqlSession session=MyBatisSessionFactory.getSession();

 PageBean pb=new PageBean();
 // SELECT Clause; optional; default is *
 pb.setSelect("*"); 
 // FROM Name; rquired
 pb.setFrom("Account account");
 // WHERE Clause; optional; default is ''
 pb.setCondition(" and account.qxid>=10");
 // Append where clause condition; optional; default is ''
 //pb.addCondition(""); 
 // SortName; optional; default is ''
 pb.setSort("account.accountid");
 // SortOrder; optional; default is 'asc'
 pb.setSortOrder("desc");
 // Page Number; optional; default is 1
 pb.setPageNo(1);
 // Rows per page; optional; default is 10
 pb.setRowsPerPage(4);

 // Query way one: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // Query way two: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);
 
 // Pagination data
 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount());
 ```

- Example2
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

 // Query way one: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // Query way two: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);

 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount());
 ```
 
- Example3

 A multi-table query must specify a fact table alias: `pageBean.setPrimaryTable();`

 ```JAVA
 PageBean pb=new PageBean();
 // SELECT Clause; optional; default is *
 pb.setSelect("*"); 
 // FROM Name; rquired
 pb.setFrom("Account account, AccountType type");
 // A multi-table query must specify a fact table alias
 pageBean.setPrimaryTable("account");
 // WHERE Clause; optional; default is ''
 pb.setCondition(" and account.typeId=type.id and account.qxid>=10");
 // SortName; optional; default is ''
 pb.setSort("account.accountid");
 // SortOrder; optional; default is 'asc'
 pb.setSortOrder("desc");
 // Page Number; optional; default is 1
 pb.setPageNo(1);
 // Rows per page; optional; default is 10
 pb.setRowsPerPage(4);

 // Query way one: Mappers Interface
 AccountDAO accountDAO=session.getMapper(AccountDAO.class);
 accountDAO.pagination(pb)
 
 // Query way two: SQL ID
 session.selectList("cn.easyproject.easymybatis.pagination.dao.AccountDAO.pagination", pb);
 
 // Pagination data
 System.out.println(pb.getData());
 System.out.println(pb.getPageTotal());
 System.out.println(pb.getPageNo());
 System.out.println(pb.getRowsPerPage());
 System.out.println(pb.getRowsCount()); 
 ```
 
## EasyCriteria Query

1. New your EasyCriteria class, must `extends EasyCriteria implements Serializable`

2. Write your condition by `getCondition()`


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

3. Use EasyCriteria in PageBean Query

- Example

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
### [The official home page](http://www.easyproject.cn/easymybatispagination/en/index.jsp 'The official home page')

[Comments](http://www.easyproject.cn/easymybatispagination/en/index.jsp#donation 'Comments')

If you have more comments, suggestions or ideas, please contact me.


### [官方主页](http://www.easyproject.cn/easymybatispagination/zh-cn/index.jsp '官方主页')

[留言评论](http://www.easyproject.cn/easymybatispagination/zh-cn/index.jsp#donation '留言评论')

如果您有更好意见，建议或想法，请联系我。




Email：<inthinkcolor@gmail.com>

[http://www.easyproject.cn](http://www.easyproject.cn "EasyProject Home")





We believe that the contribution of each bit by bit, will be driven to produce more and better free and open source products a big step.

**Thank you donation to support the server running and encourage more community members.**

[![PayPal](http://www.easyproject.cn/images/paypaldonation5.jpg)](https://www.paypal.me/easyproject/10 "Make payments with PayPal - it's fast, free and secure!")
