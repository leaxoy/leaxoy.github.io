# 深入Go标准库之 Database

&emsp;&emsp;很多应用都离不开数据库的访问与操作，在JAVA中，我们通常使用Mybatis来进行数据库操作，在Python中，通常会使用SQLAlchemy。

&emsp;&emsp;在Go里面，标准库已经提供了数据库相关的操作，具体的有两个包`database/sql`和`database/sql/driver`。`database/sql`提供了连接池，对于底层的`connection`操作被隐藏到了该包里面，主要有连接池的维护，底层connection的保活，最大空闲连接数等等都已经实现，对用户来说是透明的。`database/sql/driver`提供了程序到数据库的映射关系，比如一个类型的转换，数据列信息。通常`connection`用来实现`driver`的接口。

## database/sql
### Register
用来注册一个数据库驱动，每个名字的数据库驱动只可注册一次，多次注册会导致panic。
### ColumnType
数据列信息，包含了列名和类型。以及其他的一些用来scan的属性。
```go
type ColumnType struct {
    name string

    hasNullable       bool
    hasLength         bool
    hasPrecisionScale bool

    nullable     bool
    length       int64
    databaseType string
    precision    int64
    scale        int64
    scanType     reflect.Type
}
```
#### ColumnType.DatabaseTypeName
数据库中的类型名称，比如`VARCHAR`, `BIGINT`, `Timestamp`
### Conn
`driver.Conn`的包装，用来实现一些方法.底层会调用`driver.Conn`的方法。
### DB
数据库连接池，维护了所有的`Conn`.
### DBStats
数据库连接池的统计数据.
### IsolationLevel
数据库事务的隔离级别
- `LevelDefault`
- `LevelReadUncommitted`
- `LevelReadCommitted`
- `LevelWriteCommitted`
- `LevelRepeatableRead`
- `LevelSnapshot`
- `LevelSerializable`
- `LevelLinearizable`
### NamedArg
命名Field，作为执行数据库操作使得args, 需要底层的驱动进行支持.

`db.Exec("insert into tests (id, name) values(:id, :name)", sql.Named("id", 1), sql.Named("name", "李大辉"))`

通过`sql.Named(name string, value interface{}) NamedArg`生成.
> 需要注意的是，最常用的数据库MySQL的驱动暂不支持这种写法。
### Null 类型
&emsp;&emsp;包含了大量的`Null`类型，表明一个字段是否为空，`NullBool`, `NullFloat64`, `NullInt32`, `NullInt64`, `NullString`, `NullTime`.

&emsp;&emsp;如果一个数据库的字段可能为空的话，建议使用这里的类型。可以正确处理零值与`null`的关系。
这些类型都实现了`sql.Scanner`和`driver.Valuer`接口，分别用来数据库到应用程序的转换以及反向的转换。
### Result
&emsp;&emsp;数据库`Exec(Context)`系列操作的返回值，是一个`interface`， 有两个方法：
- `LastInsertId() (int64, error)` 返回插入操作的插入的ID
- `RowsAffected() (int64, error)` 返回更新操作的影响的行数
### Row(s)
&emsp;&emsp;数据执行`QueryRow(Context)`和`Query(Context)`的结果。
`Rows`可以执行`Next`方法，用来判断结果集中是否有更多的记录, `NextResultSet`判断是否有下一个结果集.
### Scanner
接口类型，定义了从数据库数据解析到Go数据的操作。
### Stmt
预处理好的SQL语句，协程安全。
> 如果是基于Conn或者Tx的话，当Conn或者Tx关闭后，再次使用会报错。
> 如果是基于DB的话，则不会有此问题的存在。
### Tx

## database/sql/driver
## 自定义类型
&emsp;&emsp;得益于标准库的设计，我们可以很方便的拓展自己的类型，比如说字符串列表，在数据库中，我们想要存储成`string`，但是在程序中，我们希望得到的是`[]string`，如果没有自定义类型的支持，我们将不得不在程序中进行额外的转换，在次数少的时候，尚可cover住，在大量的这种case下，难免会有疏漏。

&emsp;&emsp;为了实现自定义类型，我们需要两个接口，`sql.Scanner`和`driver.Valuer`/`driver.ValueConverter`，前者支持数据库数据到程序数据的转换，后者支持程序数据到数据库数据的转换。

Show me the code:
```go
type StringArray []string

func (c *StringArray) Scan(src interface{}) error {
    switch x := src.(type) {
    case string:
        *c = strings.Split(x, ",")
        return nil
    case []byte:
        *c = strings.Split(string(x), ",")
        return nil
    default:
        return nil
    }
}

func (c StringArray) ConvertValue(v interface{}) (driver.Value, error) {
    switch x := v.(type) {
    case *StringArray:
        return x.Value()
    case StringArray:
        return x.Value()
    case []string:
        return StringArray(x).Value()
    case string:
        return x, nil
    case []byte:
        return x, nil
    default:
        return nil, nil
    }
}

func (c StringArray) Value() (driver.Value, error) {
    x := strings.Join(c, ",")
    return x, nil
}
```
