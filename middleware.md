# HTTP 中间件

- [简介](#introduction)
- [定义中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
- [中间件参数](#middleware-parameters)
- [可终止中间件](#terminable-middleware)

<a name="introduction"></a>
## 简介

HTTP 中间件为过滤访问你的应用的 HTTP 请求提供了一个方便的机制。例如，Laravel 默认包含了一个验证用户的中间件。如果没有经过身份验证，中间件将会将用户重定向至登录页面。然而，如果用户经过了验证，中间件将会允许请求继续在应用中执行下去。

当然，除了身份验证，中间件也可以被用来执行多种多样的任务。一个 CORS 中间件可能负责在所有应用发出去的响应中加入适当的头部。一个日志中间件可能记录所有发送给应用的请求的日志。

Laravel 框架已经自带了一些中间件，包括维护、身份验证、 CSRF 保护等等。所有中间件都位于 `app/Http/Middleware` 文件夹中。

<a name="defining-middleware"></a>
## 定义中间件

使用 `make:middleware` Artisan 命令创建一个新的中间件：

    php artisan make:middleware OldMiddleware

这个命令会在 `app/Http/Middleware` 文件夹中放置一个新的 `OldMiddleware` 类。在这个中间件中，我们只允许 `age` 大于 200 的才能访问到路由。否则，我们会把用户重定向至 "home" 的 URI。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class OldMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('age') <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

正如你看到的，如果用户给出的 `age` 小于或等于 `200`，中间件会给客户端返回一个 HTTP 重定向；否则，请求会继续在应用程序中执行下去。只用调用带有 `$request` 的 `$next` 方法，就可以将请求继续在应用中传递到更深层的逻辑（允许跳过中间件）。 

在抵达应用程序之前，请求层层通过一系列中间件是最好的设计。每一层可以对其进行检查，甚至是完全拒绝请求。


### *Before* / *After* 中间件

一个中间件是否在一个请求之前或之后运行取决于这个中间件本身。举个例子，下面的中间件将会在请求前执行一些 **前置** 操作。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

然而，下面这个中间件会在请求后执行一些 **后置** 操作：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 注册中间件

### 全局中间件

如果你期望中间件在每一个 HTTP 请求时都会执行，只需要将中间件类加入到 `app/Http/Kernel.php` 类的 `$middleware` 属性的列表中。

### 指派中间件给路由

如果你期望将中间件指派给一个特定的路由，你需要首先在 `app/Http/Kernel.php` 文件中指定一个键值。默认情况下，这个类的 `$routeMiddleware` 属性包含了 Laravel 目前已经配置的中间件。只需要在这个属性的列表里加上一组自定义的键值即可。

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    ];

一旦中间件在 HTTP kernel 文件中被定义，你可以在路由选项数组中使用 `middleware` 键来指派：

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接收额外的自定义参数。例如，如果你的应用需要验证用户是否在执行 action 之前拥有给定的 “角色”，你可以创建一个接受角色名称作为额外参数的 `RoleMiddleware` 中间件。

中间件的额外参数会在 `$next` 参数后传入：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

中间件参数可以在定义路由时将中间件的名称和参数用 `:` 隔开来指定。多个参数应当用逗号隔开：

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## 可终止中间件

有时候中间件可能需要在一个请求已经发送给浏览器后做一些工作。例如， Laravel 包含的 "session" 中间件需要在响应发送给用户 _之后_ 保存 session 数据。你需要将中间件定义为 "可终止的" 来完成这件事情。 

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

`terminate` 方法会收到请求（request）和响应（response）。一旦你已经定义了一个可终止的中间件，你需要将它添加到 HTTP kernel 的全局中间件列表中去。

当调用你的中间件的 `terminate` 方法时，Laravel 将通过 [服务容器](/docs/{{version}}/container) 解析并创建一个全新的中间件实例。如果你希望在 `handle` 和 `terminate` 方法中引用同一个中间件实例的话，请使用服务容器的 `singleton` 方法来注册中间件。
