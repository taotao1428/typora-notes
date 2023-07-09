# validate过程分析

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-dCBeb2HL-1638799411330)(img/validate过程分析.asset/image-20211206201006393.png)]



## validate简介

将sql解析成SqlNode之后，已经验证了基本的语法问题。validate阶段就是结合库表信息和配置验证sqlNode是否正确。如果validate验证无问题，才可以将其转化为relNode，进行下一步操作。

### 验证点

1. 标识符。库、表、方法、运算符、操作符
2. 字段。字段是否存在，字段类型是否正确，字段个数是否正确



### 代码

使用Planner验证sqlNode

```java
private RelNode buildRelNode(String sql, Planner planner) throws Exception {
    SqlNode sqlNode = planner.parse(sql);
    planner.validate(sqlNode); // 验证阶段
    return planner.rel(sqlNode).project();
}
```



手动构造sqlNode和validator

```java
// 1. 构建schema
SchemaPlus schemaPlus = Frameworks.createRootSchema(true);
schemaPlus.add("T", new ReflectiveSchema(new TestOne.TestSchema()));
this.schemaPlus = schemaPlus;

// 2. 构建sqlNode
String sql = "select a.id, a.name, b.address from T.person as a left join T.address as b on a.id = b.ownerId where a.id = 'aa' and b.id = 'cc'";
SqlParser.Config parserConfig = SqlParser.config().withCaseSensitive(false);
SqlParser parser = SqlParser.create(new SourceStringReader(sql), parserConfig);
SqlNode sqlNode = parser.parseStmt();

log.info("sqlNode: \n{}\n", sqlNode);

// 3. 构建validator
SqlStdOperatorTable operatorTable = SqlStdOperatorTable.instance(); // 包含标准的操作符
JavaTypeFactoryImpl typeFactory = new JavaTypeFactoryImpl(RelDataTypeSystem.DEFAULT);
CalciteConnectionConfigImpl connectionConfig = CalciteConnectionConfig.DEFAULT
        .set(CalciteConnectionProperty.CASE_SENSITIVE, String.valueOf(parserConfig.caseSensitive()));

CalciteCatalogReader calciteCatalogReader = new CalciteCatalogReader( // 代表库表信息
        CalciteSchema.from(schemaPlus),
        CalciteSchema.from(schemaPlus).path(null),
        typeFactory, connectionConfig);

final SqlOperatorTable opTab = SqlOperatorTables.chain(operatorTable, calciteCatalogReader);
SqlValidator.Config validatorConfig = SqlValidator.Config.DEFAULT // 验证器配置
        .withIdentifierExpansion(true); // 将*展开

SqlValidatorImpl sqlValidator = new MysqlSqlValidator(opTab, calciteCatalogReader, typeFactory, validatorConfig);

// 4. 执行验证
SqlNode validatedSqlNode = sqlValidator.validate(sqlNode); // 验证sqlNode

log.info("validatedSqlNode: \n{}\n", validatedSqlNode);
```



## 如何实现

因为sql包含子查询、join和方法等操作，它们之间组合可以形成比较复杂的sql。为介绍validate的基本流程，先从一张仅涉及单表的查询select分析，例如有下面的sql：

```sql
SELECT `A`.`id` AS `ID`, LOWER(`A`.`name`), `B`.`province` AS `PROVINCE`
FROM `T`.`person` AS `A`
LEFT JOIN `T`.`address` AS `B` ON `A`.`id` = `B`.`ownerId`
WHERE `A`.`id` = 'aa' AND `B`.`id` = 'cc'
```

分析：

1. 表是否存在。sql涉及从T.person和T.address表中获取数据。验证点1：T.person和T.address表需要存在
2. 字段是否存在。on字句中引用的字段必须来自于它左边出现的表中的字段。（当join中出现子查询，需要将子查询也看做一张表）；select子句和where子句可以从from中出现的表中获取字段
3. 方法是否存在，方法参数是否正确。sql中涉及Lower方法



可以看出，如果要校验表的和字段，需要先确定一个表的范围。在validate实现中，就是使用Scope和NameSpace两个概念表示范围。Scope和NameSpace定义如下：

1. Scope表示一个作用域，Scope有继承关系，Scope中可以注册NameSpace，与Scope绑定的字句可以在该Scope和父级Scope中所有NameSpace中查询表或者字段。select，join等关键字都可以表示一个Scope。
2. NameSpace表示命名空间。在from中出现的表或者子查询或产生namespace，产生的NameSpace需要注册到起所属的Scope中。
3. 在初始的过程中，会给每个子句绑定其所在的Scope，方便后期校验。



### 相关对象



### 验证流程





## 总结