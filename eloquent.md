# Eloquent：入门

- [简介](#introduction)
- [定义模型（model）](#defining-models)
    - [Eloquent Model Conventions](#eloquent-model-conventions)
- [Retrieving Multiple Models](#retrieving-multiple-models)
- [Retrieving Single Models / Aggregates](#retrieving-single-models)
    - [Retrieving Aggregates](#retrieving-aggregates)
- [Inserting & Updating Models](#inserting-and-updating-models)
    - [Basic Inserts](#basic-inserts)
    - [Basic Updates](#basic-updates)
    - [Mass Assignment](#mass-assignment)
- [Deleting Models](#deleting-models)
    - [Soft Deleting](#soft-deleting)
    - [Querying Soft Deleted Models](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes)
- [Events](#events)

<a name="introduction"></a>
## 简介

Laravel 所自带的 Eloquent ORM 是一个优美、简洁的 ActiveRecord 实现，用来实现数据库操作。每个数据表都有一个与之相对应的“模型（Model）”，用于和数据表交互。模型（model）帮助你在数据表中查询数据，以及向数据表内插入新的记录. 

在开始之前，请务必在 `config/database.php` 文件中正确配置数据库的连接参数。如需更多数据库配置方面的信息，请查看[此文档](/docs/{{version}}/database#configuration)。

<a name="defining-models"></a>
## 定义模型（model）

开始讲解前，我们先来创建一个 Eloquent 模型（model）。模型（model）文件通常被放在 `app` 目录下，但是，你也可以将其放置于任何地方，只要能够通过 `composer.json` 配置文件自动加载即可。所有的 Eloquent 模型（model）都继承自 `Illuminate\Database\Eloquent\Model` 类。

创建一个模型（model）实例的最简单方法是使用 [Artisan 命令行工具](/docs/{{version}}/artisan) 的 `make:model` 指令：

    php artisan make:model User

如果你希望在生成模型（model）的同时生成 [数据库将迁移](/docs/{{version}}/schema#database-migrations) ，可以通过添加 `--migration` 或 `-m` 参数来实现：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型规范

现在，让我们来看一个 `Flight` 模型类（model class），我们用它从 `flights` 数据表中存取信息：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### 数据表的表名

注意，我们并没有告诉 Eloquent 将 `Flight` 模型（model）和哪个数据表进行关联。默认的规则是：类名的复数形式用来当做数据表的表名，除非明确指定另一个名称。所以，在这种情况下，Eloquent 将自动推导出 `Flight` 模型与 `flights` 数据表关联。你可以在模型（model）中定义一个 `table` 属性，用来指定另一个数据表名称：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### 主键

Eloquent will also assume that each table has a primary key column named `id`. You may define a `$primaryKey` property to override this convention.
Eloquent 假定每一个数据表中都存在一个命名为 `id` 的列作为主键。你可以通过定义一个 `$primaryKey` 属性来明确指定一个主键。

#### 时间戳

默认情况下，Eloquent 期望数据表中存在 `created_at` 和 `updated_at` 字段。如果你不希望由 Eloquent 来管理这两个字段，可以在模型（model）中将 `$timestamps` 属性设置为 `false`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果你需要定制时间戳的格式，可以通过在模型（model）中设置 `$dateFormat` 属性来实现。这个属性决定了日期属性如何在数据库中存储，也决定当模型（model）被序列化为数组 或者 JSON 格式时日期属性的格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="retrieving-multiple-models"></a>
## 获取多个模型

一旦你创建了一个模型（model）并将其[关联到了一个数据表](/docs/{{version}}/schema)，你就可以从数据库中获取数据了。将每一个 Eloquent 模型（model）想象为一个强大的[查询构造器](/docs/{{version}}/queries)，该查询构造器能够帮你通过模型（model）从数据库中查询需要的数据。例如：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Show a list of all available flights.
         *
         * @return Response
         */
        public function index()
        {
            $flights = Flight::all();

            return view('flight.index', ['flights' => $flights]);
        }
    }

#### 获取字段的值

对于任何一个 Eloquent 模型（model）实例，都可以将字段名当做模型（model）的属性，从而相对应的属性来获取对应的字段的 值，例如，我们将查询之后获得的所有 `Flight` 实例挨个遍历，并且输出每个实例的 `name` 字段的值:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Adding Additional Constraints

The Eloquent `all` method will return all of the results in the model's table. Since each Eloquent model serves as a [query builder](/docs/{{version}}/queries), you may also add constraints to queries, and then use the `get` method to retrieve the results:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> **Note:** Since Eloquent models are query builders, you should review all of the methods available on the [query builder](/docs/{{version}}/queries). You may use any of these methods in your Eloquent queries.

#### Collections

For Eloquent methods like `all` and `get` which retrieve multiple results, an instance of `Illuminate\Database\Eloquent\Collection` will be returned. The `Collection` class provides [a variety of helpful methods](/docs/{{version}}/eloquent-collections#available-methods) for working with your Eloquent results. Of course, you may simply loop over this collection like an array:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Chunking Results

If you need to process thousands of Eloquent records, use the `chunk` command. The `chunk` method will retrieve a "chunk" of Eloquent models, feeding them to a given `Closure` for processing. Using the `chunk` method will conserve memory when working with large result sets:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

The first argument passed to the method is the number of records you wish to receive per "chunk". The Closure passed as the second argument will be called for each chunk that is retrieved from the database.

<a name="retrieving-single-models"></a>
## Retrieving Single Models / Aggregates

Of course, in addition to retrieving all of the records for a given table, you may also retrieve single records using `find` and `first`. Instead of returning a collection of models, these methods return a single model instance:

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

#### Not Found Exceptions

Sometimes you may wish to throw an exception if a model is not found. This is particularly useful in routes or controllers. The `findOrFail` and `firstOrFail` methods will retrieve the first result of the query. However, if no result is found, a `Illuminate\Database\Eloquent\ModelNotFoundException` will be thrown:

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

If the exception is not caught, a `404` HTTP response is automatically sent back to the user, so it is not necessary to write explicit checks to return `404` responses when using these methods:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Retrieving Aggregates

Of course, you may also use the query builder aggregate functions such as `count`, `sum`, `max`, and the other aggregate functions provided by the [query builder](/docs/{{version}}/queries). These methods return the appropriate scalar value instead of a full model instance:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Inserting & Updating Models

<a name="basic-inserts"></a>
### 基本的插入操作

如需在数据库中新建一条记录，只要简单地新建一个模型（model）实例，然后为此实例设置属性，最后调用 `save` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

在上面这个例子中，我们把通过 HTTP 请求传进来的 `name` 参数直接赋值给 `App\Flight` 模型实例的 `name` 属性。当我们调用 `save` 方法时就会向数据库中插入一条记录。当调用 `save` 方法时 `created_at` 和 `updated_at` 时间戳就会被自动更新，不需要我们自己动手。

<a name="basic-updates"></a>
### 基本的更新操作

`save` 方法可以用于更新数据库中已经存在的模型（model）。为了更新一个模型（model），首先你必须从数据库中将其取出，然后为需要更新的属性赋值，最后调用 `save` 方法。此外，`updated_at` 时间戳会被自动更新，所以不需要手动设置 `updated_at` 的值：

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

更新操作也可以在符合指定查询条件的多个模型实例上进行。在下面例子中，所有标记为 `active` 并且 `destination` 是 `San Diego` 的飞机都被标记为延时：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

传递给 `update` 方法的参数必须是一个数组，数组中包含的是一系列键值对，分别对应需要更新的字段名称和更新后的值。

<a name="mass-assignment"></a>
### 批量赋值

You may also use the `create` method to save a new model in a single line. The inserted model instance will be returned to you from the method. However, before doing so, you will need to specify either a `fillable` or `guarded` attribute on the model, as all Eloquent models protect against mass-assignment.

A mass-assignment vulnerability occurs when user's pass unexpected HTTP parameters through a request, and then that parameter changes a column in your database you did not expect. For example, a malicious user might send an `is_admin` parameter through an HTTP request, which is then mapped onto your model's `create` method, allowing the user to escalate themselves to an administrator.

So, to get started, you should define which model attributes you want to make mass assignable. You may do this using the `$fillable` property on the model. For example, let's make the `name` attribute of our `Flight` model mass assignable:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Once we have made the attributes mass assignable, we can use the `create` method to insert a new record in the database. The `create` method returns the saved model instance:

    $flight = App\Flight::create(['name' => 'Flight 10']);

While `$fillable` serves as a "white list" of attributes that should be mass assignable, you may also choose to use `$guarded`. The `$guarded` property should contain an array of attributes that you do not want to be mass assignable. All other attributes not in the array will be mass assignable. So, `$guarded` functions like a "black list". Of course, you should use either `$fillable` or `$guarded` - not both:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

In the example above, all attributes **except for `price`** will be mass assignable.

#### Other Creation Methods

There are two other methods you may use to create models by mass assigning attributes: `firstOrCreate` and `firstOrNew`. The `firstOrCreate` method will attempt to locate a database record using the given column / value pairs. If the model can not be found in the database, a record will be inserted with the given attributes.

The `firstOrNew` method, like `firstOrCreate` will attempt to locate a record in the database matching the given attributes. However, if a model is not found, a new model instance will be returned. Note that the model returned by `firstOrNew` has not yet been persisted to the database. You will need to call `save` manually to persist it:

    // Retrieve the flight by the attributes, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve the flight by the attributes, or instantiate a new instance...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## Deleting Models

To delete a model, call the `delete` method on a model instance:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Deleting An Existing Model By Key

In the example above, we are retrieving the model from the database before calling the `delete` method. However, if you know the primary key of the model, you may delete the model without retrieving it. To do so, call the `destroy` method:

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### Deleting Models By Query

Of course, you may also run a delete query on a set of models. In this example, we will delete all flights that are marked as inactive:

    $deletedRows = App\Flight::where('active', 0)->delete();

<a name="soft-deleting"></a>
### Soft Deleting

In addition to actually removing records from your database, Eloquent can also "soft delete" models. When models are soft deleted, they are not actually removed from your database. Instead, a `deleted_at` attribute is set on the model and inserted into the database. If a model has a non-null `deleted_at` value, the model has been soft deleted. To enable soft deletes for a model, use the `Illuminate\Database\Eloquent\SoftDeletes` trait on the model and add the `deleted_at` column to your `$dates` property:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

Of course, you should add the `deleted_at` column to your database table. The Laravel [schema builder](/docs/{{version}}/schema) contains a helper method to create this column:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

Now, when you call the `delete` method on the model, the `deleted_at` column will be set to the current date and time. And, when querying a model that uses soft deletes, the soft deleted models will automatically be excluded from all query results.

To determine if a given model instance has been soft deleted, use the `trashed` method:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models

#### Including Soft Deleted Models

As noted above, soft deleted models will automatically be excluded from query results. However, you may force soft deleted models to appear in a result set using the `withTrashed` method on the query:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

The `withTrashed` method may also be used on a [relationship](/docs/{{version}}/eloquent-relationships) query:

    $flight->history()->withTrashed()->get();

#### Retrieving Only Soft Deleted Models

The `onlyTrashed` method will retrieve **only** soft deleted models:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Restoring Soft Deleted Models

Sometimes you may wish to "un-delete" a soft deleted model. To restore a soft deleted model into an active state, use the `restore` method on a model instance:

    $flight->restore();

You may also use the `restore` method in a query to quickly restore multiple models:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Like the `withTrashed` method, the `restore` method may also be used on [relationships](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### Permanently Deleting Models

Sometimes you may need to truly remove a model from your database. To permanently remove a soft deleted model from the database, use the `forceDelete` method:

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## Query Scopes

Scopes allow you to define common sets of constraints that you may easily re-use throughout your application. For example, you may need to frequently retrieve all users that are considered "popular". To define a scope, simply prefix an Eloquent model method with `scope`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Utilizing A Query Scope

Once the scope has been defined, you may call the scope methods when querying the model. However, you do not need to include the `scope` prefix when calling the method. You can even chain calls to various scopes, for example:

    $users = App\User::popular()->women()->orderBy('created_at')->get();

#### Dynamic Scopes

Sometimes you may wish to define a scope that accepts parameters. To get started, just add your additional parameters to your scope. Scope parameters should be defined after the `$query` argument:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Now, you may pass the parameters when calling the scope:

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent models fire several events, allowing you to hook into various points in the model's lifecycle using the following methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Events allow you to easily execute code each time a specific model class is saved or updated in the database.

Eloquent 模型（model）能够触发多个事件，通过调用下面所列出的方法，你可以在模型（model）的生命周期中的每个“关键点”上执行自己的代码，从而影响模型（model）的执行流程：`creating`、`created`、`updating`、`updated`、`saving、`saved`、`deleting`、`deleted`、`restoring`、`restored`。每当某个模型（model）被保存或更新到数据库中时，你都能通过事件轻松地插入自己编写的代码并让它执行。

<a name="basic-usage"></a>
### 基本用法

每当一个新模型（model）头一次被保存时，都将触发 `creating` 和 `created` 事件。 如果模型（model）已经存在于数据库中，并且 `save` 方法被调用了，将触发 `updating` / `updated` 事件。无论如何，`saving` / `saved ` 事件都会被触发。

例如，我们在一个[服务提供者（service provider）](/docs/{{version}}/providers) 中定义一个 Eloquent 事件监听器。在我们的事件监听器中，我们要在给定的模型（model）上调用 `isValid` 方法，如果该模型（model）是无效的，则返回 `false` 。如果 Eloquent 事件监听器中返回的是 `false` ，将取消 `save` / `update` 操作：

    <?php

    namespace App\Providers;

    use App\User;
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
            User::creating(function ($user) {
                if ( ! $user->isValid()) {
                    return false;
                }
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
