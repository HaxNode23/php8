---
title:   UTF-8 사용하기
isChild: true
anchor:  php_and_utf8
---

## UTF-8 사용하기 {#php_and_utf8_title}

_이 섹션은 [Alex Cabal](https://alexcabal.com/)이 작성한 [PHP Best Practices](https://phpbestpractices.org/#utf-8)에
포함되어 있는 내용을 기반으로 하여 작성되었습니다._

### 한 줄로 끝나는 팁은 없습니다. 주의깊고 일관성있게 작업해야 합니다.

PHP는 현재 유니코드 지원을 언어 내부적으로 해주고 있지 않습니다. UTF-8 문자열을 문제없이 처리할 수 있는 방법은 있지만,
HTML 코드, SQL 쿼리, PHP 코드 등 웹 어플리케이션의 모든 레벨을 다뤄야 하는 수준의 이야기입니다. 여기서는 활용할 수
있는 간략한 설명을 제공하는데 초점을 두겠습니다.

### PHP 코드 수준에서의 UTF-8

문자열을 변수에 할당하거나 두 문자열을 이어붙이는 등의 기본적인 문자열 연산은 UTF-8 문자열이라고 해도 특별할 것이
없습니다. 하지만 `strpos()`나 `strlen()`과 같은 대부분의 문자열 함수들은 신경을 써야 합니다. 이런 함수들은
`mb_strpos()`나 `mb_strlen()` 같이 원래 이름 앞에 `mb_`가 붙은 함수들이 별도로 존재합니다. 그런 함수들을 멀티바이트
문자열 함수라고 하고, [멀티바이트 문자열 익스텐션][Multibyte String Extension]에 의해서 제공됩니다. 멀티바이트 문자열
함수들은 유니코드 문자열을 다루기 위해서 특별히 제공되는 함수들입니다.

유니코드 문자열을 다룰 때에는 항상 `mb_`로 시작하는 함수들을 사용해야 합니다. 예를 들어 `substr()` 함수를 UTF-8
문자열에 대해서 사용해보면, 결과물에는 이상하게 깨진 글자가 나온다는 사실을 알게 될 좋은 기회가 될 것입니다. UTF-8
문자열을 다룰 때에는 `mb_substr()` 함수를 사용해야 합니다.

유니코드 문자열을 다룰 때에는 항상 `mb_`로 시작하는 함수를 사용해야 한다는 걸 기억하는 게 어려운 일입니다. 한 군데라도
까먹고 일반 문자열 함수를 사용하면 깨진 유니코드 문자열을 보게 될 수가 있습니다.

모든 문자열 함수가 그에 해당되는 멀티바이트 버전의 `mb_` 로 시작되는 함수를 가지고 있는 것은 아닙니다. 여러분이
사용하려고 했던 문자열 함수가 그런 경우라면, 운이 없었다고 할 수 밖에요.

모든 PHP 스크립트 파일(또는 모든 PHP 스크립트에 include 되는 공용 스크립트)의 가장 윗부분에 `mb_internal_encoding()`
함수를 사용해서 문자열 인코딩을 지정하고, 그 바로 다음에 `mb_http_output()` 함수로 브라우저에 출력될 문자열 인코딩도
지정해야 합니다. 이런 식으로 모든 스크립트에 문자열의 인코딩을 명시적으로 지정하는 것이, 나중에 고생할 일을 아주
많이 줄여줍니다.

추가로, 많은 문자열 함수들이 함수 파라미터의 문자열 인코딩을 지정할 수 있는 옵셔널 파라미터를 제공한다는 사실을
이야기해야겠습니다. 그런 경우마다 항상 UTF-8 문자열 인코딩을 명시적으로 지정해야 합니다. 예를 들어, `htmlentities()`
함수의 경우가 그렇습니다. UTF-8 문자열을 전달하는 경우 항상 UTF-8로 지정하십시오. 또한 PHP 5.4.0 부터는
`htmlentities()` 함수와 `htmlspecialchars()` 함수의 경우에 UTF-8이 기본 인코딩으로 변경되었다는 것을 알아두는게
좋겠습니다.

마지막으로, 만약 분산 배포되는 환경에서 동작하는 어플리케이션을 만들어야 하는데, `mbstring` 익스텐션이 활성화 되어
있는지 확신할 수 없는 상황이라면, [patchwork/utf8]이라는 Composer 패키지를 사용해보는 것도 좋겠습니다. 그 패키지를
사용하면, `mbstring` 익스텐션이 활성화되어 있으면 그쪽 함수들을 사용하고 활성화되어 있지 않으면 일반 문자열 함수가
대신 호출되는 식으로 동작하게 만들어줍니다.

[Multibyte String Extension]: https://secure.php.net/book.mbstring
[patchwork/utf8]: https://packagist.org/packages/patchwork/utf8

### 데이터베이스 수준에서의 UTF-8

MySQL을 사용하는 PHP 스크립트가 있다면, 앞에서 이야기한 내용을 충실히 따르더라도 데이터베이스에 저장되는 문자열은
UTF-8이 아닌 다른 인코딩으로 저장될 가능성이 있습니다. 

PHP에서 MySQL로 전달되는 문자열이 UTF-8로 확실히 전달되게 하려면, 사용하는 데이터베이스와 테이블이 모두 `utf8mb4`
캐릭터 셋과 콜레이션(collation)을 사용하게 해야합니다. 그리고 PDO 연결문자열에도 `utf8mb4` 캐릭터 셋을 사용한다고
명시해야 합니다. 다음의 예제를 보세요. _정말 중요합니다_.

UTF-8 문자열을 제대로 사용하려면 반드시 `utf8mb4` 캐릭터 셋을 사용해야 합니다. `utf8` 캐릭터 셋이 아닙니다!
(놀라워라...) 그 이유는 '더 읽어보기'를 참고하세요.

### 브라우저 수준에서의 UTF-8

PHP 스크립트가 UTF-8 문자열을 브라우저에 제대로 전송하게 하려면 `mb_http_output()` 함수를 사용하세요.

그리고 HTTP 응답이 UTF-8로 되어 있다는 것을 브라우저도 알 수 있게 해줘야 되겠지요. 오늘날엔 HTTP 응답 헤더에 이렇게 charset을 설정해주는 것이 보편적입니다. 

{% highlight php %}
<?php
header('Content-Type: text/html; charset=UTF-8')
{% endhighlight %}

HTML 응답 내용의 `<head>` 태그에
[charset `<meta>` 태그](http://htmlpurifier.org/docs/enduser-utf8.html)를 넣는 것이 전통적인 방식이었습니다. 이렇게 하는
데에 잘못된 점은 하나도 없지만, 성능을 좀 더 올릴 수 있는 방법도 있습니다. HTTP 응답의 `Content-Type` 헤더에 charset
설정을 하면 [훨씬 빠르게 동작](https://developers.google.com/speed/docs/best-practices/rendering#SpecifyCharsetEarly)
한다고 합니다.

(역자 주: 현재 해당 링크는 charset 관련된 내용을 담고 있지 않네요. 오래된 글이긴 하지만
[이 링크](https://code.google.com/p/page-speed/wiki/SpecifyCharsetEarly)를 참고하면 도움이 될 것 같습니다.)

The browser will then need to be told by the HTTP response that this page should be considered as UTF-8. The historic
approach to doing that was to include the [charset `<meta>` tag](http://htmlpurifier.org/docs/enduser-utf8.html) in
your page's `<head>` tag. This approach is perfectly valid, but setting the charset in the `Content-Type` header is
actually [much faster](https://developers.google.com/speed/docs/best-practices/rendering#SpecifyCharsetEarly).

{% highlight php %}
<?php
// Tell PHP that we're using UTF-8 strings until the end of the script
mb_internal_encoding('UTF-8');
 
// Tell PHP that we'll be outputting UTF-8 to the browser
mb_http_output('UTF-8');
 
// Our UTF-8 test string
$string = 'Êl síla erin lû e-govaned vîn.';
 
// Transform the string in some way with a multibyte function
// Note how we cut the string at a non-Ascii character for demonstration purposes
$string = mb_substr($string, 0, 15);
 
// Connect to a database to store the transformed string
// See the PDO example in this document for more information
// Note the `charset=utf8mb4` in the Data Source Name (DSN)
$link = new PDO(
    'mysql:host=your-hostname;dbname=your-db;charset=utf8mb4',
    'your-username',
    'your-password',
    array(
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_PERSISTENT => false
    )
);
 
// Store our transformed string as UTF-8 in our database
// Your DB and tables are in the utf8mb4 character set and collation, right?
$handle = $link->prepare('insert into ElvishSentences (Id, Body) values (?, ?)');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->bindValue(2, $string);
$handle->execute();
 
// Retrieve the string we just stored to prove it was stored correctly
$handle = $link->prepare('select * from ElvishSentences where Id = ?');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->execute();
 
// Store the result into an object that we'll output later in our HTML
$result = $handle->fetchAll(\PDO::FETCH_OBJ);

header('Content-Type: text/html; charset=UTF-8');
?><!doctype html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>UTF-8 test page</title>
    </head>
    <body>
        <?php
        foreach($result as $row){
            print($row->Body);  // This should correctly output our transformed UTF-8 string to the browser
        }
        ?>
    </body>
</html>
{% endhighlight %}

### Further reading

* [PHP Manual: String Operations](http://php.net/language.operators.string)
* [PHP Manual: String Functions](http://php.net/ref.strings)
    * [`strpos()`](http://php.net/function.strpos)
    * [`strlen()`](http://php.net/function.strlen)
    * [`substr()`](http://php.net/function.substr)
* [PHP Manual: Multibyte String Functions](http://php.net/ref.mbstring)
    * [`mb_strpos()`](http://php.net/function.mb-strpos)
    * [`mb_strlen()`](http://php.net/function.mb-strlen)
    * [`mb_substr()`](http://php.net/function.mb-substr)
    * [`mb_internal_encoding()`](http://php.net/function.mb-internal-encoding)
    * [`mb_http_output()`](http://php.net/function.mb-http-output)
    * [`htmlentities()`](http://php.net/function.htmlentities)
    * [`htmlspecialchars()`](http://php.net/function.htmlspecialchars)
* [PHP UTF-8 Cheatsheet](http://blog.loftdigital.com/blog/php-utf-8-cheatsheet)
* [Handling UTF-8 with PHP](http://www.phpwact.org/php/i18n/utf-8)
* [Stack Overflow: What factors make PHP Unicode-incompatible?](http://stackoverflow.com/questions/571694/what-factors-make-php-unicode-incompatible)
* [Stack Overflow: Best practices in PHP and MySQL with international strings](http://stackoverflow.com/questions/140728/best-practices-in-php-and-mysql-with-international-strings)
* [How to support full Unicode in MySQL databases](http://mathiasbynens.be/notes/mysql-utf8mb4)
* [Bringing Unicode to PHP with Portable UTF-8](http://www.sitepoint.com/bringing-unicode-to-php-with-portable-utf8/)
* [Stack Overflow: DOMDocument loadHTML does not encode UTF-8 correctly](http://stackoverflow.com/questions/8218230/php-domdocument-loadhtml-not-encoding-utf-8-correctly)
