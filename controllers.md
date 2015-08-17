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

你可能更希望把请求处理逻辑交由控制器类来处理，而不是都交给一个 `routes.php` 文件。控制器可以将相应的 HTTP 请求逻辑集合到一个类里面。控制器类一般存放在 `app/Http/Controllers` 目录下。

<a name="basic-controllers"></a>
## 基础控制器

如下是一个基础控制器类的例子。在 Laravel 的默认安装中，所有的控制器都继承了控制器基类。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
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

我们可以像下面一样路由至控制器动作方法当中：

    Route::get('user/{id}', 'UserController@showProfile');

现在，当一个请求匹配到一个特定的路由URI时，`UserController` 类的 `showProfile` 方法将会被执行。当然，路由的参数 `id` 也会传递到相应的方法当中。

#### 控制器 & 命名空间

有一点是必须特别注意的，那就是当我们在定义一个控制器路由的时候，我们无需指定一个完整的控制器命名空间。我们只需要定义紧跟根命名空间之后的那一部分名称。在默认情况下，`RouteServiceProvider` 将会读取包含在根控制器命名空间下的路由集合。

如果你选择利用 PHP 的命名空间机制以嵌套的方式组织控制器在 `App\Http\Controllers` 目录下的结构的话，引用类时只需指定相对于 `App\Http\Controllers` 根命名空间的类名即可。因此，如果你的控制器类的全名是 `App\Http\Controllers\Photos\AdminController` 的话，只需像下面这样注册路由即可：

    Route::get('foo', 'Photos\AdminController@method');

#### 命名控制器路由

你也许希望像闭包路由一样指定控制器路由的名称：

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

一旦你给控制器路由分配了一个名称，为控制器动作生成 URL 将会变得非常简单。使用 `action` 助手方法来为控制器动作生成 URL。此外，我们只需要指定紧跟基目录空间 `App\Http\Controllers` 后面的控制器类名即可：

    $url = action('FooController@method');

你也可以使用 `route` 助手方法来为一个命名过的控制器路由生成 URL：

    $url = route('name');

<a name="controller-middleware"></a>
## Controller Middleware 控制器中间件

你可以像下面的例子一样把[中间件](/docs/{{version}}/middleware)指派给控制器路由：

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

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
## RESTful 资源控制器

资源控制器可以让你快捷的创建 RESTful 控制器。举个例子，你也许希望创建一个控制器来处理所有有关于“照片”的 HTTP 请求，这些“照片”都是你的应用储存起来的。通过使用 Artisan 命令 `make:controller`，我们可以快速的创建一个控制器：

    php artisan make:controller PhotoController

这个 Artisan 命令将会在 `app/Http/Controllers/` 目录下生成一个控制器文件`PhotoController.php`。这个控制器将会包含为每一种可用资源操作的方法。

接下来，你需要注册一个指向控制器的资源路由：

    Route::resource('photo', 'PhotoController');

单这一行的路由声明就在照片资源上创建了许多处理各种各样 RESTful 请求的路由。同样的，在生成的控制器里面，已经拥有了所有相关的控制器方法，包括一些注释来告诉你这些方法对应的 URI 和动词。


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
#### 局部资源路由

W当声明一个资源路由的时候，你也许会在路由上指定了一个动作的子集来处理请求：

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### 命名资源路由

在默认情况下，所有的资源控制器动作都会有一个路由名称。然而，你也可以通过传递一个 `names` 数组来重写这些名称。

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### 嵌套资源

有时候，你可能需要为一个“嵌套”资源定义路由规则。例如，一个照片资源可能会附加有很多的“评论”。为了“嵌套”到资源路由器，可以在你的路由声明中使用`.`号来实现：

    Route::resource('photos.comments', 'PhotoCommentController');

这条路由将会注册了一个“嵌套”资源，你可以通过的 URL 来进行访问：`photos/{photos}/comments/{comments}`。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class PhotoCommentController extends Controller
    {
        /**
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
#### 扩展资源控制器

如果有需要在资源控制器默认路由的基础上添加额外的路由，那么你应该在调用 `Route::resource` 之前就定义好那些路由。否则，`resource` 方法定义的路由的优先级会高于你添加的扩展路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## 隐式控制器

Laravel 可以让你只需要定义一条路由规则，就可以在一个控制器类中轻松的处理所有动作。首先，使用`Route::controller` 方法定义一条路由。这个 `controller` 方法接收两个参数。第一个参数是控制器处理的基础 URI，第二个参数则是控制器的类名。

    Route::controller('users', 'UserController');

 接着，只需要在你的控制器中加入方法。方法的名字应当以它们所响应的 HTTP 动词开头、然后是首字母大写的URI名称：

    <?php

    namespace App\Http\Controllers;

    class UserController extends Controller
    {
        /**
         * 响应／users的GET请求
         */
        public function getIndex()
        {
            //
        }

        /**
         * 响应／users／show／1的GET请求
         */
        public function getShow($id)
        {
            //
        }

        /**
         * 响应／users/admin－profile的GET请求
         */
        public function getAdminProfile()
        {
            //
        }

        /**
         * 响应／users／profile的POST请求
         */
        public function postProfile()
        {
            //
        }
    }

正如上面的例子所示，控制器里的 `index` 方法将会响应根 URI，也就是在这个例子当中的 `users`。

#### 指定路由名称

如果你想要为控制器自定义[命名](/docs/{{version}}/routing#named-routes)路由，你可以通过给`controller`方法的第三个参数传递一个名称数组来实现：

    Route::controller('users', 'UserController', [
        'getShow' => 'user.show',
    ]);

<a name="dependency-injection-and-controllers"></a>
## 依赖注入 & 控制器

#### 构造器注入

Laravel [服务容器](/docs/{{version}}/container) 是用来转换所有 Laravel 控制器的。正因如此，你可以在控制器的构造器当中对依赖关系进行类型限制。依赖关系会被自动的转换和注入到控制器实例当中：


    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户仓库实例。
         */
        protected $users;

        /**
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

当然，你也可以对 [Laravel contract](/docs/{{version}}/contracts) 进行类型限制。只要容器能够对它进行转换，你就可以这么做。

#### 方法注入

作为依赖注入的补充，你也可以在你的控制器动作方法中对依赖关系进行类型限制。例如，我们可以像如下所示例子那样，对 `Illuminate\Http\Request` 实例进行类型限制：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
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

如果你的控制器方法如果预料到需要接收一个路由参数，你只需要把这个路由参数放到其它依赖对象后面即可。例如，如果你是这样定义路由的：

    Route::put('user/{id}', 'UserController@update');

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
## 路由缓存

如果你的应用软件只使用控制器的基本路由，那么你可以充分利用 Laravel 的路由缓存。使用路由缓存可以极大的减少应用程序注册所有路由的时间。在某些情况下，路由的注册过程可以提速100倍以上！那么要怎么样生成路由缓存呢？你只需要执行 Artisan 命令 `route:cache` 即可：

    php artisan route:cache

没错，就是这么简单！你的路由缓存文件将会替代原有的路由文件 `app/Http/routes.php` 而被使用。请牢记，如果你增加了新的路由，那么你必须重新生成一个新的路由缓存。正因如此，在项目部署的时候才运行生成缓存的命令 `route:cache` 显得更加的明智。

如果你想在不再生成新的路由缓存的情况下删除路由缓存的话，直接执行 `route:clear` 命令：

    php artisan route:clear


