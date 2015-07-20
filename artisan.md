# Artisan 控制台

- [简介](#introduction)
- [创建命令](#writing-commands)
    - [命令结构](#command-structure)
- [命令的输入输出](#command-io)
    - [定义输入期望](#defining-input-expectations)
    - [获取输入](#retrieving-input)
    - [在输入时提示](#prompting-for-input)
    - [输出信息](#writing-output)
- [注册命令](#registering-commands)
- [在代码中调用命令](#calling-commands-via-code)

<a name="introduction"></a>
## 简介

Artisan是Laravel中自带的命令行工具的名称。它提供了一些对您的应用开发有帮助的命令。它是由强大的Symfony Console组件驱动的。为了查看所有可用的Artisan的命令，您可以使用`list`命令来列出它们：

    php artisan list

每一个命令都包含有”help”信息，用来显示和描述这个命令可用的参数和选项。为了查看帮助信息，仅需在命令前添加`help`即可：

    php artisan help migrate

<a name="writing-commands"></a>
## 创建命令

除了Artisan自带的命令，您也可以创建与您的应用相关的自定义命令。这些命令将会被储存在 `app/Console/Commands` 文件夹下，然而，只要您的命令可以被 `composer.json` 自动载入，您可以自由地选择命令的存放位置。

要创建一个新命令，您可以使用 `make:console` 这个Artisan命令，这将会生成一个命令的模板文件帮助您进行开发：

    php artisan make:console SendEmails

上面的命令会在 `app/Console/Commands/SendEmails.php` 文件中生成一个类。在创建命令时，加上 `--command` 这个选项，将可以指定这个终端命令的名称，即在运行Artisan命令的时候所输入的名称：


    php artisan make:console SendEmails --command=emails:send

<a name="command-structure"></a>
### 命令结构

一旦您的命令生成后，您需要填写对应命令类中的 `signature` 和 `description` 属性，这将会在 `list` 命令清单上显示您的命令。

`handle` 方法会在您的命令执行时被调用。您可以在这个方法中加入命令的具体逻辑。下面是一个自定义命令的例子。

既然我们要所有我们所需要的依赖注入到命令的构造器中。Laravel的[服务容器](/docs/{{version}}/container) 将会在构造器中自动的注入所有类型约束的依赖。为了更好的代码复用性，保持您的控制台命令简洁并且让它们由应用服务去处理完成它们的任务是一个很好的实践方法。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;
    use Illuminate\Foundation\Inspiring;

    class Inspire extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="command-io"></a>
## 命令的输入输出

<a name="defining-input-expectations"></a>
### 定义输入期望

在写控制台命令的代码的时候时，收集用户输入的参数或选项是一项很普遍的事情。Laravel通过使用您命令中的 `signature` 属性来使得定义您所期望的用户输入的形式变得非常方便。`signature` 属性允许您用一种简洁、富有表现力、仿路由的语法去定义命令的名称、参数、和选项。

所有用户提供的参数和选项被大括号所包裹，如下所示：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

在这个例子中，命令定义了一个**必填**的参数: `user`。您也可以将其改为可选参数或者为可选参数定义一个默认值：

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

选项，像参数一样，也是用户输入的一种。但是，当它们在命令行中指定时，它们的前缀是由两个连字符构成。我们可以在`signature`中像这样定义选项：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中，`--queue` 开关在调用Artisan命令的时候被指定。如果 `--queue` 开关参数被传入，这个选项的值即为 `true`，否则，这个选项的值就为 `false`：

    php artisan email:send 1 --queue

您也可以指定在选项后面加 `=` 号来让用户来分配一个值，表明这个值是会被用户所提供的：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中，用户可能会像这样传入选项的值：

    php artisan email:send 1 --queue=default

您也可以像这样来给选项的值赋一个默认值

    email:send {user} {--queue=default}

#### 输入描述

您可以用一个冒号分隔来对输入的参数和选项进行具体的描述：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="retrieving-input"></a>
### 获取输入

当您的命令执行的时候，很显然需要得到命令输入的参数和选项的值。要得到它们的话，您可以使用 `argument` 和 `option` 方法：

使用 `argument` 方法获得参数的值

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果需要得到所有参数的值，调用不带参数的 `argument` 方法就可以得到一个参数值的数组：

    $arguments = $this->argument();

得到选项的值的方法和得到参数的方法一样简单，使用 `option` 方法即可。和 `argument` 方法一样，您也可以调用不带参数的 `option` 方法来获得所有选项值的一个数组：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->option();

如果参数或者选项不存在，方法将返回 `null`。

<a name="prompting-for-input"></a>
### 询问时输入

除了显示输出，您有可能需要询问用户去提供命令的输入。`ask` 方法会提示设定好的信息显示给用户，并且获取他们的输入，然后返回用户在命令中的输入：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法和 `ask` 方法一样，但是用户的输入不会显示在控制台中。这个方法对用户输入的敏感信息比如密码非常有用：

    $password = $this->secret('What is the password?');

#### 提示用户进行确认

如果您需要让用户确认的话，您可以使用 `confirm` 方法。默认情况下，这个方法会返回 `false`。但是，如果用户输入 `y` ，这个方法会返回 `true`。

    if ($this->confirm('Do you wish to continue? [y|N]')) {
        //
    }

#### 提示用户进行选择

`anticipate` 方法可以被用来为可能的选择提供自动补全。无论选项是什么，用户仍然可以使用任何答案。

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

如果您需要给用户一些预设好的选项，您可以使用 `choice` 方法。用户选择答案的索引，但是方法会返回答案的值。如果没有一个选项被选择的话，您可以设置一个默认值。

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### 输出信息

使用 `info`，`comment`，`question` 和`error`方法将输出显示在控制台上。每一种方法会得到一个对应的ANSI颜色。

使用 `info` 方法来向用户显示信息。典型的，这个将会在控制台显示绿色字符：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

使用 `error` 方法显示错误信息。错误信息会显示绿色字符：

    $this->error('Something went wrong!');

#### 输出表格

使用`table` 方法会让您正确地格式化数据的行列。仅需向方法传入表格的头部。表格的宽和高将会由给出的数据动态的计算出来：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对于执行时间较长的任务，显示进度是非常有必要的。使用 `output` 对象，我们可以开始，推进和停止进度条。当您开始进度的时候，您必须定义执行的步数，然后在每一步执行结束后推进进度：

    $users = App\User::all();

    $this->output->progressStart(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $this->output->progressAdvance();
    }

    $this->output->progressFinish();

更高级的选项，可以查看[Symfony Progress Bar component documentation](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## 注册命令

一旦完成了命令的编写，您需要将它和Artisan注册在一起，这样它才能够正常使用。书写的代码在 `app/Console/Kernel.php` 文件中完成。

在这个文件中，您会在 `commands` 属性中发现一个命令的列表。简单地将您的命令的类名添加到列表中就可以注册您的命令了。当Artisan启动的时候，所有在这个属性中列出来的命令将会被[service container](/docs/{{version}}/container)解析，并且在Artisan中注册。

    protected $commands = [
        'App\Console\Commands\SendEmails'
    ];

<a name="calling-commands-via-code"></a>
## 在代码中调用命令

有时候您期望在CLI外执行Artisan命令。例如，您可能期望在路由或者控制器中执行一个Artisan命令。您可以使用在 `Artisan` Facade 中的 `call` 方法去完成这件事情。`call`方法第一个参数接受命令的名称，第二个参数接受一个由参数组成的数组。这个方法将会返回一个返回值。

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

使用 `Artisan` Facade 中的 `queue` 命令可以让您在后台的队列[queue workers](/docs/{{version}}/queues)中执行您的命令：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果您需要强制指定一个不接受字符串选项的值，比如在 `migrate:refresh` 命令中的 `--force` 标志，您可以传入布尔值 `true` 或 `false` 的参数。

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### 在其他命令中调用命令

有时候你可能希望调用其他在已存在的Artisan命令中的命令。您可以用 `call` 方法来这样做。这个 `call` 方法接受命令名和一个由参数构成的数组参数。

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

如果您期望调用其他控制台命令并且隐藏它所有的输出，您可以使用 `callSilent` 方法。`callSilent` 方法和 `call` 方法的使用方法一样： 

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
