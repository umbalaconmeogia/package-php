# Giải thích cách tạo PHP package.

## Overview

Tài liệu này giải thích về tác dụng của chương trình quản lý PHP package *composer*, ý nghĩa của file composer.json, và cách để đóng gói một package của mình.

## Lệnh require của PHP

Lệnh [require](https://www.php.net/manual/en/function.require.php) (cũng như lệnh [include](https://www.php.net/manual/en/function.include.php)) của PHP cho phép ta chèn các phần code trong một file PHP khác vào đoạn code ta đang viết.

Như vậy thay vì ta phải viết một file PHP thật dài, thì có thể chia các dòng code vào các file khác nhau (để dễ quản lý) và dùng lệnh *require* để chèn các đoạn code PHP trong file đó vào chỗ ta muốn.

Giả sử ta có một file *lib/MyLibrary.php* có dòng code của function helloWorld() như sau

```php
<?php
function helloWorld()
{
    echo "Hello World\n";
}
```

Giờ ta có file *sampleRequire.php* trong đó ta chèn code của file *lib/MyLibrary.php* vào như sau

```php
<?php
require "lib/MyLibrary.php";

helloWorld();
```

Khi chạy chương trình *sampleRequire.php*, ta sẽ được kết quả của việc gọi function helloWorld().
```shell
php sampleRequire.php
```
cho kết quả
```shell
Hello World
```

*Giải thích*: Require file *lib/MyLibrary.php* trong *sampleRequire.php* ở trên cũng tương đương với viết code của file *sampleRequire.php* như sau (không dùng đến lệnh require).
```php
<?php
function helloWorld()
{
    echo "Hello World\n";
}

helloWorld();
```

![Require](docs/images/require.png)

## Trình quản lý thư viện composer

Bằng việc chia các dòng code PHP vào các file như ở trên, ta sẽ có thể sử dụng những dòng code này khi cần đến, bằng cách *require* các file đó khi cần thiết.
Điều này cho phép ta dễ xây dựng và sử dụng được những chương trình/thư viện PHP lớn, phức tạp.

Tập hợp của những file PHP này được gom lại thành các package để dễ quản lý và sử dụng.

Nhiều package được công khai (public) trên mạng internet, để mọi người có thể lấy về sử dụng.

Ta có thể download các package (là một tập hợp các file PHP) về, và dùng lệnh require để chèn nó vào trong chương trình PHP của ta, và có thể sử dụng tính năng được người khác viết ra trong các package đó.

Để việc sử dụng được dễ dàng, thì các package cần được xây dựng theo những *quy chuẩn* nhất định, để chúng có tính thống nhất về cấu trúc.

Ngoài ra, bản thân các package A nào đó cũng lại sử dụng đến các package B khác nữa, ta gọi là package A phụ thuộc package B. Đến lượt package B lại có thể phụ thuộc vào các package khác nữa.

Để sử dụng package A, thì ngoài việc download code của package A về, ta còn cần phải download cả code của package B, và code của cả các package mà B phụ thuộc vào nữa...

![Dependency](docs/images/dependency.png)

Chúng có thể là một mối quan hệ vô cùng rối rắm, mà nếu làm việc này một cách thủ công thì sẽ rất phiền. Nên người ta dùng một chương trình để giúp tự động hóa việc đó, gọi là chương trình quản lý phụ thuộc (dependency management).

[Composer](https://getcomposer.org/) là một chương trình quản lý phụ thuộc như vậy. Khi ta cần một package A, bên cạnh việc download A về, nó sẽ tìm những package mà A phụ thuộc và download chúng.

Để làm được điều đó, Composer dựa vào file *composer.json* để biết ta cần các package nào, và package đó có thể được download về từ đâu. Nó sẽ lần theo các thông tin đó để tự động lôi các package cần thiết về cho ta.

Các package đó sẽ được để trong một thư mục tên là *vendor*, và nó còn tạo ra một file *vendor/autoload.php* trong đó require toàn bộ các package đã được download về đó.
Và chương trình của ta chỉ cần require cái file *vendor/autoload.php* đó là có thể sử dụng những package ta đã download về.

### Khai báo phụ thuộc trong composer.json

Ta tạo một file *composer.json* như sau
```json
{
    "minimum-stability": "dev",
    "require": {
        "umbalaconmeogia/package-php-dev": "*"
    },
    "repositories": [
        {
            "type": "vcs",
            "url":  "git@github.com:umbalaconmeogia/package-php-dev.git"
        }
    ]
}
```

Chỉ dẫn *require* trong file *composer.json* này báo rằng ta cần sử dụng package tên là *umbalaconmeogia/package-php-dev*, và chỉ dẫn *repositories* cho biết ta có thể tìm nó ở trên github, tại địa chỉ https://github.com/umbalaconmeogia/package-php-dev

Khi chạy lệnh
```shell
composer install
```
nó sẽ tạo ra thư mục *vendor*, download code của các package cần thiết, và tạo ra file *vendor/autoload.php* để require code của package đã download về.
Nó cũng tạo ra file *composer.lock* nữa, nhưng ta sẽ không đề cập tới ở đây.

Giờ ta tạo một chương trình PHP *samplePackagePhpDev.php* để sử dụng package *umbalaconmeogia/package-php-dev* đã download về ở trên như sau
```php
<?php
require "vendor/autoload.php";

\umbalaconmeogia\packagephpdev\MyLibrary2::goodBye();
```

Khi chạy chương trình *samplePackagePhpDev.php*, ta sẽ được kết quả của việc gọi function [MyLibrary2::goodBye()](https://github.com/umbalaconmeogia/package-php-dev/blob/main/src/MyLibrary2.php#L6).
```shell
php samplePackagePhpDev.php
```
cho kết quả
```shell
Good bye
```

## Cách tạo package

Nói ngắn gọn, thì để tạo ra một package PHP, thì ta tạo cho nó một file composer.json.

Package *umbalaconmeogia/package-php-dev* có một file [composer.json](https://github.com/umbalaconmeogia/package-php-dev/blob/main/composer.json) của nó.

File này có nội dung như sau
```json
{
    "name": "umbalaconmeogia/package-php-dev",
    "autoload": {
        "psr-4": {"umbalaconmeogia\\packagephpdev\\": "src/"}
    }
}
```
Nó định nghĩa tên của package này là *umbalaconmeogia/package-php-dev*,
và các file đặt trong thư mục *src* của nó sẽ có *namespace* là *umbalaconmeogia\packagephpdev*.
Do đó, file MyLibrary2.php đặt trong thư mục *src* sẽ có namespace là "umbalaconmeogia\packagephpdev\MyLibrary2".

Bằng việc tuân theo các quy chuẩn của PHP (PSR = PHP Standards Recommendations), trong việc định nghĩa các thông tin trong file composer.json, ta có thể tạo ra các thư viện và composer sẽ giúp ta sử dụng được chúng một cách dễ dàng.

Đến đây, ta đã biết được file *composer.json* định nghĩa các thông tin sau:
1. Tên và namespace của một package (để các chương trình khác có thể sử dụng nó, thông qua các đoạn chương trình PHP của composer - có một thư mục *vendor/composer* chứa các chương trình PHP để giúp kết nối các package được download về thông qua Composer, và file *vendor/autoload.php* mà ta đã biết, thực ra là nó lại require các file PHP nằm trong thư mục *vendor/composer* này).
2. Các thư viện mà ta cần sử dụng cũng như nơi để download nó về (thông qua các chỉ dẫn *require* và *repositories*).

Chú ý rằng ý thứ 2 (các thư viện mà ta cần sử dụng cũng như nơi để download nó về) có thể diễn giải là các thư viện mà chương trình của ta muốn sử dụng (trong phần "Khai báo phụ thuộc trong composer.json" ở trên), và cũng có thể hiểu là các thư viện mà bản thân cái thư viện ta đang phát triển (umbalaconmeogia/package-php-dev) muốn sử dụng code của nó. Vẫn là cùng một ý nghĩa.

![package-php](docs/images/package-php.png)

Đến đây, ta đã có được kiến thức tối thiểu để có thể đóng gói code PHP của ta thành một thư viện (bằng việc tạo ra một file composer.json cho nó), và sử dụng thư viện đó trong các chương trình/thư viện khác (thông qua việc require nó trong file composer.json của chương trình đó, rồi dùng lệnh composer để download nó về).

## Quản lý version của package
TBD

## Quy trình phát triển một package
TBD

```json
{
    "minimum-stability": "dev",
    "require": {
        "umbalaconmeogia/package-php-dev": "*"
    },
    "repositories": [
		{
            "type": "path",
            "url": "../package-php-dev",
            "symlink": true
        }
    ]
}
```

## Hiểu sâu hơn về composer

### Các lệnh của composer
TBD

### File composer.json
TBD
