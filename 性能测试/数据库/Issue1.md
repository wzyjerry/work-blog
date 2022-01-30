我在Postgres上使用ent，表结构定义如下：

``` yaml
id: string
info[]:
	url: string
	count: int
```

其中info是一个对象数组，包含两个字段url和count。现在希望查询所有url值为x.com的数据，对应SQL查询为：

``` sql
SELECT * FROM tb WHERE info @> '[{"url":"x.com"}]'
```

我拼接了如下ent查询：

``` go
items, err := client.Debug().TB.Query().Where(func(s *entsql.Selector) {
  s.Where(sqljson.ValueContains(TB.FieldInfo, []interface{}{map[string]interface{}{"url", "x.com"}}))
}).All(ctx)
```

没有得到想要的结果。经查询在使用Postgres后端时，sqljson.ValueContains函数会调用normalizePG，该函数在arg为自定义类型时会将arg序列化为字符串。接下来，在实际进行Arg赋值时又会将arg序列化一次，导致实际发送给数据库的查询参数中自定义类型的arg被序列化了两次。

在我的例子中，将normalizePG函数中default的marshal删除可以得到期望的结果，现在我想知道，这里的两次序列化是预期的行为吗？如果是，我应该使用何种方式实现示例中的需求？



Hey,

I use ent with Postgres backend, and the table structure is defined as follows:

``` yaml
id: string
info[]:
  url: string
  count: int
```

Where info is an array of objects, containing two fields url and count. Now I want to query all data which url equals "x.com", the corresponding SQL is:

``` sql
SELECT * FROM tb WHERE info @> '[{"url":"x.com"}]'
```

So, I wrote the following ent query:

``` go
items, err := client.Debug().TB.Query().Where(func(s *entsql.Selector) {
  s.Where(sqljson.ValueContains(TB.FieldInfo, []interface{}{map[string]interface{}{"url", "x.com"}}))
}).All(ctx)
```

But I didn't get the desired result. I found that when using the Postgres backend, the "sqljson.ValueContains" function will call "normalizePG", which will serialize "arg" into a string when "arg" is a custom type. Next, "arg" will be serialized another time when the actual "Arg" assignment is performed, resulting in the arg of custom type in the query parameters actually sent to the database being serialized twice.
``` go
// ValueContains return a predicate for checking that a JSON
// value (returned by the path) contains the given argument.
//
//	sqljson.ValueContains("a", 1, sqljson.Path("b"))
//
func ValueContains(column string, arg interface{}, opts ...Option) *sql.Predicate {
	return sql.P(func(b *sql.Builder) {
		...
		switch b.Dialect() {
    ...
		case dialect.Postgres:
			opts, arg = normalizePG(b, arg, opts)
			path.Cast = "jsonb"
			path.value(b)
			b.WriteString(" @> ").Arg(marshal(arg)) // <=== marshal twice here
		}
	})
}

// normalizePG adds cast option to the JSON path is the argument type is
// not string, in order to avoid "missing type casts" error in Postgres.
func normalizePG(b *sql.Builder, arg interface{}, opts []Option) ([]Option, interface{}) {
	if b.Dialect() != dialect.Postgres {
		return opts, arg
	}
	base := []Option{Unquote(true)}
	switch arg.(type) {
	case string:
	case bool:
		base = append(base, Cast("bool"))
	case float32, float64:
		base = append(base, Cast("float"))
	case int8, int16, int32, int64, int, uint8, uint16, uint32, uint64:
		base = append(base, Cast("int"))
	default: // convert unknown types to text.
		arg = marshal(arg) // <=== This line
	}
	return append(base, opts...), arg
}
```
In my example, deleting the default "marshal" in the "normalizePG" function can get the desired result. Now I want to know, is this the expected behavior? If so, how should I implement the requirements in the example?
Thanks for any suggestions!

