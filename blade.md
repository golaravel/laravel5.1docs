# Blade 模板

- [简介](#introduction)
- [模板继承](#template-inheritance)
    - [定义一个页面布局模板](#defining-a-layout)
    - [扩展一个页面布局模板](#extending-a-layout)
- [展示数据](#displaying-data)
- [控制结构](#control-structures)
- [Service Injection](#service-injection)
- [扩展 Blade](#extending-blade)

<a name="introduction"></a>
## 简介

Blade 是 Laravel 提供的一个既简单又强大的模板引擎。和其他流行的 PHP 模板引擎不一样，Blade 并不限制你在视图（view）中使用原生 PHP 代码。所有 Blade 视图页面都将被编译成原生 PHP 代码并缓存起来，除非你的模板文件被修改了，否则不会重新编译，这就意味着 Blade 基本上不会给你的应用增加任何额外负担。Blade 视图文件使用 `.blade.php` 文件扩展名，并且一般被存放在 `resources/views` 目录。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义一个页面布局模板

使用 Blade 能获得的两个主要好处是 _模板继承（template inheritance）_ 和 _视图片断（sections）_。我们先通过一个简单的实例认识一下 Blade 模板吧。首先，我们先来看一下命名为 "master" 的页面布局模板。由于大部分网站应用都会在页面之间共享某些相同的布局，所以将这个页面布局定义在一个单独的 Blade 模板中更便于使用。 

    <!-- Stored in resources/views/layouts/master.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

如上所见，这个文件中包含了通常见到的 HTML 标签。不过，请注意一下 `@section` 和 `@yield` 指令。`@section` 指令正像其名字所暗示的一样是用来定义一个视图片断（section）的；`@yield` 指令是用来展示某个指定 section 所代表的内容的。

好了，我们已经定义了一个页面布局模板了，接下来我们定义一个继承页面布局模板的子页面模板。

<a name="extending-a-layout"></a>
### 扩展一个页面布局模板

定义子页面时，你需要使用 Blade 提供的 `@extends` 指令来为子页面指定其所“继承”的页面布局模板。Views which `@extends` a Blade layout may inject content into the layout's sections using `@section` directives. Remember, as seen in the example above, the contents of these sections will be displayed in the layout using `@yield`:

    <!-- Stored in resources/views/layouts/child.blade.php -->

    @extends('layouts.master')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

在上面的实例中，`sidebar` section is utilizing the `@@parent` directive to append (rather than overwriting) content to the layout's sidebar. The `@@parent` directive will be replaced by the content of the layout when the view is rendered.

Of course, just like plain PHP views, Blade views may be returned from routes using the global `view` helper function:

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## 展示数据

You may display data passed to your Blade views by wrapping the variable in "curly" braces. For example, given the following route:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以像下面这样展示 `name` 变量所代表的内容：

    Hello, {{ $name }}.

当然，你不仅仅被局限于展示传递给视图的变量，你还可以展示 PHP 函数的返回值。事实上，你可以在 Blade 模板的双花括号表达式中调用任何 PHP 代码：

    The current UNIX timestamp is {{ time() }}.

> **注意：** Blade 中的 `{{ }}` 表达式的返回值将被自动传递给 PHP 的 `htmlentities` 函数进行处理，以防止 XSS 攻击。

#### Blade & JavaScript 框架

由于很多 JavaScript 框架也使用 "花括号" 来表示一个需要在浏览器端解析的表达式，因此，你可以通过使用 `@` 符号来告知 Blade 模板引擎保持后面的表达式原样输出。如下所示：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

在上述实例中，`@` 符号将被 Blade 移除，同时，`{{ name }}` 表达式将被原样输出，这就能够确保在浏览器端被你的 JavaScript 框架解释了。

#### 如果数据存储就将其输出

某些时候你需要输出变量的值，但是在输出前你并不知道这个变量是否被赋值了。我们将这个问题化作代码描述如下：

    {{ isset($name) ? $name : 'Default' }}

不过，上面的表达式的确很繁复，幸好 Blade 提供了如下的速记方式：

    {{ $name or 'Default' }}

在这个实例中，如果 `$name` 变量被赋了值，它的值将被输出出来。然而，如果没有被赋值，将输出默认的单词 `Default` 。

#### 展示未转义的数据

默认情况下，Blade 的 `{{ }}` 表达式会自动通过 PHP 的 `htmlentities` 函数进行处理，以防止 XSS 攻击。如果你不希望自己的数据被转义，请使用如下语法：

    Hello, {!! $name !!}.

> **注意：** 务必要对用户上传的数据小心处理。最好对内容中的所有 HTML 使用双花括号进行输出，这样就能转义内容中的任何 HTML 标签了。

<a name="control-structures"></a>
## Control Structures

In addition to template inheritance and displaying data, Blade also provides convenient short-cuts for common PHP control structures, such as conditional statements and loops. These short-cuts provide a very clean, terse way of working with PHP control structures, while also remaining familiar to their PHP counterparts.

#### If 表达式

通过 `@if`、`@elseif`、`@else` 和 `@endif` 指令可以创建 `if` 表达式。这些指令其实都有相对应的 PHP 表达式：

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

为了使用方面，Blade 还提供了 `@unless` 指令：

    @unless (Auth::check())
        You are not signed in.
    @endunless

#### 循环

除了状态表达式，Blade 还提供了简单的循环指令，与 PHP 所支持的循环结构相一致。并且，每一个指令的功能都和相对应的 PHP 表达式是一致的。

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

#### 引入子视图

Blade 提供的 `@include` 指令允许你方便地在一个视图中引入另一个视图。所有父视图中可用的变量也都可以在被引入的子视图中使用。

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

虽然被引入的子视图能够访问父视图的所有可用数据，但是，你还可以向被引入的子视图传递额外的数据：

    @include('view.name', ['some' => 'data'])

#### 注释

Blade 还允许你在视图中添加注释。然而，和 HTML 注释不同的是，Blade 注释不会出现在最终生成的 HTML 中。

    {{-- This comment will not be present in the rendered HTML --}}

<a name="service-injection"></a>
## Service Injection

The `@inject` directive may be used to retrieve a service from the Laravel [service container](/docs/{{version}}/container). The first argument passed to `@inject` is the name of the variable the service will be placed into, while the second argument is the class / interface name of the service you wish to resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## 扩展 Blade

Blade even allows you to define your own custom directives. You can use the `directive` method to register a directive. When the Blade compiler encounters the directive, it calls the provided callback with its parameter.

下面这个实例将创建一个 `@datetime($var)` 指令，用于格式化 `$var` 参数：

    <?php

    namespace App\Providers;

    use Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function($expression) {
                return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
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

As you can see, Laravel's `with` helper function was used in this directive. The `with` helper simply returns the object / value it is given, allowing for convenient method chaining. The final PHP generated by this directive will be:

    <?php echo with($var)->format('m/d/Y H:i'); ?>


