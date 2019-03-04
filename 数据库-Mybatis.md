## Mybatis与Hibernate的区别

1.  hibernate是全自动，而mybatis是半自动。hibernate拥有完整的JavaBean对象与数据库的映射结构来自动生成sql。而mybatis仅有基本的字段映射，对象数据以及对象实际关系仍然需要通过手写sql来实现和管理。
2. hibernate无法SQL优化。



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

