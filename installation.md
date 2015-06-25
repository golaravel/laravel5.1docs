# 安装

- [Installation](#安装)
- [Configuration](#设置)
	- [基本设置](#basic-configuration)
	- [环境设置](#environment-configuration)
	- [Configuration Caching](#configuration-caching)
	- [访问配置值](#accessing-configuration-values)
	- [为应用命名](#naming-your-application)
- [维护模式](#maintenance-mode)

<a name="installation"></a>
## 安装

### 运行环境需求

Laravel 框架对系统环境有一些要求。当然，所有这些要求在 [Laravel Homestead](/docs/{{version}}/homestead) 虚拟机中都是预装好的：

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP 扩展
- PDO PHP 扩展
- Mbstring PHP 扩展
- Tokenizer PHP 扩展
</div>

<a name="install-laravel"></a>
### 安装 Laravel

Laravel 利用 [Composer](http://getcomposer.org) 来管理其自身的依赖包。因此，在使用 Laravel 之前，请务必确保在你的机器上已经安装了 Composer 。

#### 通过 Laravel 安装工具安装 Laravel

首先，使用 Composer 下载 Laravel 安装包：

	composer global require "laravel/installer=~1.1"

请确保将 `~/.composer/vendor/bin` 目录设置于你的 `PATH` 环境变量里， 这样 `laravel` 执行文件就就能被你的系统检测到了。

一旦安装完成后，就可以使用 `laravel new` 命令在你指定的目录中建立一份全新安装的 `Laravel` 应用。例如： `laravel new blog` 命令会在当前目录下建立一个名为 `blog` 的目录， 此目录里面存放着全新的 Laravel ，并且所有依赖包也已经安装好了。此方法的安装速度会比通过 Composer 安装快很多。

	laravel new blog

#### 通过 Composer Create-Project 命令安装 Laravel

还可以通过 Composer 的 `create-project` 命令来安装 Laravel：

	composer create-project laravel/laravel --prefer-dist

<a name="configuration"></a>
## 配置

<a name="basic-configuration"></a>
### 基本配置

Laravel 框架所用的所有配置文件都被存放在 `config` 目录下。每个配置项都有文档说明，所以请通读所有配置文件以熟悉所有可用的配置项。

#### 目录权限

安装 Laravel 之后，可能需要你配置一下目录权限。web 服务器需要拥有 `storage` 目录下的所有目录和 `bootstrap/cache` 目录的写权限。如果你在使用 [Homestead](/docs/{{version}}/homestead) 虚拟机，这些权限都已经帮你设置好了。

#### Application Key

安装 Laravel 之后接下来需要做的就是设置一个随机字串作为应用的 key。如果你是通过 Composer 或 Laravel 安装器安装的 Laravel，这个 key 已经由 `key:generate` 命令自动生成并设置了。一般情况下，这个作为 key 的字串的长度是 32 个字符。这个 key 还可以在 `.env` 环境配置文件中设置。如果你没有将 `.env.example` 文件改名为 `.env`，那就现在就做吧。**如果应用的 key 没有被配置，会话和其他需要加密的数据将不安全！**

#### 额外的配置

Laravel 开箱即用，几乎不需要什么配置。现在就可以开始你的开发之旅了！不过，建议你浏览一下 `config/app.php` 文件和此文件中的文档。它包含了几个配置项，例如 `timezone` 和 `locale` ，可能需要根据你自身的情况稍作修改。

你可能还需要为 Laravel 中的几个组件做些配置，例如：

- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [会话](/docs/{{version}}/session#configuration)

完成 Laravel 安装后，建议阅读 [配置你的本地开发环境](/docs/{{version}}/installation#environment-configuration).

<a name="pretty-urls"></a> 章节。
#### 美化链接

**Apache**

Laravel 框架自带了 `public/.htaccess` 文件用来从网址中删除 `index.php`。如果你用的是 Apache 来运行你的 Laravel 应用，请务必启用 Apache 的 `mod_rewrite` 模块。

如果 Laravel 自带的 `.htaccess` 文件在你的 Apache 中不起作用，请试一试下面的配置：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

**Nginx**

在 Nginx 中，将下面的指令放到站点配置文件中就可以实现美化链接的功能：

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

当然，如果你用的是 [Homestead](/docs/{{version}}/homestead)，美化链接的功能已经被自动配置好了。

<a name="environment-configuration"></a>
### 环境配置

It is often helpful to have different configuration values based on the environment the application is running in. For example, you may wish to use a different cache driver locally than you do on your production server. It's easy using environment based configuration.

To make this a cinch, Laravel utilizes the [DotEnv](https://github.com/vlucas/phpdotenv) PHP library by Vance Lucas. In a fresh Laravel installation, the root directory of your application will contain a `.env.example` file. If you install Laravel via Composer, this file will automatically be renamed to `.env`. Otherwise, you should rename the file manually.

All of the variables listed in this file will be loaded into the `$_ENV` PHP super-global when your application receives a request. You may use the `env` helper to retrieve values from these variables. In fact, if you review the Laravel configuration files, you will notice several of the options already using this helper!

Feel free to modify your environment variables as needed for your own local server, as well as your production environment. However, your `.env` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration.

If you are developing with a team, you may wish to continue including a `.env.example` file with your application. By putting place-holder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application.

#### Accessing The Current Application Environment

The current application environment is determined via the `APP_ENV` variable from your `.env` file. You may access this value via the `environment` method on the `App` [facade](/docs/{{version}}/facades):

	$environment = App::environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value. You may even pass multiple values if necessary:

	if (App::environment('local')) {
		// The environment is local
	}

	if (App::environment('local', 'staging')) {
		// The environment is either local OR staging...
	}

An application instance may also be accessed via the `app` helper method:

	$environment = app()->environment();

<a name="configuration-caching"></a>
### Configuration Caching

To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which can be loaded quickly by the framework.

You should typically run the `config:cache` command as part of your deployment routine.

<a name="accessing-configuration-values"></a>
### Accessing Configuration Values

You may easily access your configuration values using the global `config` helper function. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

	$value = config('app.timezone');

To set configuration values at runtime, pass an array to the `config` helper:

	config(['app.timezone' => 'America/Chicago']);

<a name="naming-your-application"></a>
### Naming Your Application

After installing Laravel, you may wish to "name" your application. By default, the `app` directory is namespaced under `App`, and autoloaded by Composer using the [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/). However, you may change the namespace to match the name of your application, which you can easily do via the `app:name` Artisan command.

For example, if your application is named "Horsefly", you could run the following command from the root of your installation:

	php artisan app:name Horsefly

Renaming your application is entirely optional, and you are free to keep the `App` namespace if you wish.

<a name="maintenance-mode"></a>
## 维护模式

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, an `HttpException` will be thrown with a status code of 503.

To enable maintenance mode, simply execute the `down` Artisan command:

	php artisan down

To disable maintenance mode, use the `up` command:

	php artisan up

### Maintenance Mode Response Template

The default template for maintenance mode responses is located in `resources/views/errors/503.blade.php`.

### Maintenance Mode & Queues

While your application is in maintenance mode, no [queued jobs](/docs/{{version}}/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.
