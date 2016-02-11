# 授权

- [简介](#introduction)
- [定义权限](#defining-abilities)
- [检查权限](#checking-abilities)
	- [通过Gate Facade](#via-the-gate-facade)
	- [通过User模型](#via-the-user-model)
	- [使用Blade模版](#within-blade-templates)
    - [使用Form请求](#within-form-requests)
- [策略](#policies)
	- [创建策略](#creating-policies)
	- [编写策略](#writing-policies)
	- [检查策略](#checking-policies)
- [控制器授权](#controller-authorization)

<a name="introduction"></a>
## 简介

除了提供“开箱即用”的[授权](/docs/{{version}}/authentication)服务外，Laravel还提供一种简单的方式来管理授权逻辑和资源权限。这里有多种方法和帮助函数来辅助你管理你的授权逻辑，下文将对所有涉及到的方法为你一一道来。

> **注意:** 授权服务在Laravel 5.1.11时被加入，在将这些功能集成到你的应用中前请参考[升级指南](/docs/{{version}}/upgrade)。

<a name="defining-abilities"></a>
## 定义权限

对于一个用户可能执行的某种动作，判断它的权限的最简单的途径就是使用`Illuminate\Auth\Access\Gate`类。而定义所有这些权限最方便的地方就是Laravel自带的`AuthServiceProvider`。举例来说，我们定义一个`update-post`权限来接收当前的`User`和一个`Post`[模型](/docs/{{version}}/eloquent)。在我们定义的这个权限里，我们得判断用户的`id`是否和post的`user_id`匹配：

	<?php

	namespace App\Providers;

	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * Register any application authentication / authorization services.
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);

	        $gate->define('update-post', function ($user, $post) {
	        	return $user->id === $post->user_id;
	        });
	    }
	}

值得注意的一点是我们并没有检查给定的`$user`是否为`NULL`。因为在当前用户不是授权用户或者没有`forUser`方法的使用权限的情况下`Gate`都将为所有权限返回`false`。

#### 基于类的权限

除了上例中注册为授权回调函数闭包外，你还可以传递权限名和一个“类名@方法名”的字符串来注册权限方法。事后这个类会通过[服务容器](/docs/{{version}}/container)被解析：

    $gate->define('update-post', 'Class@method');

<a name="intercepting-all-checks"></a>
<a name="intercepting-authorization-checks"></a>
#### 拦截权限检查

某些时候，你可能希望给用户授予所有权限。为满足这种需求，你可以使用`before`方法来定义一个回调函数，这个回调函数会在所有授权检查之前被调用：

    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

如果`before`回调函数返回非null值，那么这个返回值就是权限检查结果。

用膝盖联想一下自然也应该存在一个`after`方法来以相同的方式定义回调函数，只不过是在所有检查被执行之后被调用。当然，在`after`函数内你也可以对检查结果不做任何修改：

    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="checking-abilities"></a>
## 检查权限

<a name="via-the-gate-facade"></a>
### 通过Gate Facade

权限定义后我们可以有多种姿势去“检查”它。首先，我们可以使用`Gate` [facade](/docs/{{version}}/facades)的`check`，`allows`或者`denies`方法。这三种方法接收的参数都是权限名和传递给权限函数的参数。你 **不需要** 在参数中传递当前的用户，因为`Gate`会自动把当前用户一起传递给权限的回调函数。所以，在检查上例中定义的`update-post`权限时，我们只需要在参数中传递一个`Post`实例给`denies`方法：

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

依然是用膝盖联想一下，`allows`方法必定和`denies`方法相反，当动作通过授权时返回`true`。至于这个`check`方法嘛，它只是`allows`方法的一个别名。

#### 检查任意用户的权限

如果你想用`Gate` facade来检查非当前用户是否有给定的权限，可以使用`forUser`方法：

	if (Gate::forUser($user)->allows('update-post', $post)) {
		//
	}

#### 传递多个参数

如果你想定义一个传入多个参数的权限回调函数，也是可以的：

	Gate::define('delete-comment', function ($user, $post, $comment) {
		//
	});

如果你的权限回调函数像上面那样定义了多个参数，在权限检查时你需要把回调函数的参数放入数组再传入`Gate`的方法中：

	if (Gate::allows('delete-comment', [$post, $comment])) {
		//
	}

<a name="via-the-user-model"></a>
### 通过User模型

让咱换种姿势：通过`User`模型来检查权限。默认情况下，Laravel的`App\User`模型使用的`Authorizable` trait提供了两种方法：`can`和`cannot`。这两种方法的使用和`Gate` facade的`allows`，`denies`方法相似，我们仅仅需要在上文的例子中稍作改动：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
        	$post = Post::findOrFail($id);

        	if ($request->user()->cannot('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

这次不用想也知道了吧，`can`方法和`cannot`方法相反：

	if ($request->user()->can('update-post', $post)) {
		// Update Post...
	}

<a name="within-blade-templates"></a>
### 使用Blade模版

如果你还嫌上文介绍的姿势不够便利，别急，Laravel还提供了Blade指令`@can`来快速检查当前用户是否有给定的权限。举个栗子：

	<a href="/post/{{ $post->id }}">View Post</a>

	@can('update-post', $post)
		<a href="/post/{{ $post->id }}/edit">Edit Post</a>
	@endcan

你还可以把`@can`指令和`@else`指令一并使用：

	@can('update-post', $post)
		<!-- The Current User Can Update The Post -->
	@else
		<!-- The Current User Can't Update The Post -->
	@endcan

<a name="within-form-requests"></a>
### 使用Form请求

最后，你可以在[form请求](/docs/{{version}}/validation#form-request-validation)的`authorize`方法中使用你在`Gate`中定义的权限。例如：

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policies"></a>
## 策略

<a name="creating-policies"></a>
### 创建策略

如果把应用中所有的授权逻辑都定义在`AuthServiceProvider`里会让整个应用显得臃肿不堪，Laravel允许你把授权逻辑拆分到"策略"类中。策略是用于组织管理授权逻辑的PHP原生类。

首先，让我们生成一个策略来管理`Post`模型的授权。你可以使用[artisan命令](/docs/{{version}}/artisan)`make:policy`来生成策略。生成的策略会被放在`app/Policies`目录：

	php artisan make:policy PostPolicy

#### 注册策略

策略生成后，我们需要在`Gate`类中注册它。`AuthServiceProvider`包含了`policies`属性用以映射实体和管理该实体的策略。然后，我们指定`Post`模型的策略就是`PostPolicy`类：

	<?php

	namespace App\Providers;

	use App\Post;
	use App\Policies\PostPolicy;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * The policy mappings for the application.
	     *
	     * @var array
	     */
	    protected $policies = [
	        Post::class => PostPolicy::class,
	    ];

	    /**
	     * Register any application authentication / authorization services.
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);
	    }
	}

<a name="writing-policies"></a>
### 编写策略

策略被生成且注册后，我们就能添加每一种权限的授权方法了。例如，在我们的`PostPolicy`定义一个`update`方法，该方法能判断给定的用户是否有权限"update"一个`Post`：

	<?php

	namespace App\Policies;

	use App\User;
	use App\Post;

	class PostPolicy
	{
		/**
		 * Determine if the given post can be updated by the user.
		 *
		 * @param  \App\User  $user
		 * @param  \App\Post  $post
		 * @return bool
		 */
	    public function update(User $user, Post $post)
	    {
	    	return $user->id === $post->user_id;
	    }
	}

你可以继续在策略中定义方法用来检查必要的权限。例如，你可以定义`show`,`destroy`,或者`addComment`方法来授权各种`Post`的动作。

> **注意:** 所有的策略都会被Laravel的[服务容器](/docs/{{version}}/container)解析，这意味着你可以在策略类的构造函数中类型提示任何依赖，它们将会自动被注入。

#### 拦截所有检查

有时，你可能希望给一个用户授予所有权限。在这种情况下可以在策略中定义一个`before`方法，这个方法会在本策略中所有授权检查完成后被调用：

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

如果`before`方法返回一个非null值，那么这个值就是权限检查结果。

<a name="checking-policies"></a>
### 检查策略

策略方法的调用和授权回调函数闭包的调用完全相同。你可以使用`Gate` facade，`User` model，Blade指令`@can`，或者`策略`帮助函数。

#### 通过Gate Facade

`Gate`会自动根据传入参数判断应该使用哪一个策略。所以，如果我们传入一个`Post`实例到`denies`方法，`Gate`会使用与之相对应的`PostPolicy`来授权相应动作：

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

#### 通过User模型

当给定的参数允许时，`User`模型的`can`和`cannot`方法会自动使用策略。对于应用获取的任何`User`实例，这两种方法提供了一种方便的方式来授权动作。

	if ($user->can('update', $post)) {
		//
	}

	if ($user->cannot('update', $post)) {
		//
	}

#### 使用Blade模版

同样的，在给定参数允许时，Blade指令`@can`会使用策略：

	@can('update', $post)
		<!-- The Current User Can Update The Post -->
	@endcan

#### 通过策略帮助函数

全局的`策略`帮助函数可以用来根据传入参数获取`策略`类。例如，我们可以传递一个`Post`实例到`策略`帮助函数来得到一个对应的`PostPolicy`类：

	if (policy($post)->update($user, $post)) {
		//
	}

<a name="controller-authorization"></a>
## 控制器授权

默认情况下，Laravel自带的`App\Http\Controllers\Controller`基类使用`AuthorizesRequests` trait，提供了`authorize`方法，这个方法用以快速授权一个给定动作，如果动作没有被授权则抛出一个`HttpException`异常。

`authorize`方法的用法与其他授权方法诸如`Gate::allows`和`$user->can()`一样。我们使用`authorize`方法来快速授权一个更新`Post`的请求：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	$this->authorize('update', $post);

        	// Update Post...
        }
    }

如果这个动作通过授权，控制器将会继续执行；如果`authorize`方法判断出这个动作没有通过授权，则将抛出一个`HttpException`异常并且生成一个带有`403 Not Authorized`状态码的HTTP响应。显而易见，`authorize`是一种十分方便快捷的可抛出异常方法。

`AuthorizesRequests` trait还提供了`authorizeForUser`方法来授权非当前用户：

	$this->authorizeForUser($user, 'update', $post);

#### 自动判断策略方法

通常来说，一个策略的方法对应一个控制器的方法。例如，在上例的`update`方法中，控制器方法和策略方法拥有相同的方法名：`update`。

因此，Laravel允许你简单地传递实例到`authorize`方法，被授权的权限将会自动基于调用的方法名进行判断。因为`authorize`在控制器的`update`方法中被调用，所以`PostPolicy`的`update`方法也会被调用：

    /**
     * Update the given post.
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
    	$post = Post::findOrFail($id);

    	$this->authorize($post);

    	// Update Post...
    }
