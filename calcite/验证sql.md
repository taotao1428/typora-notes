# 验证sql

两个重要概念

## scope

表示一个作用域。作用域与sqlNode是一一对应关系。在sqlNode中出现的变量都可以在作用域中寻找真正的引用。



例如：`select p.* from T.person as p where name = 'taotao'`整个sql语句对应一个Scope。其中出现的`p.*`,`name`都可以在Scope中寻找真正的引用。如果没有找到，说明sql写错了。





## namespace

表示为作用域提供变量引用的空间。namespace包含跟Schema，table。schema提供对table的引用，table提供对字段的引用。



例如：`select p.* from T.person as p where name = 'taotao'`。`T.person as p`就是一个namespace，`p.*`就可以在该namespace中寻找引用

Select语句的namespace：`SelectNamespace`





## 校验类

`CalciteSqlValidator`

```java
final SqlOperatorTable opTab =
    SqlOperatorTables.chain(operatorTable, catalogReader);
return new CalciteSqlValidator(opTab,
    catalogReader,
    getTypeFactory(),
    sqlValidatorConfig
        .withDefaultNullCollation(connectionConfig.defaultNullCollation())
        .withLenientOperatorLookup(connectionConfig.lenientOperatorLookup())
        .withSqlConformance(connectionConfig.conformance())
        .withIdentifierExpansion(true));
```



`SqlValidatorImpl`

```java
protected final IdentityHashMap<SqlNode, SqlValidatorNamespace> namespaces =
    new IdentityHashMap<>();
```

namespace用于保存sqlNode和namespace之间的关系

```java
protected void registerNamespace(
    @Nullable SqlValidatorScope usingScope, // 父级Scope
    @Nullable String alias,
    SqlValidatorNamespace ns,
    boolean forceNullable) {
  namespaces.put(requireNonNull(ns.getNode(), () -> "ns.getNode() for " + ns), ns);
  if (usingScope != null) { // 通常在from后面的表都代表namespace，需要别注册到select语句对应的Scope上
    assert alias != null : "Registering namespace " + ns + ", into scope " + usingScope
        + ", so alias must not be null";
    usingScope.addChild(ns, alias, forceNullable);
  }
}
```

scopes用于保存sqlNode和scope之间的关系

```java
protected final IdentityHashMap<SqlNode, SqlValidatorScope> scopes =
    new IdentityHashMap<>();
```

clauseScopes用于保存Select语句中，其他部分使用的scope

```java
private final Map<IdPair<SqlSelect, Clause>, SqlValidatorScope>
    clauseScopes = new HashMap<>();
```




```java
/**
 * Registers a query in a parent scope.
 *
 * @param parentScope Parent scope which this scope turns to in order to
 *                    resolve objects
 * @param usingScope  Scope whose child list this scope should add itself to
 * @param node        Query node
 * @param alias       Name of this query within its parent. Must be specified
 *                    if usingScope != null
 * @param checkUpdate if true, validate that the update feature is supported
 *                    if validating the update statement
 */
private void registerQuery(
    SqlValidatorScope parentScope,
    @Nullable SqlValidatorScope usingScope,
    SqlNode node,
    SqlNode enclosingNode,
    @Nullable String alias,
    boolean forceNullable,
    boolean checkUpdate) {
  requireNonNull(node, "node");
  requireNonNull(enclosingNode, "enclosingNode");
  Preconditions.checkArgument(usingScope == null || alias != null);

  SqlCall call;
  List<SqlNode> operands;
  switch (node.getKind()) {
  case SELECT:
    final SqlSelect select = (SqlSelect) node;
    final SelectNamespace selectNs =
        createSelectNamespace(select, enclosingNode);
    registerNamespace(usingScope, alias, selectNs, forceNullable);
    final SqlValidatorScope windowParentScope =
        (usingScope != null) ? usingScope : parentScope;
    SelectScope selectScope =
        new SelectScope(parentScope, windowParentScope, select);
    scopes.put(select, selectScope);

    // Start by registering the WHERE clause
    clauseScopes.put(IdPair.of(select, Clause.WHERE), selectScope);
    registerOperandSubQueries(
        selectScope,
        select,
        SqlSelect.WHERE_OPERAND);

    // Register FROM with the inherited scope 'parentScope', not
    // 'selectScope', otherwise tables in the FROM clause would be
    // able to see each other.
    final SqlNode from = select.getFrom();
    if (from != null) {
      final SqlNode newFrom =
          registerFrom(
              parentScope,
              selectScope,
              true,
              from,
              from,
              null,
              null,
              false,
              false);
      if (newFrom != from) {
        select.setFrom(newFrom);
      }
    }

    // If this is an aggregating query, the SELECT list and HAVING
    // clause use a different scope, where you can only reference
    // columns which are in the GROUP BY clause.
    SqlValidatorScope aggScope = selectScope;
    if (isAggregate(select)) {
      aggScope =
          new AggregatingSelectScope(selectScope, select, false);
      clauseScopes.put(IdPair.of(select, Clause.SELECT), aggScope);
    } else {
      clauseScopes.put(IdPair.of(select, Clause.SELECT), selectScope);
    }
    if (select.getGroup() != null) {
      GroupByScope groupByScope =
          new GroupByScope(selectScope, select.getGroup(), select);
      clauseScopes.put(IdPair.of(select, Clause.GROUP_BY), groupByScope);
      registerSubQueries(groupByScope, select.getGroup());
    }
    registerOperandSubQueries(
        aggScope,
        select,
        SqlSelect.HAVING_OPERAND);
    registerSubQueries(aggScope, SqlNonNullableAccessors.getSelectList(select));
    final SqlNodeList orderList = select.getOrderList();
    if (orderList != null) {
      // If the query is 'SELECT DISTINCT', restrict the columns
      // available to the ORDER BY clause.
      if (select.isDistinct()) {
        aggScope =
            new AggregatingSelectScope(selectScope, select, true);
      }
      OrderByScope orderScope =
          new OrderByScope(aggScope, orderList, select);
      clauseScopes.put(IdPair.of(select, Clause.ORDER), orderScope);
      registerSubQueries(orderScope, orderList);

      if (!isAggregate(select)) {
        // Since this is not an aggregating query,
        // there cannot be any aggregates in the ORDER BY clause.
        SqlNode agg = aggFinder.findAgg(orderList);
        if (agg != null) {
          throw newValidationError(agg, RESOURCE.aggregateIllegalInOrderBy());
        }
      }
    }
    break;
```





注册From语句，主要有下面两个动作

1. 从from中获取namespace中，并将其添加到usingScope中
2. 注册from中子查询语句

```java
private SqlNode registerFrom(
    SqlValidatorScope parentScope,
    SqlValidatorScope usingScope,
    boolean register,
    final SqlNode node,
    SqlNode enclosingNode,
    @Nullable String alias,
    @Nullable SqlNodeList extendList,
    boolean forceNullable,
    final boolean lateral) {
  final SqlKind kind = node.getKind();
```





## 验证

验证

`validateQuery`



验证namespace

```java
protected void validateNamespace(final SqlValidatorNamespace namespace,
    RelDataType targetRowType) {
  namespace.validate(targetRowType);
  SqlNode node = namespace.getNode();
  if (node != null) {
    setValidatedNodeType(node, namespace.getType());
  }
}
```