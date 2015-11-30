# 数据库使用入门

- [介绍](#introduction)
- [执行原生 SQL 语句](#running-queries)
    - [监听查找事件](#listening-for-query-events)
- [数据库事务](#database-transactions)
- [使用多个数据库连接](#accessing-connections)

<a name="introduction"></a>
## 介绍

Laravel 让连接数据库和执行SQL语句变得相当简单。它支持原生SQL语句，[fluent query 生成器](/docs/{{version}}/queries), 和 [Eloquent ORM](/docs/{{version}}/eloquent)。 当前 Laravel 支持以下4种数据库系统：

- MySQL
- Postgres
- SQLite
- SQL Server

<a name="configuration"></a>
### 配置

Laravel 让链接数据库和执行SQL语句变的很简单。数据库的相关配置存放在 `config/database.php` 中。你可以定义所有的数据库连接，以及指定默认的连接。在该文件中你能找到所有支持的数据库的连接示例。

默认情况下 Laravel 的范例数据库 [environment configuration](/docs/{{version}}/installation#environment-configuration) 在 [Laravel Homestead](/docs/{{version}}/homestead) 下可以直接使用。[Laravel Homestead](/docs/{{version}}/homestead) 是一个便捷的用于在本地机器上进行 Laravel开发的虚拟环境。当然，你可以根据需要修改本地的数据库配置。

<a name="read-write-connections"></a>
#### 读取 / 写入 连接

有时你希望将一个连接用于 SELECT 操作，另一个连接用于 INSERT, UPDATE, DELETE 操作。 Laravel让这些变得很简单，并且确保不管你是用原生query, query 生成器 或是 Eloquent ORM时都是使用正确的连接。

让我们通过下面这个例子来看看如何配置读取/写入连接：

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

注意我们在配置数组中添加了两个键值对： `read` 和 `write`。它们都有一个值为以`host`为键的数组。关于`read` 和 `write` 的其余的数据库配置项则合并在 `mysql` 数组中

所以我们只需要将想要覆写的选项加入到 `read` 和 `write` 数组中。在这个例子中，`192.168.1.1` 将被用于“读取”连接，同时 `192.168.1.2`被用于“写入”连接。数据库的其它配置比如密码，前缀，字符编码等则共用 `mysql` 数组中的配置。

<a name="running-queries"></a>
## 运行原生SQL语句

当你把数据库连接配置好，就可以使用 `DB` facade 执行语句了。`DB` facade 为不同类型 `select`, `update`, `insert`, `delete`, and `statement` 的语句提供了多种方法。

#### 执行 Select 查找

你可以通过 `DB` facade 中 `select` 方法执行一个简单的查询：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

原生的SQL语句被当做第一个参数传给 `select` 方法，同时第二个参数为语句中绑定的参数。通常它们都是 `where` 从句中进行限制的值。参数绑定防止了SQL注入攻击。

`select`方法的返回值一直是数组。数组中的各个结果为一个PHP `StdClass` 对象，你可以通过它来访问结果的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用名字绑定

相比于使用 `?` 来进行参数绑定，你可以使用名字绑定来执行一个语句：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 执行 Insert 插入语句

你可以使用 `DB` facade中的`insert` 方法来执行 `insert` 插入语句。和 `select` 类似，该方法以原生SQL语句为第一个参数，绑定数组为第二个参数：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 执行 Update 语句

`update`方法应用于更新数据库中已存在的纪录。该方法返回被更新的纪录的个数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 执行 Delete 语句

`delete`方法应用于从数据库中删除记录，和`update`一样，该方法将被删除的纪录的个数作为返回值：

    $deleted = DB::delete('delete from users');

#### 执行一般语句

有些数据库语句没有任何返回值。对于这样的操作，你可以使用 `DB` facade 中的 `statement` 方法：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### 监听查找事件

如果你想获得所有执行了的SQL语句，你可以使用 `listen` 方法。该方法对于纪录日志和调试很有用。你可以在 [service provider](/docs/{{version}}/providers) 中注册你的query监听器：

    <?php

    namespace App\Providers;

    use DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function($sql, $bindings, $time) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>
## 数据库事务

你可以使用 `DB` facade 中的 `transaction` 方法来执行一个数据库事务中的一组操作。如果事务 `闭包` 抛出任何异常，事务将会自动回滚。若 `闭包` 执行成功，该事务将自动被提交。当使用 `transaction` 方法时，你无需手动回滚或是手动提交：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 手动执行事务

如果你希望手动开始一个事务并且对于回滚和提交拥有完全的控制，你可以使用 `DB` facade中的 `beginTransaction` 方法：

    DB::beginTransaction();

你可以通过 `rollBack` 方法进行回滚：

    DB::rollBack();

最后，你可以通过 `commit` 方法提交事务：

    DB::commit();

> **注意:** 使用 `DB` facade 的 transaction 方法同样可以控制 [query 生成器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent)。

<a name="accessing-connections"></a>
## 使用多个数据库连接

当有多个数据库连接时，你可以通过 `DB` facade的 `connection` 方法取得各个连接。传给 `connection` 方法的 `name` 参数需要和配置文件 `config/database.php` 中的某个连接对应：

    $users = DB::connection('foo')->select(...);

你也可以通过连接实例的 `getPdo` 方法来获取基础的 PDO 实例：

    $pdo = DB::connection()->getPdo();
