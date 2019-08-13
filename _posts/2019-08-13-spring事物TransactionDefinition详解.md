## TransactionDefinition详解

### TransactionDefinition定义事务隔离级别

TransactionDefinition.ISOLATION_DEFAULT	 

+ PlatformTransactionManager的默认隔离级别（对大多数数据库来说就是ISOLATION_ READ_COMMITTED）MySQL默认采用ISOLATION_REPEATABLE_READ，Oracle采用READ__COMMITTED级别。

TransactionDefinition.ISOLATION_READ_UNCOMMITTED

+ 最低的隔离级别。事实上我们不应该称其为隔离级别，因为在事务完成前，其他事务可以看到该事务所修改的数据。而在其他事务提交前，该事务也可以看到其他事务所做的修改。可能导致脏，幻，不可重复读

TransactionDefinition.ISOLATION_READ_COMMITTED

+ 大多数数据库的默认级别。在事务完成前，其他事务无法看到该事务所修改的数据。遗憾的是，在该事务提交后，你就可以查看其他事务插入或更新的数据。这意味着在事务的不同点上，如果其他事务修改了数据，你就会看到不同的数据。可防止脏读，但幻读和不可重复读仍可以发生。

TransactionDefinition.ISOLATION_REPEATABLE_READ

+ 比ISOLATION_READ_COMMITTED更严格，该隔离级别确保如果在事务中查询了某个数据集，你至少还能再次查询到相同的数据集，即使其他事务修改了所查询的数据。然而如果其他事务插入了新数据，你就可以查询到该新插入的数据。可防止脏读，不可重复读，但幻读仍可能发生。

TransactionDefinition.ISOLATION_SERIALIZABLE

+ 代价最大、可靠性最高的隔离级别，所有的事务都是按顺序一个接一个地执行。避免所有不安全读取。

### TransactionDefinition定义事务传播行为

TransactionDefinition.PROPAGATION_REQUIRED

+ 当前如果有事务，Spring就会使用该事务；否则会开始一个新事务

TransactionDefinition.PROPAGATION_SUPPORTS

+ 当前如果有事务，Spring就会使用该事务；否则不会开始一个新事务

TransactionDefinition.PROPAGATION_MANDATORY

+ 当前如果有事务，Spring就会使用该事务；否则会抛出异常

TransactionDefinition.PROPAGATION_REQUIRES_NEW

+ Spring总是开始一个新事务。如果当前有事务，则该事务挂起

TransactionDefinition.PROPAGATION_NOT_SUPPORTED

+ Spring不会执行事务中的代码。代码总是在非事务环境下执行，如果当前有事务，则该事务挂起

TransactionDefinition.PROPAGATION_NEVER

+ 即使当前有事务，Spring也会在非事务环境下执行。如果当前有事务，则抛出异常

TransactionDefinition.PROPAGATION_NESTED

+ 如果当前有事务，则在嵌套事务中执行。如果没有，那么执行情况与Transaction- Definition.PROPAGATION_REQUIRED一样