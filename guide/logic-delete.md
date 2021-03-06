# 逻辑删除

SpringBoot 配置方式：

- application.yml 加入配置(如果你的默认值和mp默认的一样,该配置可无):

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  #全局逻辑删除字段值 3.3.0开始支持，详情看下面。
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

- 实体类字段上加上`@TableLogic`注解

```java
@TableLogic
private Integer deleted;
```

::: tip 说明:
- 字段支持所有数据类型(推荐使用 `Integer`,`Boolean`,`LocalDateTime`)
- 如果使用`LocalDateTime`,建议逻辑未删除值设置为字符串`null`,逻辑删除值只支持数据库函数例如`now()`
:::
  
- 效果: 使用mp自带方法删除和查找都会附带逻辑删除功能 (自己写的xml不会)

``` sql
example
删除 update user set deleted=1 where id =1 and deleted=0
查找 select * from user where deleted=0
```
  
- 全局逻辑删除: since 3.3.0

  如果公司代码比较规范，比如统一了全局都是flag为逻辑删除字段。
  
  使用此配置则不需要在实体类上添加 @TableLogic。
  
  但如果实体类上有 @TableLogic 则以实体上的为准，忽略全局。  即先查找注解再查找全局，都没有则此表没有逻辑删除。

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  #全局逻辑删除字段值
```
  
  
::: tip 附件说明
- 逻辑删除是为了方便数据恢复和保护数据本身价值等等的一种方案，但实际就是删除。
- 如果你需要再查出来就不应使用逻辑删除，而是以一个状态去表示。

如： 员工离职，账号被锁定等都应该是一个状态字段，此种场景不应使用逻辑删除。

- 若确需查找删除数据，如老板需要查看历史所有数据的统计汇总信息，请单独手写sql。
:::

::: tip 效果说明:
自动注入的`method`里,除了 insert 外的 select update 操作均会屏蔽 逻辑删除字段 的相关自动逻辑
既在sql内自动追加条件只能对未删除数据进行 select 和 update 
:::

## 常见问题:

> Q1: 如何 insert ?
> A1:
>> 1. 字段在数据库定义默认值(推荐)
>> 2. insert 前自己 set 值  
>> 3. 使用自动填充功能  

