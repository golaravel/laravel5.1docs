# 加密

- [设置](#configuration)
- [基本用法](#basic-usage)

<a name="configuration"></a>
## 设置

在使用 Laravel 的加密功能前，你需要先为 `config/app.php` 配置文件中的 `key` 参数设置一个值，这个值是一个包含 32 个随机字符的字符串。如果这个值没有正确设置，所有由 Laravel 加密的数据都是不安全的。

<a name="basic-usage"></a>
## 基本用法

#### 加密

通过 `Crypt` [facade](/docs/{{version}}/facades) 可以加密一段数据。所有加密采用的都是 OpenSSL 和 `AES-256-CBC` cipher。并且，所有加密过的数据都会被赋予一个“信息验证码”（MAC），以防被加密后所得到的字符串被篡改。

如下例，我们使用 `encrypt` 方法将加密过的数据存入 [Eloquent model](/docs/{{version}}/eloquent)：

	<?php

	namespace App\Http\Controllers;

	use Crypt;
	use App\User;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * 为用户保存一段私密信息
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function storeSecret(Request $request, $id)
		{
			$user = User::findOrFail($id);

			$user->fill([
				'secret' => Crypt::encrypt($request->secret)
			])->save();
		}
	}

#### 解密

当然，你可以使用 `Crypt` 中的 `decrypt` 方法进行解密。如果解密失败，例如 MAC 被篡改了，将会抛出 `Illuminate\Contracts\Encryption\DecryptException` 异常：

	use Illuminate\Contracts\Encryption\DecryptException;

	try {
		$decrypted = Crypt::decrypt($encryptedValue);
	} catch (DecryptException $e) {
		//
	}
