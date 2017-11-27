---
layout:     post
title:      "SSM（一）-MyBatis Enum 映射"
subtitle:   "extends typeHandler"
date:       2017-11-20 12:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - SSM
---

> 伤情最是晚凉天，憔悴斯人不堪怜。邀酒摧肠三杯醉，寻香惊梦五更寒。
钗头凤斜卿有泪，荼蘼花了我无缘。小楼寂寞心与月，也难如钩也难圆。

---

## 前言  
这篇博文是偶来得之，今天在做SSM框架集成ElasticSearch的demo，先弄了个空的SSM框架，然后写了个简单的list查询，其中有个字段sex（性别），想用 **枚举** 来实现以下，因为之前用枚举类型比较少，正好借助这个熟练下，却想不到也遇到了不少的坑，花了两个多小时的时间，才实现了这个简单的功能。  

## 概览

一开始我是直接简单暴力，直接照着另一个demo的代码，写了一个`SexEnum`枚举类，然后直接ResultMap都没写直接上来就强行映射，可想而知，是行不通的，于是开始google了几篇文章，才知道要自定义转换器，于是就有了以下操作；这些操作的目的：**可以做到直接使用枚举插入数据库中会自动变成数字或者其他东西，而查询时可以直接将数据库中的数字或其他东西转换成对应的枚举值**。

代码托管在github：[SsmESDemo](https://github.com/zhangxx0/SsmESDemo)  

同时，这个demo也是SSM集成ES的demo，不出意外的话，接下来会有介绍它的博文。

## 定义枚举类

```java
/**
 * 性别枚举类
 */
public enum  SexEnum {
    MALE(0,"男"),
    FEMALE(1,"女");

    private int sex;
    private String sexName;

    SexEnum(int sex, String sexName) {
        this.sex = sex;
        this.sexName = sexName;
    }
    public int getSex() {
        return sex;
    }
    public String getSexName() {
        return sexName;
    }

}
```

## 枚举映射Handler  

继承`BaseTypeHandler`，并重写其四个方法：  

```java
/**
 * 性别枚举handler
 *
 * @date 2017年11月20日15:26:03
 */
public class SexEnumTypeHandler extends BaseTypeHandler<SexEnum> {

    private Class<SexEnum> sexEnum;

    public SexEnumTypeHandler(Class<SexEnum> sexEnum) {
        if (sexEnum == null) {
            throw new IllegalArgumentException("Type argument can not be null");
        }
        this.sexEnum = sexEnum;
    }

    public void setNonNullParameter(PreparedStatement preparedStatement, int i, SexEnum sexEnum, JdbcType jdbcType) throws SQLException {
        preparedStatement.setInt(i,sexEnum.getSex());
    }

    public SexEnum getNullableResult(ResultSet resultSet, String columnName) throws SQLException {
        int i = resultSet.getInt(columnName);
        if (resultSet.wasNull()) {
            return null;
        } else {
            return getValuedEnum(i);
        }
    }

    public SexEnum getNullableResult(ResultSet resultSet, int columnIndex) throws SQLException {
        int i = resultSet.getInt(columnIndex);
        if (resultSet.wasNull()) {
            return null;
        } else {
            return getValuedEnum(i);
        }
    }

    public SexEnum getNullableResult(CallableStatement callableStatement, int columnIndex) throws SQLException {
        int i = callableStatement.getInt(columnIndex);
        if (callableStatement.wasNull()) {
            return null;
        } else {
            return getValuedEnum(i);
        }
    }

    private SexEnum getValuedEnum(int value) {
        SexEnum[] objs = sexEnum.getEnumConstants();
        for(SexEnum em:objs){
            if(em.getSex()==value){
                return  em;
            }
        }
        throw new IllegalArgumentException(
                "Cannot convert " + value + " to " + sexEnum.getSimpleName() + " by value.");
    }
}
```

## 实体类使用枚举

```java
public class UserDemo {
    private int age;
    private Long id;
    private String userName;
    private SexEnum sex;
    private Date birthday;
    // ...
}
```

## MyBatis的xml文件修改  

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.danke.dao.UserDemoDao">
    <!--目的:为dao接口方法提供sql语句配置
    即针对dao接口中的方法编写我们的sql语句-->

    <resultMap id="userDemoMap" type="UserDemo">
        <id column="id" javaType="LONG" property="id"/>
        <result column="user_name" jdbcType="VARCHAR" property="userName"/>
        <result column="age" jdbcType="INTEGER" property="age"/>
        <result column="sex" property="sex" typeHandler="com.danke.util.typehandler.SexEnumTypeHandler"/>
        <result column="birthday" jdbcType="DATE" property="birthday"/>
        <result column="create_date" jdbcType="TIME" property="createDate"/>
    </resultMap>

    <select id="queryAll" resultMap="userDemoMap">
        SELECT id,user_name,age,sex,birthday,create_date
        FROM t_user
        ORDER BY create_date DESC
    </select>

</mapper>
```

## 前台页面对枚举的使用

```js
  <td>${item.sex.sexName}</td>
```


## 通用Enum映射的实现
```java
  TODO
```


## 总结与参考  


参考：  
[MyBatis 使用通用 Enum 映射](http://albertchen.top/2016/02/18/MyBatis-%E4%BD%BF%E7%94%A8%E9%80%9A%E7%94%A8-Enum-%E6%98%A0%E5%B0%84/)  
[mybatis枚举自动转换实现](http://blog.csdn.net/fighterandknight/article/details/51520402)  
[mybatis 官方文档](http://www.mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)     


2017年11月-by zhang.xx
