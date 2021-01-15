---
title: Mysql连接封装脚本
date: 2021-01-15 15:58:39
tags: [编程, 过去]
category:
- 200 工作类
- 230 工作技能
- 231 懒人必备脚本
---

## 使用方法
### Mysql连接工具类
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.*;
import java.util.*;

public class MysqlConnectManager {

	private static Logger log = LoggerFactory.getLogger(MysqlConnectManager.class);

	private static ResourceBundle rb = ResourceBundle.getBundle("systemParamters");
	private static String driver = rb.getString("mysql.driverClass");
	private static String url = rb.getString("mysql.jdbcUrl");
	private static String username = rb.getString("mysql.user");
	private static String password = rb.getString("mysql.password");
	
	private volatile static MysqlConnectManager singleton = null;
	private static Connection con = null;
	
	private MysqlConnectManager() {
		try {
			//注册 JDBC 驱动
			Class.forName(driver);
			con = getConn();			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 获取mysql连接
	 * @return
	 */
	public static MysqlConnectManager getInstance() {
		if (singleton == null) {
			synchronized (MysqlConnectManager.class) {
				if (singleton == null) {
					singleton = new MysqlConnectManager();
				}
			}
		}
		return singleton;
	}
	
	public Connection getConn(){
		if(con == null){
			try {
				//连接数据库..
				con = DriverManager.getConnection(url, username, password);
			}catch (Exception e) {
				e.printStackTrace();
			}
		}
		return con;
	}
	
	
	public void closeConn(){
		if(con != null){
			try {
				con.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
	
	public void executeMysql(String sql, Boolean closeConn){
		getConn();
		try {
			PreparedStatement stmt = con.prepareStatement(sql);
			stmt.execute();
		} catch (Exception e) {
			e.printStackTrace();
		}
		if(closeConn)
			closeConn();
	}

	/**
	 * 插入日志
	 */
	public void insertLog(String dataStr,int status,String reason,String url,Boolean closeConn){
		String sql = "INSERT INTO l_biz_tiza_data_log(create_time,data_str,status,reason,url) VALUES(NOW(),'"+dataStr+"',"+status+",'"+reason+"','"+url+"')";
		executeMysql(sql,closeConn);
	}

	/**
	 * 查询
	 * @param columns 列
	 * @param tableName 表名
	 * @param whereSql where条件
	 * @param closeConn 是否关闭连接
	 * @return
	 */
	public List<Map<String, Object>> queryMysql(String columns, String tableName, String whereSql, Boolean closeConn){
		List<Map<String, Object>> res = new ArrayList<Map<String, Object>>();
		getConn();
		try {
			String sql = "select "+columns+" from "+tableName;
			if(!isEmpty(whereSql)){
				sql += " where "+whereSql;
			}
			log.info(sql+"sql执行了");
			PreparedStatement stmt = con.prepareStatement(sql);
			ResultSet rs = stmt.executeQuery();

			while (rs.next()) {
				Map<String, Object> ele = new HashMap<String, Object>();

				int count = 1;
				for(String s : columns.split(",")){
					if(s.contains(" as ")){
						ele.put(s.split(" as ")[1], rs.getObject(count++));
					}else{
						ele.put(s, rs.getObject(count++));
					}
				}
				res.add(ele);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		if(closeConn)
			closeConn();
		return res;
	}


	/**
	 * 判断字符串是否为空
	 */
	public static boolean isEmpty(String str) {

		boolean flag = false;
		if (str == null || "".equals(str) || "null".equals(str)) {

			flag = true;
		} else {
			flag = false;
		}
		return flag;
	}

}
```
### 定义systemParamters.properties文件
```
mysql.jdbcUrl=jdbc:mysql://172.16.70.180:3306/test_db?rewriteBatchedStatements=true&failOverReadOnly=false&maxReconnects=1&autoReconnect=true&useUnicode=true&characterEncoding=utf-8&useSSL=false
mysql.user=root
mysql.password=password
mysql.driverClass=com.mysql.jdbc.Driver
```
### pom文件引入依赖
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
### 使用
```
List<Map<String, Object>> list = MysqlConnectManager.getInstance().queryMysql(columns, "l_biz_data", "terminal_id=1", false);
```
























