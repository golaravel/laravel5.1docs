# Redis

- [简介](#introduction)
- [基本用法](#basic-usage)
    - [管道命令](#pipelining-commands)
- [发布 / 订阅](#pubsub)

<a name="introduction"></a>
## 简介

[Redis](http://redis.io) 是一个开源的，先进的键-值存储库。因其可存储 [strings](http://redis.io/topics/data-types#strings), [hashes](http://redis.io/topics/data-types#hashes), [lists](http://redis.io/topics/data-types#lists), [sets](http://redis.io/topics/data-types#sets), 和 [sorted sets](http://redis.io/topics/data-types#sorted-sets)，又常常称为数据结构服务器。在Laravel中使用Redis之前，你需要通过Composer安装 `predis/predis` 包 (~1.0)。

<a name="configuration"></a>
### 配置

你的应用的 Redis 配置位于 `config/database.php` 配置文件中。在此文件中，你会看到 `redis` 数组包含了你所使用的 Redis 服务器：

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

默认的服务器配置应该足以供开发使用。然而，你可以依照自己的环境自由地修改这个数组。简单地为每个 Redis 服务器指定名称，主机地址以及所使用的端口。

`cluster` 选项会告知 Laravel Redis 客户端在你的 Redis 节点上执行客户端的分片，允许你集中节点以及创建大量的可用内存。但请注意客户端分片不能处理故障转移；因此它主要适用于缓存另一个主存储器中的数据。

另外，你可以在 Redis 连接定义内定义一个 `options` 数组，这允许你指定一个 Predis 集合 [客户端选项](https://github.com/nrk/predis/wiki/Client-Options)。

如果你的 Redis 服务器要求身份验证，你可把 `password` 配置项添加到你的 Redis 服务器配置数组中来提供密码。

> **注意:** 如果你通过 PECL 安装了 Redis 的 PHP 扩展，你需要在 `config/app.php` 文件中为 Redis 重起一个别名。

<a name="basic-usage"></a>
## 基本用法

你可以通过 `Redis` [facade](/docs/{{version}}/facades) 调用许多方法来和 Redis 交互。`Redis`  facade 支持动态方法，这意味着你可以通过 facade 调用任何 [Redis命令](http://redis.io/commands) 并且这些命令将直接传递给 Redis。在这个例子中，我们将在 `Redis` facade 上调用 `get` 方法来使 Redis 调用 `GET` 命令：

    <?php

    namespace App\Http\Controllers;

    use Redis;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

当然，正如上面所提及的，你可在 `Redis` facade 上使用任意 Redis 命令。Laravel 使用魔术方法来向Redis服务器传递命令，因此也可以简单地为 Redis 命令传递参数：

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

或者，也可用 `command` 方法向服务器传递命令，该方法接收命令名称作为第一个参数，数组作为第二个参数：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 使用多Redis连接

你可通过调用 `Redis::connection` 方法来获得一个 Redis 实例：

    $redis = Redis::connection();

这将为你返回默认的 Redis 实例。如果你没使用服务器集群，可以向 `connection` 方法传递服务器名来获得你配置好的特定服务器：

    $redis = Redis::connection('other');

<a name="pipelining-commands"></a>
### 管道命令

当你需要在一个操作中发送许多命令时，应该使用管道。`pipeline` 方法接受一个参数：一个接收 Redis 实例的 `Closure` (闭包)。你可向这个Redis实例发布所有的命令，它们会在单一操作中全被执行：

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## 发布 / 订阅

Laravel 也为 Redis 的 `publish` 和 `subscribe` 命令提供了便捷的接口。这些 Redis 命令允许你在给定的 "频道" 中监听消息。你可从另一个应用将消息发布到该频道中，甚至是使用另外的编程语言，来简化应用和进程之间的交流。

首先，让我们通过 Redis 的 `subscribe` 方法来为一个频道安装监听器。我们将用 [Artisan命令](/docs/{{version}}/artisan) 来完成调用，因为调用 `subscribe` 方法会开启一个长时间运行的进程：

    <?php

    namespace App\Console\Commands;

    use Redis;
    use Illuminate\Console\Command;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function($message) {
                echo $message;
            });
        }
    }

现在，我们可以用 `publish` 向频道发布消息了：

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### 通配符订阅

使用 `psubscribe` 方法可以订阅一个通配符频道，来方便捕获所有频道的所有消息。`$channel` 名作为第二个参数传入所提供的回调 `Closure` 中：

    Redis::psubscribe(['*'], function($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function($message, $channel) {
        echo $message;
    });
