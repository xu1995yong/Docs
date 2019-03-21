## Mybatis与Hibernate的区别

1.  hibernate是全自动，而mybatis是半自动。hibernate拥有完整的JavaBean对象与数据库的映射结构来自动生成sql。而mybatis仅有基本的字段映射，对象数据以及对象实际关系仍然需要通过手写sql来实现和管理。
2.  hibernate无法SQL优化。

## Mybatis的基本构成

1. SqlSessionFactoryBuilder：根据配置信息或代码生成SqlSessionFactory。
2. SqlSessionFactory：生成Sqlsession的工厂。SqlSessionFactory的生命周期与应用的生命周期相同。
3. SqlSession：负责执行SQL并返回结果，是线程不安全的。
4. SQL Mapper：Mapper是由Java和xml文件（或注解）共同组成，用于定义参数类型、描述缓存、描述SQL语句、定义查询结果和POJO对应关系。

## Mybatis中的缓存

### 一级缓存

一级缓存是SqlSession级别的缓存，作用于SqlSession对象。不同的sqlSession间的一级缓存（HashMap）互不影响。

一级缓存的范围有SESSION和STATEMENT两种，默认是SESSION，但在SESSION级别一级缓存存在脏读的问题。

### 二级缓存

二级缓存是mapper级别的缓存，当多个SqlSession操作同一个Mapper的sql语句时，可以共享该Mapper的二级缓存。二级缓存需要手动开启。

### 缓存的回收策略

可用的回收策略有4种, 默认的是 LRU:

1.      LRU – 最近最少使用的:移除最长时间不被使用的对象。

2.      FIFO – 先进先出:按对象进入缓存的顺序来移除它们。

3.      SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。

4.      WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。





## Mybatis对枚举类型的映射

### Mybatis提供的枚举类型TypeHandler

Mybatis自身对枚举类型提供了两种TypeHandler：

- EnumTypeHandler：EnumTypeHandler是mybatis默认的枚举类型TypeHandler，该TypeHandler存储枚举的名称。
- EnumOrdinalTypeHandler：该TypeHandler存储枚举的索引。此时数据库表字段一般对应用smallint/int类型。

### 自定义枚举类型TypeHandler

由于有时默认TypeHandler不能满足需求，所以需要自定义枚举类型。

#### 实现自定义的TypeHandler

```java
public class UserStatusTypeHandler implements TypeHandler<UserStatus> {
    @Override
    public void setParameter(PreparedStatement preparedStatement, int parameterIndex, UserStatus userStatus, JdbcType jdbcType)throws SQLException {
        preparedStatement.setInt(parameterIndex, userStatus.getCode());
    }

    @Override
    public UserStatus getResult(ResultSet resultSet, String columnName) throws SQLException {
        int code = resultSet.getInt(columnName);
        UserStatus userStatus = UserStatus.getUserStatus(code);
        return userStatus;
    }

    @Override
    public UserStatus getResult(ResultSet resultSet, int columnIndex) throws SQLException {
        int code = resultSet.getInt(columnIndex);
        UserStatus userStatus = UserStatus.getUserStatus(code);
        return userStatus;
    }

    @Override
    public UserStatus getResult(CallableStatement callableStatement, int columnIndex) throws SQLException {
        int code = callableStatement.getInt(columnIndex);
        UserStatus userStatus = UserStatus.getUserStatus(code);
        return userStatus;
    }
}
```

#### 注册自定义的TypeHandler

```xml
<typeHandlers>
    <typeHandler handler="com.xu.pojo.UserStatusTypeHandler" javaType="com.xu.pojo.UserStatus"/>
</typeHandlers>
```

