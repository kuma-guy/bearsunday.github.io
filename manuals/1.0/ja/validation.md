---
layout: docs-ja
title: バリデーション
category: Manual
permalink: /manuals/1.0/ja/validation.html
---

# バリデーション

`@Valid`アノテーションを使用すると、メソッドの実行前にバリデーションメソッドが自動的に実行されるようになります。
エラーを検知すると例外が発生しますが、代わりに別のメソッドを呼ぶこともできます。

分離したバリデーションのコードは可読性に優れテストが容易です。バリデーションのライブラリは[Aura.Filter](https://github.com/auraphp/Aura.Filter)や[Respect\Validation](https://github.com/Respect/Validation)、あるいは[PHP標準のFilter](http://php.net/manual/ja/book.filter.php)を使います。

## インストール

composerインストール

{% highlight bash %}
composer require ray/validate-module
{% endhighlight %}

アプリケーションモジュール`src/Module/AppModule.php`で`ValidateModule`をインストールします。

{% highlight php %}
<?php
use Ray\Validation\ValidateModule;

class AppModule extends AbstractModule
{
    protected function configure()
    {
        // ...
        $this->install(new ValidateModule);
    }
}
{% endhighlight %}

## アノテーション

バリデーションのために`@Valid`、`@OnValidate`、`@OnFailure`の３つのアノテーションが用意されています。


まず、バリデーションを行いたいメソッドに`@Valid`とアノテートします。

{% highlight php %}
<?php
use Ray\Validation\Annotation\Valid;
// ...
    /**
     * @Valid
     */
    public function createUser($name)
    {
{% endhighlight %}

`@OnValidate`とアノテートしたメソッドでバリデーションを行います。引数は元のメソッドと同じにします。メソッド名は自由です。

{% highlight php %}
<?php
use Ray\Validation\Annotation\OnValidate;
// ...
    /**
     * @OnValidate
     */
    public function onValidate($name)
    {
        $validation = new Validation;
        if (! is_string($name)) {
            $validation->addError('name', 'name should be string');
        }

        return $validation;
    }
{% endhighlight %}

バリデーション失敗した要素には`要素名`と`エラーメッセージ`を指定してValidationオブジェクトに`addError()`し、最後にValidationオブジェクトを返します。

バリデーションが失敗すれば`Ray\Validation\Exception\InvalidArgumentException`例外が投げられますが、
`@OnFailure`メソッドが用意されていればそのメソッドの結果が返されます。

{% highlight php %}
<?php
use Ray\Validation\Annotation\OnFailure;
// ...
    /**
     * @OnFailure
     */
    public function onFailure(FailureInterface $failure)
    {
        // original parameters
        list($this->defaultName) = $failure->getInvocation()->getArguments();

        // errors
        foreach ($failure->getMessages() as $name => $messages) {
            foreach ($messages as $message) {
                echo "Input '{$name}': {$message}" . PHP_EOL;
            }
        }
    }
{% endhighlight %}
`@OnFailure`メソッドには`$failure`が渡され`($failure->getMessages()`でエラーメッセージや`$failure->getInvocation()`でオリジナルメソッド実行のオブジェクトが取得できます。

## 複数のバリデーション

１つのクラスに複数のバリデーションメソッドが必要なときは以下のようにバリデーションの名前を指定します。

{% highlight php %}
<?php
use Ray\Validation\Annotation\Valid;
use Ray\Validation\Annotation\OnValidate;
use Ray\Validation\Annotation\OnFailure;
// ...

    /**
     * @Valid("foo")
     */
    public function fooAction($name, $address, $zip)
    {

    /**
     * @OnValidate("foo")
     */
    public function onValidateFoo($name, $address, $zip)
    {

    /**
     * @OnFailure("foo")
     */
    public function onFailureFoo(FailureInterface $failure)
    {
{% endhighlight %}

## その他のバリデーション

複雑なバリデーションの時は別にバリデーションクラスをインジェクトして、`onValidate`メソッドから呼び出してバリデーションを行います。DIなのでコンテキストによってバリデーションを変えることもできます。
