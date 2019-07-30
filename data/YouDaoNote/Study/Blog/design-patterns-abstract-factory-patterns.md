---
title: 设计模式(四) 抽象工厂模式
date: 2017-05-25 09:03:48
tags:
 - 设计模式
 - 抽象工厂
categories: [设计模式]
---

# 定义


# UML类图


# 简单实现

## 最基本的数据访问程序
### User表
```
public class User {
    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

### SqlServerUser类
```
public class SqlServerUser {
    private static final String TAG = "SqlServerUser";
    public void insertUser(User user){
        Log.d(TAG, "insert: 在SQL Server中录入一个用户记录");
    }

    public User getUser(int id){
        Log.d(TAG, "getUser: 在SQL Server中获取一个用户记录");
        return null;
    }
}
```

### 客户端调用
```java
	User user = new User();
	user.setId(1);
	user.setName("Allever");
	SqlServerUser sqlServerUser = new SqlServerUser();
	sqlServerUser.insertUser(user);
	sqlServerUser.getUser(1);
```
如果换为MySQL数据库改动就很多

## 使用工厂方法改进

### IUser接口
```
public interface IUser {
    void insertUser(User user);
    User getUser(int id);
}
```

### SqlServerUser类
```
public class SqlServerUser implements IUser {

    private static final String TAG = "SqlServerUser";
    @Override
    public void insertUser(User user){
        Log.d(TAG, "insert: 在SQL Server中录入一个用户记录");
    }

    @Override
    public User getUser(int id){
        Log.d(TAG, "getUser: 在SQL Server中获取一个用户记录");
        return null;
    }
}
```

### MySQLUser类
```
public class MySQLUser implements IUser {
    private static final String TAG = "MySQLUser";
    @Override
    public void insertUser(User user){
        Log.d(TAG, "insert: 在MySQL Server中录入一个用户记录");
    }

    @Override
    public User getUser(int id){
        Log.d(TAG, "getUser: 在MySQL Server中获取一个用户记录");
        return null;
    }
}
```

### IFactory抽象工厂接口
```
public interface IFactory {
    IUser createUser();
}
```

### SqlServerFactory具体工厂类
```
public class SqlServerFactory implements IFactory {
    @Override
    public IUser createUser() {
        return new SqlServerUser();
    }
}
```



### MySQLServerFactory具体工厂
```
public class MySQLServerFactory implements IFactory {
    @Override
    public IUser createUser() {
        return new MySQLUser();
    }
}
```

### 客户端调用
```
        User user = new User();
        user.setId(1);
        user.setName("Allever");

        IFactory factory = new SqlServerFactory();
        IUser iUser = factory.createUser();
        iUser.insertUser(user);
        iUser.getUser(1);
```

如果改为MySQL数据库，这改动这里
```
IFactory factory = new MySQLServerFactory();
```

## 使用抽象工厂

### 新增Department表
```
public class Department {
    private int id;
    private String deptName;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }
}
```

### 新增IDepartment接口
```
public interface IDepartment {
    void insertDepartment(Department department);
    Department getDepartment(int id);
}
```

### 新增SqlServerDepartment类
```
public class SqlServerDepartment implements IDepartment {
    private static final String TAG = "SqlServerDepartment";
    @Override
    public void insertDepartment(Department department) {
        Log.d(TAG, "insertDepartment: 在Sql Server 中插入Department");
    }

    @Override
    public Department getDepartment(int id) {
        Log.d(TAG, "insertDepartment: 在Sql Server 中获取Department");
        return null;
    }
}
```

### 新增MySQLDepartment类
```
public class MySQLDepartment implements IDepartment {
    private static final String TAG = "MySQLDepartment";
    @Override
    public void insertDepartment(Department department) {
        Log.d(TAG, "insertDepartment: 在MySQL 中插入Department");
    }

    @Override
    public Department getDepartment(int id) {
        Log.d(TAG, "insertDepartment: 在MySQL 中获取Department");
        return null;
    }
}
```

### 修改IFactory接口
```
public interface IFactory {
    IUser createUser();
    IDepartment createDepartment();	//新增
}
```

### 修改SqlServerFactory类
```
public class SqlServerFactory implements IFactory {
    @Override
    public IUser createUser() {
        return new SqlServerUser();
    }

    @Override
    public IDepartment createDepartment() {
        return new SqlServerDepartment();
    }
}
```

### 修改MySQLServerFactory类
```
public class MySQLServerFactory implements IFactory {
    @Override
    public IUser createUser() {
        return new MySQLUser();
    }

    @Override
    public IDepartment createDepartment() {
        return new MySQLDepartment();
    }
}
```

### 客户端调用
```
        User user = new User();
        user.setId(1);
        user.setName("Allever");
        Department department = new Department();
        department.setId(1);
        department.setDeptName("Computer");

        IFactory factory = new MySQLServerFactory();
        IUser iUser = factory.createUser();
        iUser.insertUser(user);
        iUser.getUser(1);
        
        IDepartment iDepartment = factory.createDepartment();
        iDepartment.insertDepartment(department);
        iDepartment.getDepartment(1);
```
# 总结
只有一个User类和User操作类的时候，只需要工厂方法模式， 但现在数据库中有很多表， 而SQL Server 和 MySQL 又是两大不同的分类， 所以解决这种涉及到多个产品系列的时候，就要用到抽象工厂模式。
