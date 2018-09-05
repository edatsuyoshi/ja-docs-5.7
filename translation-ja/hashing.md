# ハッシュ

- [イントロダクション](#introduction)
- [Configuration](#configuration)
- [基本的な使用法](#basic-usage)

<a name="introduction"></a>
## イントロダクション

LaravelのHash[ファサード](/docs/{{version}}/facades)は保存するユーザーパスワードに対し、安全なBcryptとArgon2ハッシュを提供します。Laravelアプリケーションに組み込まれている、`LoginController`と`RegisterController`を使用していれば、登録と認証で自動的にBcrypt使用します。

> {tip} Bcryptは「ストレッチ回数」が調整できるのでパスワードのハッシュには良い選択肢です。つまりハードウェアのパワーを上げればハッシュの生成時間を早くすることができます。

<a name="configuration"></a>
## 設定

The default hashing driver for your application is configured in the `config/hashing.php` configuration file. There are currently three supported drivers: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) and [Argon2](https://en.wikipedia.org/wiki/Argon2) (Argon2i and Argon2id variants).

> {note} The Argon2i driver requires PHP 7.2.0 or greater and the Argon2id driver requires PHP 7.3.0 or greater.

<a name="basic-usage"></a>
## 基本的な使用法

`Hash`ファサードの`make`メソッドを呼び出し、パスワードをハッシュできます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * ユーザーパスワードを更新
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // 新しいパスワードの長さのバリデーション…

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### BcryptのWork Factorの調整

Bcryptアルゴリズムを使用する場合、`make`メソッドで`rounds`オプションを使用することにより、アルゴリズムのwork factorを管理できます。しかし、ほとんどのアプリケーションではデフォルト値で十分でしょう。

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### Argon2のWork Factorの調整

Argon2アルゴリズムを使用する場合、`memory`と`time`、`threads`オプションを指定することにより、アルゴリズムのwork factorを管理できます。しかし、ほとんどのアプリケーションではデフォルト値で十分でしょう。

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} これらのオプションの詳細情報は、[PHP公式ドキュメント](http://php.net/manual/ja/function.password-hash.php)をご覧ください。

#### パスワードとハッシュ値の比較

`check`メソッドにより指定した平文文字列と指定されたハッシュ値を比較確認できます。しかし[Laravelに含まれている](/docs/{{version}}/authentication)`LoginController`を使っている場合は、これを直接使用することはないでしょう。このコントローラがこのメソッドを自動的に呼び出します。

    if (Hash::check('plain-text', $hashedPassword)) {
        // パスワード一致
    }

#### パスワードの再ハッシュが必要か確認

パスワードがハシュされてからハッシャーのストレッチ回数が変更されているかを調べるには、`needsRehash`メソッドを使います。

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
