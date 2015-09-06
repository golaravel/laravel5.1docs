# 辅助函数

- [简介](#introduction)
- [现有方法](#available-methods)

<a name="introduction"></a>
## 简介

Laravel包含许多PHP辅助函数。框架自身使用了许多这些函数；不过，如果觉得方便，也可以自由地在你的应用中使用它们。

<a name="available-methods"></a>
## 现有方法

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 数组

<div class="collection-method-list" markdown="1">
[array_add](#method-array-add)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-recursive)
[array_where](#method-array-where)
[head](#method-head)
[last](#method-last)
</div>

### 路径

<div class="collection-method-list" markdown="1">
[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[elixir](#method-elixir)
[public_path](#method-public-path)
[storage_path](#method-storage-path)
</div>

### 字串

<div class="collection-method-list" markdown="1">
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)
</div>

### URL

<div class="collection-method-list" markdown="1">
[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[url](#method-url)
</div>

### 其他

<div class="collection-method-list" markdown="1">
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[method_field](#method-method-field)
[old](#method-old)
[redirect](#method-redirect)
[response](#method-response)
[value](#method-value)
[view](#method-view)
[with](#method-with)
</div>

<a name="method-listing"></a>
## 方法列表

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## 数组

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

`array_add` 函数向数组中添加一个键-值对（如果给定的键不存在）：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 返回两个数组，一个包含原数组的所有键，另一个包含原数组的所有值：

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 函数将一个多维数组转换为一维数组，并使用点号指示深度：

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 方法从一个数组中移除指定的键-值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 方法返回数组中第一个通过判断返回为真的元素：

    $array = [100, 200, 300];

    $value = array_first($array, function ($key, $value) {
        return $value >= 150;
    });

    // 200

默认值可作为第三个参数传入。如果没有值通过判断，将返回默认值：

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 方法将一个多维数组转换为一维数组：

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 方法用点号从一个深度嵌套的数组中移除指定的键-值对：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 方法用点号从一个深度嵌套的数组中取出值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get` 方法也接受默认值，如果指定的键未找到，返回默认值：

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 方法从给定的数组中返回指定的键-值对：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 方法从给定的数组中拉出键-值对：

    $array = [
        ['developer' => ['name' => 'Taylor']],
        ['developer' => ['name' => 'Abigail']]
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 方法从数组中移除并返回一个键-值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 方法用点号为一个深度嵌套的数组设置值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 方法依据给定闭包的返回值排序数组：

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive` 方法用 `sort` 函数递归排序数组：

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 用给定闭包过滤数组：

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($key, $value) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函数简单返回给定数组的第一个元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回给定数组的最后一个元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 函数返回 `app` 目录的绝对路径：

    $path = app_path();

你也可以用 `app_path` 函数来生成相对于应用目录的文件的绝对路径：

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 返回项目根目录的绝对路径：

    $path = base_path();

你也可以用 `base_path` 函数来生成相对于应用目录的文件的绝对路径：

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函数返回应用配置目录的绝对路径：

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数返回应用数据库目录的绝对路径：

    $path = database_path();

<a name="method-elixir"></a>
#### `elixir()` {#collection-method}

The `elixir` function gets the path to the versioned [Elixir](/docs/{{version}}/elixir) file:

    elixir($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数返回 `public` 目录的绝对路径：
function returns the fully qualified path to the `public` directory:

    $path = public_path();

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数返回 `storage` 目录的绝对路径：
function returns the fully qualified path to the `storage` directory:

    $path = storage_path();

你也可以用 `storage_path` 函数来生成相对于storage目录的文件的绝对路径：

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字串

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函数将给定字串转换为驼峰式：

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 返回删除了名字空间的类名：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函数为给定的字串调用 `htmlentities` ：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函数判断字串是否以给定值结尾：

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 将给定字串转换为蛇形式：

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 函数限制一个字符串的长度。该函数接收一个字符串作为第一个参数，最大长度作为第二个参数：

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函数判断字串是否以给定值开头：

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 判断字串是否包含给定值：

    $value = str_contains('This is my name', 'my');

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数为字串添加给定单例：

    $string = str_finish('this/string', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函数判断字串是否匹配给定形式。星号表示通配符：

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 函数将字串转换为其复数形式。该函数目前仅支持英文：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函数生成指定长度的随机字符串：

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函数将字串转换为其单数形式。该函数目前仅支持英文：

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数将字串转换为URL友好型：
function generates a URL friendly "slug" from the given string:

    $title = str_slug("Laravel 5 Framework", "-");

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函数将字串转换为 `StudlyCase` 型：

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数使用你的 [本地化文件](/docs/{{version}}/localization) 翻译给定的语句：

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数随词形变化翻译给定的语句：

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数为给定控制器动作生成URL。无需给控制器传入完整的名字空间。相反，传入相对于 `App\Http\Controllers` 名字空间的控制器类名：

    $url = action('HomeController@getIndex');

如果方法接收路由参数，可以将它们作为第二个参数传入该函数：

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Generate a URL for an asset using the current scheme of the request (HTTP or HTTPS):

	$url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Generate a URL for an asset using HTTPS:

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数为给定名称的路由生成URL：

    $url = route('routeName');

如果路由接收参数，可以将它们作为第二个参数传入该函数：

    $url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函数为给定路径生成绝对URL：

    echo url('user/profile');

    echo url('user/profile', [1]);

<a name="miscellaneous"></a>
## 其他

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 函数返回一个认正器实例。可用来简化使用 `Auth` 门面：

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back()` 函数为用户的前一个位置生成一个重定向响应：

    return back();

<a href="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 函数使用Bcrypt计算给定值的哈希值。可用来替换使用 `Hash` 门面：

    $password = bcrypt('my-secret-password');

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函数获取配置变量的值。可用点号语法（包含文件名和你希望访问的选项）访问配置值。必须指定默认值，它将在配置项不存在时返回：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

The `config` helper may also be used to set configuration variables at runtime by passing an array of key / value pairs:

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 生成一个包含CSRF令牌的HTML隐藏 `input` 。例如，使用 [Blade 语法](/docs/{{version}}/blade):

    {!! csrf_field() !!}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函数取回当前CSRF令牌值：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出给定变量然后结束脚本的执行：

    dd($value);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函数获取一个环境变量的值或者返回默认值：

    $env = env('APP_ENV');

    // 如果变量不存在，返回默认值
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数分派给定的 [event](/docs/{{version}}/events) 到其监听器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 函数为给定类创建一个模型工厂。当在 [testing](/docs/{{version}}/testing#model-factories) 或 [seeding](/docs/{{version}}/seeding#using-model-factories) 时可使用:

    $user = factory(App\User::class)->make();

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field` 函数生成一个包含表单HTTP谓词假值的HTML `hidden` input域。例如，使用 [Blade 语法](/docs/{{version}}/blade):

    <form method="POST">
        {!! method_field('delete') !!}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函数用于 [取回](/docs/{{version}}/requests#retrieving-input) 一个闪存入 session 的旧输入值：

    $value = old('value');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函数返回一个重定向器的实例来进行 [重定向](/docs/{{version}}/responses#redirects):

    return redirect('/home');

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 函数创建一个 [响应](/docs/{{version}}/responses) 实例或者从响应工厂取得一个实例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 函数将简单地返回你给它的值。然后，如果为它传入一个 `Closure` ，`Closure` 将被执行然后返回它的结果：

    $value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数取回一个 [视图](/docs/{{version}}/views) 实例：

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

`with` 函数返回你给它的值。这个函数主要对本不可能的链式操作十分有用。

    $value = with(new Foo)->work();
