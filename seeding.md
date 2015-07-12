# 数据库: Seeding

- [简介](#introduction)
- [编写 Seeders](#writing-seeders)
    - [使用模型工厂](#using-model-factories)
    - [调用额外的 Seeders](#calling-additional-seeders)
- [执行 Seeders](#running-seeders)

<a name="introduction"></a>
## 简介

Laravel包含一个使用Seeder类将测试数据填充至你的数据库的简单方法。所有的Seeder类都放在 `database/seeds` 中。Seeder类名可自定义，但最好遵守一个合理的约定，比如 `UserTableSeeder` 等等。默认为你定义了一个 `DatabaseSeeder` 类。在此类中，你可以调用 `call` 方法来控制执行其他Seeder的顺序。

<a name="writing-seeders"></a>
## 编写 Seeders

你可以执行 `make:seeder` [Artisan 命令](/docs/{{version}}/artisan) 来生成一个Seeder。所有通过框架生成的Seeder都会被放在 `database/seeders` 目录中：

    php artisan make:seeder UserTableSeeder

一个Seeder类默认只包含一个方法：`run` 。当执行 `db:seed` [Artisan 命令](/docs/{{version}}/artisan) 时该方法被调用。在 `run` 方法中，你可以将自己需要的数据插入到数据库中。你可以使用 [query builder](/docs/{{version}}/queries) 或者 [Eloquent model factories](/docs/{{version}}/testing#model-factories) 手动插入数据。

举一个例子，让我们来修改Laravel默认的 `DatabaseSeeder` 类。在 `run` 方法中添加一条数据库插入语句。

    <?php

    use DB;
    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### 使用模型工厂

当然，手动为每个模型Seeder指定属性是很麻烦的。除此之外，你可以使用 [模型工厂](/docs/{{version}}/testing#model-factories) 来方便地生成大量的数据记录。首先，回顾一下 [模型工厂](/docs/{{version}}/testing#model-factories) 是如何定义你的工厂的。一旦定义好工厂，就可以用 `factory` 帮助函数来把记录插入到你的数据库中。

例如，让我们创建50个用户然后为每个用户建立联系：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory('App\User', 50)->create()->each(function($u) {
            $u->posts()->save(factory('App\Post')->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 调用额外的 Seeders

在 `DatabaseSeeder` 类中，你可以调用 `call` 方法来执行额外的Seeder类。使用 `call` 方法允许你将数据库填充操作拆分到多个文件中来避免单一Seeder类变得太臃肿。只需要传入你需要执行的Seeder类名即可：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        $this->call(UserTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);
    }

<a name="running-seeders"></a>
## 执行 Seeders

一旦编写好了你的Seeder类，就可以使用 `db:seed` Artisan 命令来填充你的数据库。默认的， `db:seed` 命令执行 `DatabaseSeeder` 类，该类可能会执行其他Seeder类。然而，你可以用 `--class` 选项来单独执行特定的Seeder类：

    php artisan db:seed

    php artisan db:seed --class=UserTableSeeder

你也可以用 `migrate:refresh` 命令来填充你的数据库，这将会回滚并重新迁移你的所有migrations。这个命令十分有利于完整地重建你的数据库：

    php artisan migrate:refresh --seed
