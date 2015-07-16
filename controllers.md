# HTTP 控制器

- [简介](#introduction)
- [基础控制器](#basic-controllers)
- [控制器中间件](#controller-middleware)
- [RESTful 资源控制器](#restful-resource-controllers)
    - [局部资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [嵌套资源](#restful-nested-resources)
    - [扩展资源路由](#restful-supplementing-resource-controllers)
- [隐式控制器](#implicit-controllers)
- [依赖注入 & 控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 简介

Instead of defining all of your request handling logic in a single `routes.php` file, you may wish to organize this behavior using Controller classes. Controllers can group related HTTP request handling logic into a class. Controllers are typically stored in the `app/Http/Controllers` directory.

你可能更希望把请求处理逻辑交由控制器类来处理，而不是都交给一个 `routes.php` 文件。控制器可以将相应的 HTTP 请求逻辑集合到一个类里面。控制器类一般存放在 `app/Http/Controllers` 目录下。

<a name="basic-controllers"></a>
## 基础控制器

Here is an example of a basic controller class. All Laravel controllers should extend the base controller class included with the default Laravel installation:

如下是一个基础控制器类的例子。在 Laravel 的默认安装中，所有的控制器都继承了控制器基类。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         * 显示指定用户的个人信息
         * 
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

We can route to the controller action like so:
我们可以像下面一样路由至控制器动作方法当中：

    Route::get('user/{id}', 'UserController@showProfile');

Now, when a request matches the specified route URI, the `showProfile` method on the `UserController` class will be executed. Of course, the route parameters will also be passed to the method.
现在，当一个请求匹配到一个特定的路由URI时，`UserController` 类的 `showProfile` 方法将会被执行。当然，路由的参数 `id` 也会传递到相应的方法当中。

#### Controllers & Namespaces 
#### 控制器 & 命名空间

It is very important to note that we did not need to specify the full controller namespace when defining the controller route. We only defined the portion of the class name that comes after the `App\Http\Controllers` namespace "root". By default, the `RouteServiceProvider` will load the `routes.php` file within a route group containing the root controller namespace.
有一点是必须特别注意的，那就是当我们在定义一个控制器路由的时候，我们无需指定一个完整的控制器命名空间。我们只需要定义紧跟根命名空间之后的那一部分名称。在默认情况下，`RouteServiceProvider` 将会读取包含在根控制器命名空间下的路由集合。

If you choose to nest or organize your controllers using PHP namespaces deeper into the `App\Http\Controllers` directory, simply use the specific class name relative to the `App\Http\Controllers` root namespace. So, if your full controller class is `App\Http\Controllers\Photos\AdminController`, you would register a route like so:


    Route::get('foo', 'Photos\AdminController@method');

#### Naming Controller Routes 
#### 命名控制器路由

Like Closure routes, you may specify names on controller routes:
你也许希望像闭包路由一样指定控制器路由的名称：

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

Once you have assigned a name to the controller route, you can easily generate URLs to the action. To generate a URL to a controller action, use the `action` helper method. Again, we only need to specify the part of the controller class name that comes after the base `App\Http\Controllers` namespace:
一旦你给控制器路由分配了一个名称，为控制器动作生成 URL 将会变得非常简单。使用 `action` 助手方法来为控制器动作生成 URL。此外，我们只需要指定紧跟基目录空间 `App\Http\Controllers` 后面的控制器类名即可：

    $url = action('FooController@method');

You may also use the `route` helper to generate a URL to a named controller route:
你也可以使用 `route` 助手方法来为一个命名过的控制器路由生成 URL：

    $url = route('name');

<a name="controller-middleware"></a>
## Controller Middleware 控制器中间件

[Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes like so:
你可以像下面的例子一样把[中间件](/docs/{{version}}/middleware)指派给控制器路由：

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller. You may even restrict the middleware to only certain methods on the controller class:
然而，在控制器的构造方法里面指派中间件会显得更加的方便。通过在控制器的构造器里面使用`中间件`方法，你会发现为控制器指派中间件是那么的简单。同时，你还可以限定中间件只对特定的方法起作用。

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         * 实例化一个新的UserController实例
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

            $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful Resource Controllers 
## RESTful 资源控制器

Resource controllers make it painless to build RESTful controllers around resources. For example, you may wish to create a controller that handles HTTP requests regarding "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:
资源控制器可以让你快捷的创建 RESTful 控制器。举个例子，你也许希望创建一个控制器来处理所有有关于“照片”的 HTTP 请求，这些“照片”都是你的应用储存起来的。通过使用 Artisan 命令 `make:controller`，我们可以快速的创建一个控制器：

    php artisan make:controller PhotoController

The Artisan command will generate a controller file at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.
这个 Artisan 命令将会在 `app/Http/Controllers/` 目录下生成一个控制器文件`PhotoController.php`。这个控制器将会包含为每一种可用资源操作的方法。

Next, you may register a resourceful route to the controller:
接下来，你需要注册一个指向控制器的资源路由：

    Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have methods stubbed for each of these actions, including notes informing you which URIs and verbs they handle.
单这一行的路由声明就在照片资源上创建了许多处理各种各样 RESTful 请求的路由。同样的，在生成的控制器里面，已经拥有了所有相关的控制器方法，包括一些注释来告诉你这些方法对应的 URI 和动词。


#### Actions Handled By Resource Controller
#### 资源管理器处理的动作

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

<a name="restful-partial-resource-routes"></a>
#### Partial Resource Routes
#### 局部资源路由

When declaring a resource route, you may specify a subset of actions to handle on the route:
当声明一个资源路由的时候，你也许会在路由上指定了一个动作的子集来处理请求：

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### Naming Resource Routes
#### 命名资源路由

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:
在默认情况下，所有的资源控制器动作都会有一个路由名称。然而，你也可以通过传递一个 `names` 数组来重写这些名称。

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### Nested Resources
#### 嵌套资源

Sometimes you may need to define routes to a "nested" resource. For example, a photo resource may have multiple "comments" that may be attached to the photo. To "nest" resource controllers, use "dot" notation in your route declaration:
有时候，你可能需要为一个“嵌套”资源定义路由规则。例如，一个照片资源可能会附加有很多的“评论”。为了“嵌套”到资源路由器，可以在你的路由声明中使用`.`号来实现：

    Route::resource('photos.comments', 'PhotoCommentController');

This route will register a "nested" resource that may be accessed with URLs like the following: `photos/{photos}/comments/{comments}`.
这条路由将会注册了一个“嵌套”资源，你可以通过的 URL 来进行访问：`photos/{photos}/comments/{comments}`。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class PhotoCommentController extends Controller
    {
        /**
         * Show the specified photo comment.
         * 显示指定的照片评论
         *
         * @param  int  $photoId
         * @param  int  $commentId
         * @return Response
         */
        public function show($photoId, $commentId)
        {
            //
        }
    }

<a name="restful-supplementing-resource-controllers"></a>
#### Supplementing Resource Controllers
#### 扩展资源控制器

If it becomes necessary to add additional routes to a resource controller beyond the default resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:
如果有需要在资源控制器默认路由的基础上添加额外的路由，那么你应该在调用 `Route::resource` 之前就定义好那些路由。否则，`resource` 方法定义的路由的优先级会高于你添加的扩展路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## Implicit Controllers
## 隐式控制器

Laravel allows you to easily define a single route to handle every action in a controller class. First, define the route using the `Route::controller` method. The `controller` method accepts two arguments. The first is the base URI the controller handles, while the second is the class name of the controller:
Laravel 可以让你只需要定义一条路由规则，就可以在一个控制器类中轻松的处理所有动作。首先，使用`Route::controller` 方法定义一条路由。这个 `controller` 方法接收两个参数。第一个参数是控制器处理的基础 URI，第二个参数则是控制器的类名。

    Route::controller('users', 'UserController');

 Next, just add methods to your controller. The method names should begin with the HTTP verb they respond to followed by the title case version of the URI:
 接着，只需要在你的控制器中加入方法。方法的名字应当以它们所响应的 HTTP 动词开头、然后是首字母大写的URI名称：

    <?php

    namespace App\Http\Controllers;

    class UserController extends Controller
    {
        /**
         * Responds to requests to GET /users
         * 响应／users的GET请求
         */
        public function getIndex()
        {
            //
        }

        /**
         * Responds to requests to GET /users/show/1
         * 响应／users／show／1的GET请求
         */
        public function getShow($id)
        {
            //
        }

        /**
         * Responds to requests to GET /users/admin-profile
         * 响应／users/admin－profile的GET请求
         */
        public function getAdminProfile()
        {
            //
        }

        /**
         * Responds to requests to POST /users/profile
         * 响应／users／profile的POST请求
         */
        public function postProfile()
        {
            //
        }
    }

As you can see in the example above, `index` methods will respond to the root URI handled by the controller, which, in this case, is `users`.
正如上面的例子所示，控制器里的 `index` 方法将会响应根 URI，也就是在这个例子当中的 `users`。

#### Assigning Route Names
#### 指定路由名称

If you would like to [name](/docs/{{version}}/routing#named-routes) some of the routes on the controller, you may pass an array of names as the third argument to the `controller` method:
如果你想要为控制器自定义[命名](/docs/{{version}}/routing#named-routes)路由，你可以通过给`controller`方法的第三个参数传递一个名称数组来实现：

    Route::controller('users', 'UserController', [
        'getShow' => 'user.show',
    ]);

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers
## 依赖注入 & 控制器

#### Constructor Injection
#### 构造器注入

The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor. The dependencies will automatically be resolved and injected into the controller instance:
Laravel [服务容器](/docs/{{version}}/container) 是用来转换所有 Laravel 控制器的。正因如此，你可以在控制器的构造器当中对依赖关系进行类型限制。依赖关系会被自动的转换和注入到控制器实例当中：


    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         * 用户仓库实例。
         */
        protected $users;

        /**
         * Create a new controller instance.
         * 创建一个新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it.
当然，你也可以对 [Laravel contract](/docs/{{version}}/contracts) 进行类型限制。只要容器能够对它进行转换，你就可以这么做。

#### Method Injection
#### 方法注入

In addition to constructor injection, you may also type-hint dependencies on your controller's action methods. For example, let's type-hint the `Illuminate\Http\Request` instance on one of our methods:
作为依赖注入的补充，你也可以在你的控制器动作方法中对依赖关系进行类型限制。例如，我们可以像如下所示例子那样，对 `Illuminate\Http\Request` 实例进行类型限制：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         * 储存一个新用户的信息。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies. For example, if your route is defined like so:
如果你的控制器方法如果预料到需要接收一个路由参数，你只需要把这个路由参数放到其它依赖对象后面即可。例如，如果你是这样定义路由的：

    Route::put('user/{id}', 'UserController@update');

You may still type-hint the `Illuminate\Http\Request` and access your route parameter `id` by defining your controller method like the following:
这时候，你仍然可以类型限制 `Illuminate\Http\Request`，同时，你可以用如下所示方法来定义你的控制器方法以访问 `id` 参数：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Route Caching
## 路由缓存

If your application is exclusively using controller based routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:
如果你的应用软件只使用控制器的基本路由，那么你可以充分利用 Laravel 的路由缓存。使用路由缓存可以极大的减少应用程序注册所有路由的时间。在某些情况下，路由的注册过程可以提速100倍以上！那么要怎么样生成路由缓存呢？你只需要执行 Artisan 命令 `route:cache` 即可：

    php artisan route:cache

That's all there is to it! Your cached routes file will now be used instead of your `app/Http/routes.php` file. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you may wish to only run the `route:cache` command during your project's deployment.
没错，就是这么简单！你的路由缓存文件将会替代原有的路由文件 `app/Http/routes.php` 而被使用。请牢记，如果你增加了新的路由，那么你必须重新生成一个新的路由缓存。正因如此，在项目部署的时候才运行生成缓存的命令 `route:cache` 显得更加的明智。

To remove the cached routes file without generating a new cache, use the `route:clear` command:
如果你想在不再生成新的路由缓存的情况下删除路由缓存的话，直接执行 `route:clear` 命令：

    php artisan route:clear


