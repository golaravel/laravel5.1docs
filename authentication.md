# Authentication
认证

- [简介](#introduction)
- [认证快速入门](#authentication-quickstart)
    - [路由](#included-routing)
    - [视图](#included-views)
    - [认证](#included-authenticating)
    - [获取登录用户信息](#retrieving-the-authenticated-user)
    - [保护路由](#protecting-routes)
    - [认证流量控制](#authentication-throttling)
- [手动认证用户](#authenticating-users)
    - [记住登录用户](#remembering-users)
    - [其他认证方式](#other-authentication-methods)
- [HTTP基本认证](#http-basic-authentication)
     - [无状态的HTTP基本认证](#stateless-http-basic-authentication)
- [重置密码](#resetting-passwords)
    - [数据库注意事项](#resetting-database)
    - [路由](#resetting-routing)
    - [视图](#resetting-views)
    - [重置密码之后](#after-resetting-passwords)
- [第三方认证](#social-authentication)
- [添加自定义认证驱动](#adding-custom-authentication-drivers)

<a name="introduction"></a>
## 简介

Laravel 让实现认证机制变得非常简单。事实上，几乎所有的配置已经默认完成了。认证的配置文件放在 `config/auth.php` 文件中 ，配置文件包含了一些为了修改认证服务行为并且有着良好注释的选项。

### 数据库注意事项

Laravel 在你的 `app` 文件夹中默认包含了一个 `App\User` [Eloquent 模型](/docs/{{version}}/eloquent)。这个模型可以被用作默认的 Eloquent 认证驱动。如果你的应用没有使用 Eloquent ，你可以使用 Laravel 的查询构造器作为 `database` 认证驱动。

当创建数据库框架时，请保证密码字段是至少60个字符长度。

同样，你应当确认你的 `users` （或者等价的） 表包含一个可以为空的，100个字符长度的 `remember_token` 字符串字段。这个字段将会被用来储存 “记住我” 的 session token 。这可以在迁移里面使用 `$table->rememberToken();` 来完成。

<a name="authentication-quickstart"></a>
## 认证快速入门

Laravel 预设了两个和认证相关的控制器，这两个控制器位于 `App\Http\Controllers\Auth` 命名空间。`AuthController` 处理新用户的注册和认证，而 `PasswordController` 包含了帮助已注册的用户重置他们忘记了的密码的逻辑。

<a name="included-routing"></a>
### 路由

默认情况下，没有[路由](docs/{{version}}/routing)被设定去将请求指向到认证的控制器中。你需要手动地将它们添加到 `app/Http/routes.php` 文件中：

    // Authentication routes...
    Route::get('auth/login', 'Auth\AuthController@getLogin');
    Route::post('auth/login', 'Auth\AuthController@postLogin');
    Route::get('auth/logout', 'Auth\AuthController@getLogout');

    // Registration routes...
    Route::get('auth/register', 'Auth\AuthController@getRegister');
    Route::post('auth/register', 'Auth\AuthController@postRegister');

<a name="included-views"></a>
### 视图

尽管框架中已经包含了认证的控制器，你需要提供[视图](/docs/{{version}}/views)来让这些控制器可以渲染，视图应当放置在 `resources/views/auth` 文件夹中。你可以自由地按自己的想法定义这些视图。登录视图应当被放置为 `resources/views/auth/login.blade.php` ，并且注册的视图应当被放置为 `resources/views/auth/register.blade.php`。

#### 认证登录表单示例

    <!-- resources/views/auth/login.blade.php -->

    <form method="POST" action="/auth/login">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password" id="password">
        </div>

        <div>
            <input type="checkbox" name="remember"> Remember Me
        </div>

        <div>
            <button type="submit">Login</button>
        </div>
    </form>

#### 登录表单示例

    <!-- resources/views/auth/register.blade.php -->

    <form method="POST" action="/auth/register">
        {!! csrf_field() !!}

        <div>
            Name
            <input type="text" name="name" value="{{ old('name') }}">
        </div>

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div>
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">Register</button>
        </div>
    </form>

<a name="included-authenticating"></a>
### 认证

既然你已经为认证设置好路由和视图了，你可以开始注册并且认证一个应用的新用户了。你可以简单地访问你在浏览器中定义的路由。认证控制器已经（通过他们的 traits ）包含了认证注册用户和将新用户储存到数据库中的逻辑。

当一个用户成功认证后，他们将被重定向至 `/home` URI ，你需要将这个URI注册到路由中去处理这件事情。你可以通过定义一个 `AuthController` 类下的 `redirectPath` 属性自定义认证完成后的重定向的位置：

    protected $redirectPath = '/dashboard';

当一个用户没有成功地认证，他们将会被重定向到 `/auth/login` URI。你可以通过定义一个 `AuthController` 类下的 `loginPath` 属性自定义认证认证失败后的重定向位置：

    protected $loginPath = '/login';

#### 自定义

为了修改一个你的应用程序在注册时后必填的一个表单项，或者自定义新用户记录如何插入到你的数据库中，你可以修改 `AuthController` 类。这个类负责验证和创建新用户。

`AuthController` 下的 `validator` 方法包含了新用户的验证规则。你可以自由地去改变这个方法。

`AuthController` 的 `create` 方法负责使用 [Eloquent ORM](/docs/{{version}}/eloquent) 在数据库中创建一个新的 `App\User` 记录。你可以自由地根据数据库去修改这个方法。

<a name="retrieving-the-authenticated-user"></a>
### 获取登录信息

你可以通过 `Auth` facade 来访问已经认证了的用户：

    $user = Auth::user();

额外地，如果一个用户被认证后，你可以通过一个 `Illuminate\Http\Request` 实例去访问已认证的用户。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() returns an instance of the authenticated user...
            }
        }
    }

#### 查看一个用户是否登录

你可以使用 `Auth` facade 的 `check` 方法来插卡一个用户是否已经登录到你的系统中了，如果用户已经登录，则该方法会返回 `true`：

    if (Auth::check()) {
        // The user is logged in...
    }

然而，你可以使用中间件来验证登录用户是否有权限去访问一个具体的路由或控制器。如果想要了解更多有关这方面的内容，可以查看 [保护路由](/docs/{{version}}/authentication#protecting-routes) 中的文档。

<a name="protecting-routes"></a>
### 保护路由

[路由中间件](/docs/{{version}}/middleware) 可以用来允许登录的用户访问特定的路由。Laravel 装载了 `auth` 中间件，这个中间件在 `app\Http\Middleware\Authenticate.php` 中。你仅仅需要将中间件和路由定义连接起来：

    // Using A Route Closure...

    Route::get('profile', ['middleware' => 'auth', function() {
        // Only authenticated users may enter...
    }]);

    // Using A Controller...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

当然，如果你在使用 [控制器类](/docs/{{version}}/controllers)，你可以从调用 `middleware` 方法：

    public function __construct()
    {
        $this->middleware('auth');
    }

<a name="authentication-throttling"></a>
### 认证流量控制

如果你正在使用 Laravel 内置的 `Authcontroller` 类， `Illuminate\Foundation\Authcontroller\Throttleslogins` trait 可以被用来控制应用程序的登录尝试的流量。默认地，用户如果有几次输入错误的登录口令，他将不能在一分钟内登录。流量控制和用户的 IP 地址以及用户名邮箱相互独立：

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // Rest of AuthController class...
    }

<a name="authenticating-users"></a>
## 手动登录用户

当然，你并非必须去使用 Laravel 自带的认证控制器。如果你要删除这些控制器，你需要直接使用 Laravel 认证类去管理用户的认证。不要担心，这只是小菜一碟！

我们使用 `Auth` [facade](/docs/{{version}}/facades) 访问 Laravel 的认证服务，所以我们需要确保在类的顶部引入 `Auth` facade。然后，我们来看一下 `attempt` 方法：


    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt` 方法的第一个参数接受一个由键值对组成的数组。这些数组中的值会被用来在数据库表中查找用户。所以，如上面的例子所示， 会根据 `email` 列的值取出用户。如果在表中找到了用户，会通过传递给这个函数的数组中的 `password` 的值的哈希来与储存在数据库中的密码哈希进行比较。如果两个密码哈希值相等，会重新为用户启动认证通过的 session。

如果认证成功， `attempt` 方法将会返回 `true`。否则返回 `false`。

`intended` 方法会重定向用户要访问的 URL ，这个值会在进行认证过滤前被储存起来。也可以给这个方法传入一个预设的 URI ，防止重定向的网址无法使用。

如果你想的话，你也可以向认证查询中添加除了用户的邮箱和密码的额外的条件。例如，我们会检查一个用户是不是有 active 标记。

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

手动登出一个用户，你可以使用 `Auth` facade 下的 `logout` 方法。这个方法将会清空用户 session 中的认证信息：

    Auth::logout();

> **注意:** 在上面的示例中，并不一定要使用 email 字段，这只是作为示例。你应该使用对应到数据表中的「username」的任何键值。

<a name="remembering-users"></a>
## 记住用户

如果你想要在应用中提供“记住我”的功能，你可以传入一个布尔值作为 `attempt` 方法的第二个参数，这样就可以保留用户的认证身份，或者知道他们手动登出。当然，你的 `users` 表必须包含一个字符串类型的 `remember_token` 字段用来储存"记住我“ token。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

如果你在"记住用户"，你可以使用 `viaRemember` 方法去判定用户是否拥有"记住我"的cookie来进行用户认证：

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其他认证方式

#### 认证一个用户实例
如果你需要将一个存在的用户实例登录到你的应用中，你可以对用户实例调用 `login` 方法。传入的对象必须是一个 `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) 的实现。当然，Laravel 自带的  `App\User` 模型已经实现了这个接口。

    Auth::login($user);

#### 通过ID认证用户

你可以使用 `loginUsingId` 方法来通过 ID 认证用户。这个方法仅仅接受你要认证用户的主键作为参数： 

    Auth::loginUsingId(1);

#### 在单一请求内认证用户

你可以使用 `once` 方法将一个用户在单一请求内登录。不会有任何的 session 或 cookie 产生，这样会对你创建一个无状态的 API 非常有帮助。`once` 方法和 `attempt` 有着一样的参数：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP基本认证

[HTTP基本认证](http://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一个快速的方式来认证用户而不用设定一个特定的"登录"页面。在你的路由中设定 `auth.basic` [中间件](/docs/{{version}}/middleware) 启用这个功能. `auth.basic` 中间件被包含在 Laravel 框架中了，所以你不用去定义： 

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Only authenticated users may enter...
    }]);

一旦中间件已经在路由中设定了，当你在浏览器中访问时你可以自动地得到一个验证请求。默认得，`auth.basic` 中间件会使用 `email` 字段作为”用户名“。

#### FastCGI 的注意事项

如果你在使用 PHP 的 FastCGI，HTTP基本认证可能不会正常工作。所以你需要将下面几行添加到你的 `.htaccess` 文件中：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 无状态的HTTP基本认证

你也可以使用不将用户信息存储在 session 中的 HTTP 基本认证，这对API认证非常有用。你可以通过[定义一个中间件](/docs/{{version}}/middleware)并且调用 `onceBasic` 方法来这样做。如果 `onceBasic` 方法没有将响应返回，请求会继续在应用程序中传递下去：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

然后，[注册一个路由中间件](/docs/{{version}}/middleware#registering-middleware)并将它和路由联系起来：

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Only authenticated users may enter...
    }]);

<a name="resetting-passwords"></a>
## 重置密码

<a name="resetting-database"></a>
### 数据库注意事项

大多数 Web 应用提供一个忘记密码来让用户重置他们的密码的功能。你不用在每个应用中都去重新实现这个功能， Laravel 提供了发送密码提醒并且完成重置密码的方法。

要开始的话，确认你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` contract。当然了，框架中包含的 `App\User` 模型已经实现了这个接口，使用 `Illuminate\Auth\Passwords\CanResetPassword` trait 就可以将需要实现的接口方法包含进来。

#### 生成重置密码 Token 的表迁移

然后，必须创建一个表来存储密码重置的 token。这个表的迁移已经包含在 Laravel 中了，并且就在 `database/migrations` 文件夹中。所以，你仅仅需要迁移就可以了：

    php artisan migrate

<a name="resetting-routing"></a>
### 路由

Laravel 包含了一个实现了重置用户密码方法的 `Auth\PasswordController`。然而，你需要定义指向这个控制器的路由：

    // Password reset link request routes...
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // Password reset routes...
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### 视图

除了定义 `PasswordController` 的路由，你还需要提供控制器返回渲染的视图。不要担心，我们会提供一个示例视图来帮助你开始。当然了，你可以根据自己的喜好自定义你的表单样式。

#### 密码重置链接表单示例

你需要提供一个密码重置表单的 HTML 视图。这个视图需要替换 `resources/views/auth/password.blade.php` 文件。这个表单提供了一个用户邮箱地址的输入框，让他们可以请求一个密码重置链接：

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <button type="submit">
                Send Password Reset Link
            </button>
        </div>
    </form>

当用户提交重置密码的请求的时候，他们将会收到一个包含指向 `PasswordController` 的 `getReset` 方法的链接的邮件(一般是 `/password/reset` 路由)。你需要在 `resources/views/emails/password.blade.php` 创建一个这封邮件的视图。这个视图接受一个包含可以重置密码的 `$token` 变量。这有一个可以让你着手开始的一个邮件视图示例：

    <!-- resources/views/emails/password.blade.php -->

    Click here to reset your password: {{ url('password/reset/'.$token) }}

#### 密码重置表单示例

当用户点击了邮件中的重置密码链接，他们将会看到一个密码重置表单。这个视图应该位于 `resources/views/auth/reset.blade.php`。

这有一个可以让你开始的重置表单示例：

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div>
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">
                Reset Password
            </button>
        </div>
    </form>

<a name="after-resetting-passwords"></a>
### 重置密码之后

一旦你定义了重置用户密码的路由和视图，你就可以在你的浏览器中访问你的路由。框架中包含的 `PasswordController` 已经具有发送重置密码连接邮件和更新数据库中的密码的逻辑。

当密码已经被修改了，用户会自动的登录到系统中并且重定向到 `/home`。你可以通过定义 `PasswordController` 中的 `redirectTo` 属性自定义重置密码后的重定向连接。

    protected $redirectTo = '/dashboard';

> **注意:** 默认情况下，密码重置 token 在一小时后过期。你可以通过修改 `config/auth.php` 文件中的 `reminder.expire` 选项来修改过期时间。

<a name="social-authentication"></a>
## 第三方认证

除了典型的表单认证之外， Laravel 同样提供一个使用 [Laravel Socialite](https://github.com/laravel/socialite) 的简单方便的 OAuth 认证方式。Socialite 现在支持 Facebook， Twitter， LinkedIn， Google， Github 和 Bitbucket 的认证方式。

在你的 `composer.json` 文件中添加以下依赖来开始使用 Socialite：

    composer require laravel/socialite

### 配置

在安装 Socialite 库之后，在你的 `config/app.php` 配置文件中注册 `Laravel\Socialite\SocialiteServiceProvider`：

    'providers' => [
        // Other service providers...

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

同样，在 `app` 配置文件的 `aliases` 数组中添加 `Socialite` facade：

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

你同样需要在应用中添加 OAuth 服务的认证。这些认证应该位于你的 `config/services.php` 配置文件中，并且应当使用 `facebook`, `twitter`, `linkedin`, `google`, `github` 或 `bitbucket` 作为配置的键，并且这些认证依赖于你应用定义的提供者，例如：

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### 基础用法

然后，你就可以认证用户了！你需要两条路由：一个是将用户重定向到 OAuth 提供者，另一个接受提供者验证后的回调。我们将会通过使用 `Socialite` [facade](/docs/{{version}}/facades) 访问 Socialite。

    <?php

    namespace App\Http\Controllers;

    use Socialite;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect` 方法会发送用户信息给 OAuth 提供者，而 `user` 方法会读取到来的请求并且接收提供者的用户信息。在重定向用户直线，你也可以使用 `scope` 方法设置请求的 “作用域”。这个方法会覆写所有存在的作用域：

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

当然，你需要定义好你的控制器方法的路由：

        Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
        Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');

A number of OAuth providers support optional parameters in the redirect request. To include any optional parameters in the request, call the `with` method with an associative array:

    return Socialite::driver('google')
                ->with(['hd' => 'example.com'])->redirect();

#### 接收用户详情

一旦你有了用户的实例，你可以抓取到更多的用户信息：

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-authentication-drivers"></a>
## 添加自定义认证驱动

如果你没有使用传统的关系型数据库去存储你的用户，你需要扩展 Laravel 来耦合你自己的认证驱动。我们将会使用 `Auth` facade 的 `extend` 方法去定义一个自定义的驱动。你需要将这个调用放置在一个[服务提供者](/docs/{{version}}/providers)的 `extend` 下：

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('riak', function($app) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

当你用 `extend` 方法注册完你的驱动后，你可以在 `config/auth.php` 配置文件中切换到你的新驱动了。

### 用户提供者 Contract

`Illuminate\Contracts\Auth\UserProvider` 实现仅仅负责从持久化存储(例如 MySQL 和 Riak 等等)中取 `Illuminate\Contracts\Auth\Authenticatable` 的实现。这两个接口允许 Laravel 认证机制不管用户数据如何存储或者由什么类代表而继续工作。

让我们看一眼 `Illuminate\Contracts\Auth\UserProvider` contract：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 方法接受一个代表用户的键，比如 MySQL 数据库中的自增 ID。匹配 ID 的 `Authenticatable` 实现应该被取回并且通过方法返回。

`retrieveByToken` 方法通过用户独特的 `$identifier` 和 储存在 `remember_token` 字段的 “记住我” `$token` 来获取用户。和前一个方法一样， 应当返回 `Authenticatable` 的实现。

`updateRememberToken` 方法用新的 `$token` 更新 `$user` 的 `remember_token` 字段。新 token 可以是一个通过成功的"记住我"登录而新生成的 token，或者当用户登出时置为空。

`retrieveByCredentials` 方法接受一个当尝试在应用程序中注册时传给 `Auth::attemp` 方法的认证数组。这个方法应当继续“查询”下层的持久存储用户是否存在。典型地，这个方法会运行一个在 `$credentials['username']` 的 "where" 条件查询。这个方法应该返回一个 `UserInterface` 的实现。**这个方法不应该去做任何的密码验证或者认证。**

`validateCredentials` 方法应当比较传入的 `$user` 和 `$credentials` 来进行用户认证。例如，这个方法可能将 `$user->getAuthPassword()` 字符串和一个 `$credentials['password']` 的 `Hash::make` 进行比较。这个方法应当仅仅验证用户是否有权限登录。并且返回一个布尔值。

### 认证 Contract

既然我们已经探索了 `UserProvider` 下的每一个方法，我们下面来看一看 `Authenticatable`。记住，提供者应当返回这个接口 `retrieveById` 和 `retrieveByCredentials` 方法的实现：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

这个接口非常简单。`getAuthIdentifier` 方法应当返回一个用户的“主键”。在 MySQL 后端，同样的，这个会是一个自增的主键。 `getAuthPassword` 应当返回一个用户哈希过后的密码。这个接口允许认证系统和任何 User 类进行工作，而不用关心 ORM 或者所使用的是何种存储层。默认地， Laravel 在 `app` 文件夹下包含了一个实现了这个接口的 `User` 类， 所以你可以查看这个类作为一个实现例子。
