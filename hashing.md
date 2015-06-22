# Hashing

- [简介](#introduction)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 简介

Laravel 自带的 `Hash` [facade](/docs/{{version}}/facades) 提供了采用 Bcrypt 加密算法的加密方式，用于存储用户密码。如果你正在使用 Laravel app 中自带的 `AuthController` 控制器（controller），它将自动为注册和验证过程采用 Bcrypt 算法。

对于加密密码来说，Bcrypt 是一个非常好的选择，因为它的 “加密系数（work factor）” 是可调整的，这就意味着，即便计算机硬件能力在逐步提升，通过调整 “加密系数”，计算一个哈希所消耗的时间也会随着提升，就是说破解的难度不会因为计算能力的提升而降低。

> 关于加密方面的知识，推荐看一看这篇文章：[http://www.williamlong.info/archives/3224.html](http://www.williamlong.info/archives/3224.html)

<a name="basic-usage"></a>
## 基本用法

通过调用 `Hash` facade 中的 `make` 方法即可对密码进行加密：

	<?php

	namespace App\Http\Controllers;

	use Hash;
	use App\User;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * Update the password for the user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function updatePassword(Request $request, $id)
		{
			$user = User::findOrFail($id);

			// Validate the new password length...

			$user->fill([
				'password' => Hash::make($request->newPassword)
			])->save();
		}
	}

另外，你还可以直接使用全局作用域中的  `bcrypt` 辅助函数：

	bcrypt('plain-text');

#### 根据哈希值校验密码

`check` 方法帮你检查一个纯文本字符串是否是一个哈希值。然而，如果你正在使用 [Laravel 中自带](/docs/{{version}}/authentication) 的 `AuthController`，你就不需要直接调用这个函数了，因为 `AuthController` 已经自动帮你调用这个方法了：

	if (Hash::check('plain-text', $hashedPassword)) {
		// The passwords match...
	}

#### 检查密码是否需要重新加密

`needsRehash` 函数帮助你检查自从密码被加密之后，加密系数（work factor）是否被改变了：

	if (Hash::needsRehash($hashed)) {
		$hashed = Hash::make('plain-text');
	}
