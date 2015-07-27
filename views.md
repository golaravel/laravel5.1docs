# 视图

- [基本用法](#basic-usage)
    - [向视图传递数据](#passing-data-to-views)
    - [为所有视图共享数据](#sharing-data-with-all-views)
- [视图组件](#view-composers)

<a name="basic-usage"></a>
## 基本用法

视图包含了应用程序渲染的HTML数据，并将应用程序的显示逻辑与控制逻辑有效的分离开。在Laravel中，视图被保存在`resources/views`目录中。

一个简单的视图看起来是这样的：

    <!-- View stored in resources/views/greeting.php -->

    <html>
        <body>
            <h1>Hello, <?php echo $name; ?></h1>
        </body>
    </html>

将上面的视图代码被保存在`resources/views/greeting.php`文件中，我们就可以使用`view`辅助方法来将这个视图作为响应返回：

    Route::get('/', function ()    {
        return view('greeting', ['name' => 'James']);
    });

正如你所见到的，传递给`view`辅助方法的第一个参数对应视图文件在`resources/views`目录中的名称。传递给`view`辅助方法的第二个参数是一个能在视图内取用的数据数组。

当然，视图可以被嵌套保存在`resoureces/views`目录的子目录中，"."号被用来引用嵌套的视图。例如，可以通过下面语句引用`resoureces/views/admin/profile.php`这个视图：

    return view('admin.profile', $data);

#### 判断视图是否存在

如果需要判断视图是否存在，可以在不带参数的`view`辅助方法后使用`exists`方法。如果视图存在，该方法会返回`true`：

    if (view()->exists('emails.customer')) {
        //
    }

当不带参数的`view`辅助方法被调用时，会返回一个`Illuminate\Contracts\View\Factory`实例，通过这个实例你可以调用视图工厂（View Factory）的所有方法。

<a name="view-data"></a>
### 视图数据

<a name="passing-data-to-views"></a>
#### 向视图传递数据

像前面例子那样，你可以很容易的向视图传送一组数据。

    return view('greetings', ['name' => 'Victoria']);

当使用这种办法向视图传递数据时，`$data`应该是 键/值 对应的数组数据。在视图中，你可以使用对应的键来获取值，像这样`<?php echo $key; ?>`，`{{ $key }}`会输出`$data[$key]`对应的数据。

同样的，你也可以使用`with`方法向视图传递数据，在下面的例子中，可以使用`{{ $name }}`来输出`Victoria`。

    $view = view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### 把数据共享给所有的视图

有时候，你可能需要共享一份数据给所有的视图，你应该使用视图工厂（View Factory）的`share`方法。通常，我们在服务提供者的`boot`方法中调用`share`方法来完成注册。如果需要同时共享多份数据，你可以将所有对`share`方法的调用放在一个服务提供者中（`AppServiceProvider`或者其他），或者单独的为每个需要共享的数据建立一个服务提供者。

    <?php

    namespace App\Providers;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->share('key', 'value');
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

<a name="view-composers"></a>
## 视图组件

视图组件是视图被渲染时被调用的闭包或者类方法。你可以通过视图组件将某些数据跟视图绑定到一起，这样数据就会在视图被渲染的时候自动加载。视图组件可以帮助你将这一功能的实现逻辑组织到同一个地方。

首先，在[服务提供者](/docs/{{version}}/providers)中注册我们的视图组件，并使用`view`辅助方法获取底层`Illuminate\Contracts\View\Factory`合约的实现（contract implementation）。在Laravel中，没有为视图组件设置一个默认的目录，所以我们可以将视图组件的文件任意喜欢的位置。例如，你可以新建一个`App\Http\ViewComposer`目录来保存这些文件。

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            view()->composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            view()->composer('dashboard', function ($view) {

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

> **注意：** 在建立的服务提供者中注册视图组件后，应该同时把这个服务提供者添加到`config/app.php`配置文件的`providers`数组中，来完成服务提供者的注册，服务提供者在`providers`数组中注册后才可以被应用程序调用。

现在，我们已经成功的注册了视图组件，每当`profile`视图被渲染的时候，`ProfileComposer@compose`方法都会被调用。我们来实现这个视图组件类：

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\Contracts\View\View;
    use Illuminate\Users\Repository as UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

在视图被渲染之前，`ProfileComposer`的`compose`方法会被调用，同时传入一个`Illuminate\Contracts\View\View`实例。你可以在`View`的实例上使用`with`方法将数据绑定到视图。

> **注意：** 所有的视图组件会通过[服务容器](/docs/{{version}}/container)进行解析,所以你可以在视图组件的构造函数中声明你所需的依赖，如上面代码中的`UserRepository`依赖。

#### 把视图组件绑定到多个视图

如果你需要将视图组件绑定到多个视图，可以将包含视图名称的数组作为第一个参数传入`composer`方法：

    view()->composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

`composer`方法接受`*`作为通配符，可以使用`*`将视图组件绑定到所有的视图。

    view()->composer('*', function ($view) {
        //
    });

### 视图创建者

视图创建者（View **creators**）跟视图组件几乎完全一样，不同的是视图创建者是在视图实例化之后立即执行，而视图组件在视图被渲染的时候才会执行。注册一个视图创建者可以使用`creator`方法：

    view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
