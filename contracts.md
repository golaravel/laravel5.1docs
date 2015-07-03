# Contract

- [简介](#introduction)
- [为什么要用 Contract？](#why-contracts)
- [Contract 参考](#contract-reference)
- [如何使用 Contract](#how-to-use-contracts)

<a name="introduction"></a>
## 简介

Laravel 中的 Contract 是一组定义了框架核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了实现队列任务所需实现的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了发送邮件所需要实现的方法。

框架为每个 contract 都提供了一个相对应的实现。例如，Laravel 为队列提供了各种驱动的实现，以及基于 [SwiftMailer](http://swiftmailer.org/) 对邮件功能的实现。

Laravel 所自带的所有 contract 都[有自己的 GitHub 仓库](https://github.com/illuminate/contracts)。除了方便列出所有可用的 contract 外，也可以作为独立、解耦的工具包供其他开发者使用。

### Contract Vs. Facade

Laravel 中的 [facades](/docs/{{version}}/facades) 提供了一个简单的方法来使用 Laravel 自带的服务（service），而不需要使用使用类型提示（type-hint）和在服务容器（service container）之外解析 contract。然而，使用 contract 可以明确地为类定义其依赖。对于大部分应用程序而放眼，使用 facade 就足够了。然而，如果你需要更多的低耦合，contract 就适合你了，你可以继续往下看了解更多！

<a name="why-contracts"></a>
## 为什么使用 Contract ？

你可能对于 contract 还有很多疑问。为什么要使用接口（interface）？使用接口会不会更复杂了？接下来列出的两个标题能够解释这个问题：低耦合和简单性。

### 低耦合

首先，我们来看一下这段和缓存功能的实现具有强耦合的代码。如下：

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

在这个类中，代码和缓存功能的实现之间是强耦合的。因为它依赖第三方工具包中定义的缓存类。如果这个第三方工具包的 API 改变了，我们的代码也要跟着变。

同样的，如果我们希望用另一项技术（Redis）来替换掉现在所用的（Memcached），我们必须修改代码。我们所定义的类不应该依赖是谁提供了数据以及如何提供的等等细节。

**比起上面代码中的做法，我们的代码应该依赖一个简单、不依赖第三方实现细节的接口：**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

现在，上面的代码已经不再和具体的第三方代码甚至是 Laravel 有任何耦合了。由于 contract 工具包不包含任何实现，也不依赖其他工具包，你可以很方便地为任何 contract 创造一个实现，甚至在不需要修改任何涉及到调用缓存的代码的情况下替换为你自己的缓存实现。

### 简单性

当所有的 Laravel 服务（service）都通过简洁的接口进行定义，就能够很容易地确定这个服务提供的是什么功能了。**可以将 contract 视为框架所提供的简洁明了的说明文档。**

另外，当你依赖一个简单清晰的接口时，你的代码能够被别人轻松的理解并维护。比起在一个冗长、复杂的类中追踪哪些方法可用来说，你所面对的将是一个简洁、干净的接口。

<a name="contract-reference"></a>
## Contract 索引

下面列出的是大部分 Laravel Contract 的索引文件，以及对应的 "facade" ：

Contract  |  对应的 Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## 如何使用 Contract

那么，如何实例化一个 contract 呢？其实很简单。

Laravel 中的很多类都是由 [服务容器](/docs/{{version}}/container) 来解析的，包括控制器、事件监听器、中间件、队列任务，甚至路由闭包（route closure）。因此，要实例化一个 contract，你可以在，类的构造函数中加入“类型提示（type-hint）”。

例如，请看下面的事件监听器：

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\NewUserRegistered;
    use Illuminate\Contracts\Redis\Database;

    class CacheUserInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  NewUserRegistered  $event
         * @return void
         */
        public function handle(NewUserRegistered $event)
        {
            //
        }
    }

当事件监听器被解析的时候，服务容器将会读取构造函数的类型提示（type-hint），并注入适当的值。关于如何向服务容器注册，请参考[此文档](/docs/{{version}}/container)。
