---
title: JavaWeb 项目搭建Hibernate环境
date: 2017-05-19 16:31:34
tags:
 - JavaWeb
 - Hibernate
categories: [JavaWeb]
---

# 环境
 - Eclipse
 - MySQL

# 下载Hibernate所需jar包
可到官网下载: [快速通道](http://hibernate.org/orm/)  

下载好后解压，打开压缩包下的lib目录下的require文件夹，这是hibernate的所以来的必须的jar包，copy到Project/WebContent/WEB-INF/lib 目录下，并添加依赖, 没有的自行创建  

由于我们需要连接MySQL数据库，将所需的mysql-connector-java-5.0.8-bin.jar引用进去，关于这些jar包，可以在网上搜索。  

> 注意这里我用的是4.2版本的hibernate-core-4.2.1.Final.jar

# 配置Hibernate.conf.xml

我们需要配置最重要的hibernate配置文件hibernate.cfg.xml以及进行日志处理的log4j.properties属性文件：打开上一步解压后的hibernate文件夹，打开project—>etc文件夹，将该文件夹下的hibernate.cfg.xml和log4j.properties文件拷贝到项目的src文件夹下，打开hibernate.cfg.xml文件，将session-factory标签中的内容替换成如下的内容：
```
	<session-factory>
		<!-- MySQL -->
		<property name="hibernate.connection.url">jdbc:mysql://localhost:3306?useUnicode=true&amp;characterEncoding=UTF-8</property> 
		<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property> 
		<property name="hibernate.connection.username">root</property> 
		<property name="hibernate.connection.password">dixm</property>
		<property name="hibernate.dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property> 
		<property name="hibernate.show_sql">true</property>
		<property name="hibernate.use_sql_comments">true</property> 
		<property name="hibernate.connection.pool_size">0</property>
		
		<!-- Mappings  -->
		
	</session-factory>
```

# 自动生成数据库表
## 创建DDLCreator类
```
import org.hibernate.cfg.Configuration;
import org.hibernate.tool.hbm2ddl.SchemaExport;

public class DDLCreator {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		Configuration cfg = new Configuration().configure();
		SchemaExport se = new SchemaExport(cfg);
		se.drop(true, true);    //ɾ
		se.create(true, true);  //
	}
}
```
## 创建HibernateUtil类
```
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;


@SuppressWarnings("deprecation")
public class HibernateUtil {
	private static SessionFactory sessionFactory;
	static {
		try {
			sessionFactory = new Configuration().configure().buildSessionFactory();
		} catch (Throwable ex) {
			throw new ExceptionInInitializerError(ex);
		}
	}

	public static SessionFactory getSessionFactory() {
		// Alternatively, you could look up in JNDI here
		return sessionFactory;
	}

	public static void shutdown() {
		// Close caches and connection pools
		getSessionFactory().close();
	}
	
	public static Session getSession()
	{
		Session session = sessionFactory.openSession();
		return session;
	}
	
	public static void beginSession(Session session)
	{
		session.getTransaction().begin();
	}
	
	public static void commitTransaction(Session session)
	{
		session.getTransaction().commit();
	}
	
	public static void rollbackTransaction(Session session)
	{
		Transaction tx = session.getTransaction();
		if (tx.isActive())
			tx.rollback();
	}
}

```

## 创建实体类
如 TVersion
```
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity							//表示实体类
@Table(catalog = "dbcoolweather") //数据库名
public class TVersion {
	
	private long id;
	private int version_code;
	private String version_name;
	private String description;
	private String path;
	
	//主键，自增
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public long getId() {
		return id;
	}
	public void setId(long id) {
		this.id = id;
	}
	public int getVersion_code() {
		return version_code;
	}
	public void setVersion_code(int version_code) {
		this.version_code = version_code;
	}
	public String getVersion_name() {
		return version_name;
	}
	public void setVersion_name(String version_name) {
		this.version_name = version_name;
	}
	public String getDescription() {
		return description;
	}
	public void setDescription(String description) {
		this.description = description;
	}
	public String getPath() {
		return path;
	}
	public void setPath(String path) {
		this.path = path;
	}
}
```
## 在Hibernate.cfg.xml中添加映射关系
```
<!-- Mappings  -->
<mapping class="com.allever.mycoolweather.bean.TVersion"/>
```

## 创建数据库
首先在MySQL中手动创建数据库： dbcoolweather

## 运行DDLCreator类
刷新数据库表，此时，tversion表已经建好了, 有木有很惊讶～～～

# 对数据库进行增删改查
## 创建VersionDAO类
```
import java.util.List;

import org.hibernate.Query;
import org.hibernate.Session;

import com.allever.mycoolweather.bean.TVersion;
import com.allever.mycoolweather.util.HibernateUtil;


public class VersionDAO {
	
	Session session = null;
	boolean commit = false;

	public VersionDAO() {
		this.session = HibernateUtil.getSession();
		commit = true;
		HibernateUtil.beginSession(session);
	}
	
	
	public VersionDAO(Session session) {
		this.session = session;
		commit = false;
	}
	
	public void close() {
		if (commit == true) {
			HibernateUtil.commitTransaction(session);
			session.close();
		}
	}

	public Session getSession() {
		return session;
	}
	
	
	public TVersion getById(long id) throws Exception {
		TVersion d = null;
		try {
			d = (TVersion) session.get(TVersion.class, id);
		} catch (Exception e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}
		return d;
	}
	
	
	@SuppressWarnings("unchecked")
	public List<TVersion> getByQuery(String conditions, long start,
			long limit) throws Exception {
		List<TVersion> dl = null;

		try {
			String hql = "from TVersion";
			if ((conditions != null) && (conditions.length() > 0))
				hql += " where " + conditions;

			Query query = session.createQuery(hql);
			if (limit > 0) {
				query.setFirstResult((int) start);
				query.setMaxResults((int) limit);
			}
			dl = (List<TVersion>) query.list();
		} catch (Exception e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}

		return dl;
	}
	
	@SuppressWarnings("unchecked")
	public List<TVersion> getUpdateVersion(long start,long limit) throws Exception {
		List<TVersion> dl = null;

		try {
			String hql = "from TVersion ORDER BY version_code DESC";

			Query query = session.createQuery(hql);
			if (limit > 0) {
				query.setFirstResult((int) 0);
				query.setMaxResults((int) 0);
			}
			dl = (List<TVersion>) query.list();
		} catch (Exception e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}

		return dl;
	}
	
	public TVersion add(TVersion d) throws Exception {
		Long id = null;

		try {
			id = (Long) session.save(d);
		} catch (Exception e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}

		return getById(id);
	}
	
	public void deleteById(long id) throws Exception {
		TVersion d = null;

		try {
			d = (TVersion) session.get(TVersion.class, id);
			if (d != null)
				session.delete(d);
		} catch (Exception e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}
	}
	
	public TVersion update(TVersion d) throws Exception {
		try {
			session.update(d);
		}
		catch(RuntimeException e) {
			HibernateUtil.rollbackTransaction(session);
			throw e;
		}
		
		return getById(d.getId());
	}
}
```
各位看函数名就知道要干什么了吧，这里就不哆嗦了

## 创建VersionTest类进行测试
```
public class VersionTest {
	public static void main(String[] args){
		
	}
}
```
就是一个普通的JAVA类

### 初始化DAO
```
VersionDAO dao = new VersionDAO();
//operation
dao.close();
```

### 增加一条记录
```
try{
	TVersion v1 = new TVersion();
	v1.setVersion_code(5);
	v1.setDescription("第3版");
	v1.setVersion_name("1.3");
	v1.setPath("/apk/mycoolweather_1.3.apk");
	dao.add(v1);
}catch(Exception e){
	e.printStackTrace();
}finally{
	dao.close();
}
```

### 查询一条记录
```
List<TVersion> list_tversion = new ArrayList<>();
list_tversion = dao.getByQuery("version_code="+ 5, 0, 0);
if(list_tversion.size()>0){
	TVersion v = list_tversion.get(0);
	v.getVersion_name();
}
```
### 查询所有记录
```
List<TVersion> list_tversion = new ArrayList<>();
// null or ""
list_tversion = dao.getByQuery(null, 0, 0);
for(TVersion tversion: list_tversion){
	tversion.getVersion_name();
	System.out.println(tversion.getDescription());
}
```

### 修改数据
```
List<TVersion> list_tversion = new ArrayList<>();
list_tversion = dao.getByQuery("version_code="+ 5, 0, 0);
if(list_tversion.size()>0){
	TVersion v = list_tversion.get(0);
	v.setDescription("new Description");
	dao.update(v);
}
```

### 删除数据
```
List<TVersion> list_tversion = new ArrayList<>();
list_tversion = dao.getByQuery("version_code="+ 5, 0, 0);
if(list_tversion.size()>0){
	TVersion v = list_tversion.get(0);
	v.setDescription("new Description");
	dao.deleteById(v.getId());
}
```
OK,大功告成.
写这篇复习了以前所学知识，下一步，继续学习新版本的知识


