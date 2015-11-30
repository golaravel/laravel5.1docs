# 请求的生命周期

- [介绍](#introduction)
- [生命周期简述](#lifecycle-overview)
- [聚焦于服务提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 介绍

在现实世界中使用某样工具时，如果理解它是如何运作的，那么你使用起来就会更有自信。应用程序的开发也是一样。当你理解了开发工具是如何工作的，你在使用的时候会更加自信自如。

这份文档的目的，是提供给您一个关于 Laravel 框架如何‘运作’的概述。通过更好的了解整个框架，你就会少一些“神奇”的感觉并且在开发应用时更具自信。

如果你现在还不理解所有的术语，不要灰心！争取对讲到的东西有一个基本的掌握，你的知识将会随着你对其它部分的探索不断成长。

<a name="lifecycle-overview"></a>
## 生命周期简述

### 第一要务

`public/index.php` 文件是一个Laravel应用程序的所有请求的入口。通过你的web服务器配置，所有的请求都被引导到该文件。该`index.php`文件中并没有多少代码。相反地，它只是一个起始点，用于加载框架的其他部分。

该 `index.php` 文件加载Composer生成的autoloader定义，并且从 `bootstrap/app.php` 脚本中获取一个Laravel应用程序的实例。Laravel自身的第一个动作就是创建一个应用程序 ／ [服务容器](/docs/{{version}}/container) 的实例。

### HTTP / Console Kernels

接下来，根据进入应用程序的请求的类型，该请求被发送给 HTTP kernel 类或者 console kernel 类，这两种核心类是所有请求流向的中心位置。现在暂时我们只关注 HTTP kernel 类, 它保存在 `app/Http/Kernel.php`。

该 HTTP kernel 类继承了 `Illuminate\Foundation\Http\Kernel` 类，它定义了一个名为 `bootstrappers` 的数组，该数组会在请求执行之前运行。该 bootstrappers 数组会配置错误处理，配置日志，[检测应用程序环境](/docs/{{version}}/installation#environment-configuration)，以及执行其它需要在处理请求之前完成的任务。

该 HTTP kernel 类同时也定义了一个 HTTP [中间件](/docs/{{version}}/middleware) 的列表，所有的请求在被应用程序处理前都必须经过它们。这些中间件处理读取和写入[HTTP session]，决定应用是否处于维护模式，[验证 CSRF 标记](/docs/{{version}}/routing#csrf-protection)，以及其它功能.

HTTP kernel 类的 `handle` 方法声明很简单：接收一个 `请求` 并且返回一个 `响应`。你可以将 Kernel 想象成一个黑盒，它代表你的整个应用。给它HTTP请求，它返回HTTP响应。

#### 服务提供者

加载 [service providers](/docs/{{version}}/providers) 是 Kernel 启动过程中最重要的动作之一。应用程序的所有服务提供者（service providers）都在配置文件 `config/app.php` 中的 `providers` 数组中进行设置。首先，对所有提供者调用`register` 方法，一旦所有的提供者都被注册，将会接着调用`boot`方法。

服务提供者负责引导框架的各种组件，比如数据库，队列，验证，路由组件等。因为框架所提供的所有功能都是由它们来引导和设置，服务提供者是整个Laravel启动过程中最重要的一部分。

#### 分发请求

当应用程序已经启动并且所有的服务提供者都已经注册，`Request` 将被转交给路由器进行分发。路由器将把请求分发给路由或者控制器，并执行路由所指定的中间件。

<a name="focus-on-service-providers"></a>
## 聚焦于服务提供者

服务提供者是启动一个Laravel应用程序的真正核心所在。创建一个应用实例，注册服务提供者，并将请求交给已启动的应用。真的就这么简单！

能够牢固地掌握Laravel应用是如何通过服务提供者来创建并启动，是非常有价值的。你的应用程序的默认服务提供者都存放于 `app/Providers` 目录中。

默认情况下，`AppServiceProvider` 几乎是空的。你可以在此添加应用程序自己的启动和服务容器的绑定。当然，对于大型的应用，你会希望创建多个服务提供者，每个服务提供者提供更加独立精细的启动类型。
