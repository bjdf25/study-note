**一级缓存**：作用于Session，默认开启，一个session的一条sql第一次查询时会把数据从数据库查出来之后放到缓存中，当同一条sql再次执行时会优先从缓存中查询。修改的操作会清除缓存。
**二级缓存**：作用于Mapper(Namespace)，默认关闭。二级缓存容易引起脏读幻读的情况，是因为Amapper对X表查询产生了二级缓存，这个时候Bmapper如果对X表做了修改操作，X表的数据已经发生了改变，但是由于Amapper没有对X表做修改操作，所以Amapper再次查询的话依然会走缓存，这样Amapper查询的数据就不是最新的数据，产生了脏读幻读情况。

当mybatis与spring结合时，spring中每一次拿到mapper对象就会开启一个新的session，除非是在一个事务中拿到mapper对象，这样也会使用同一个session。
对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。
一个session会查询多个mapper。