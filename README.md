PHP-TO-C-Ext is a tool to allow developer to write codes in PHP and convert it directly to php extensions.

With PHP-TO-C-Ext tool, the developer can choose a php file that is not changed quite often and convert it to native php extension, thus speeding up the server response time and lower the resource consumption for each request.

PHP-TO-C-EXT is built on top of these great things:

+ Zephir (http://zephir-lang.com/)
+ PHP Parser (https://github.com/nikic/PHP-Parser)
+ PHP to Zephir (https://github.com/fezfez/php-to-zephir)

##Installation

1. Install composer
2. git clone git@github.com:jimthunderbird/php-to-c-extension.git
3. cd php-to-c-extension
4. composer.phar install

##Usage:

```sh
$ php [path/to/php-to-c-extension]/build_extensions.php [php file to convert to c extension]
```

or

```sh
$ php [path/to/php-to-c-extension]/build_extensions.php [directory containing php files to convert to c extension]
```

##Examples:

+ [A simple dummy extension](#example-01)
+ [One namespace and multiple classes in one file](#example-02)
+ [Organize multiple files in one directory](#example-03)
+ [Using for loop](#example-04)
+ [Using sub-namespaces] (#example-05)
+ [Using interface] (#example-06)
+ [Using trait] (#example-07)
+ [Calling method in the base class with parent::](#example-08)
+ [Using the self keyword](#example-09)
+ [Using ternary operator](#example-10)
+ [Late static binding](#example-11)
+ [Performance Benchmark: Bubble Sort](#example-12)

###Example 01

Let's create a file named Dummy.php, it looks like this:

```php
<?php
namespace Dummy;
class Hello 
{
  public function say()
  {
    echo "hello";
  }
}
```

#### we can then execute:

```sh
php [path/to/php-to-c-extension]/build_extensions.php Dummy.php
```
    
After a while we should get dummy.so installed, then if we add the following line to php.ini 

```
extension=dummy.so
```
    
#### we will now have the class Dummy\Hello available for the user code.
#### If we write a file named test.php like the following:
```php
<?php
$o = new Dummy\Hello();
$o->say();
```
#### and if we run it with php -c [path/to/php.ini]/php.ini test.php, we should get "hello" printed.
#### You might have already noticed, the class Hello has the namespace Dummy and the extension name is dummy.so. 
#### In fact, in order to build a php extension with this tool, all classes must have a CamelCase namespace, and the extension name is the lowercase form of the namespace. 


###Example 02 
####Sometimes, for convenience, we might want to write a single file with one namespace and multiple classes, and we can do just that.
####Let's create a file named Dummy.php an it looks like the following:
```php 
<?php 
namespace Dummy; 
class Hello 
{
  public function say()
  {
    echo "hello\n";
  }
}

class Greeting 
{
  public function greet()
  {
    echo "greetings\n";
  }
}
``` 
####We can then execute 
####
    php [path/to/php-to-c-extension]/build_extensions.php Dummy.php 
#### Once we get the dummy.so built and added to the php.ini, we will have both Dummy\Hello and Dummy\Greeting classes available for the user code.

###Example 03 
####If we need to write more complicated php extensions, we usually need to maintain serveral source files, with PHP-TO-C-EXT tool, we can compile all files in a target directory.
####Let's create a directory src/Dummy, and inside we will 2 files, Hello.php and Greeting.php 
####Here is what src/Hello.php looks like:
```php 
<?php
namespace Dummy; 
class Hello 
{
  public function say()
  {
    echo "hello\n";
  }
}
```
####And here is what src/Greeting.php looks like:
```php 
<?php 
namespace Dummy;
class Greeting 
{
  public function greet()
  {
    echo "greetings\n";
  }
}
```
####Then if we execute 
####
    php [path/to/php-to-c-extension]/build_extensions.php src/Dummy 
####We will then have dummy.so built, and now both Dummy\Hello and Dummy\Greeting classes will be available for the user code.
####Notice that both Hello.php and Dummy.php must have namespace Dummy defined in the beginning.

###Example 04 
####For loop is a common control structure in php, here we will be creating a Dummy\SumCalculator class and built that into the dummy.so extension.
####Let's create a file src/Dummy/SumCalculator.php, and it looks like this:
```php 
<?php 
namespace Dummy;

class SumCalculator 
{
  public function getSum($start, $end)
  {
    $sum = 0;
    for ($i = $start; $i <= $end; $i++) {
      $sum += $i;
    }
    return $sum;
  }
}
``` 
####Then if we execute 
####
    php [path/to/php-to-c-extension]/build_extensions.php src/Dummy 
####We will then have dummy.so built, and now both Dummy\SumCalculator will be available for the user code.
####We can do something like this in our user code:
```php 
<?php
$calculator = new Dummy\SumCalculator();
$calculator->getSum(1,10);
``` 

###Example 05 
####We can use sub namespaces to better manage the cod in the extension.
####When using sub namespaces, we need to make sure the first part of the sub namespace match is name of our extension.
####In this example, let's create a dummy extension with the following files.
####1. src/Dummy/Vehicle.php 
####2. src/Dummy/Vehicle/Car.php 
####src/Dummy/Vehicle.php looks like this:
```php 
<?php 
namespace Dummy;

class Vehicle 
{
  public function say()
  {
    echo "I am a vehicle";
  }
}
```
####src/Dummy/Vehicle/Car.php looks like this:
```php
<?php 
namespace Dummy\Vehicle;

class Car extends \Dummy\Vehicle
{
}
``` 
####Then if we execute 
####
    php [path/to/php-to-c-extension]/build_extensions.php src/Dummy 
####We will then have dummy.so built, and then if we do the following in our user code 
```php 
$car = new Dummy\Vehicle\Car();
$car->say();
```
####We will have "I am a vehicle" printed.

###Example 06
####We can use interface just like what we do in normal php code.
####Let's created the following files:
####1. src/Dummy/MovableInterface.php 
####2. src/Dummy/Vehicle.php 
####src/Dummy/MovableInterface.php looks like this:
```php 
<?php 
namespace Dummy;
interface MovableInterface
{
  public function move();
}
```
####src/Dummy/Vehicle.php looks like this:
```php
<?php 
namespace Dummy;

class Vehicle implements MovableInterface
{
  public function move()
  {
    echo "I am moving";
  }
}
``` 
####Then if we execute 
####
    php [path/to/php-to-c-extension]/build_extensions.php src/Dummy 
####We will then have dummy.so built, and then if we do the following in our user code 
```php 
$car = new Dummy\Vehicle();
$car->move();
```
####We will have "I am moving" printed.

###Example 07 
####Trait is a new feature introduced in php 5.4 to allow grouping of functionalities and reUsed by individual classes.
####In the example below, we are going to demonstrate using trait to build a dummy php extension 
####We are going to create a file src/dummy.php, and it looks like this:
```php 
<?php 
namespace Dummy\Traits;

trait MovableTrait 
{
  public function move()
  {
    echo "I am moving.";
  }
}

trait PlayableTrait 
{
  public function play()
  {
    echo "I am playing.";
  }
}

namespace Dummy;

class Vehicle 
{
  use \Dummy\Traits\MovableTrait;
  use \Dummy\Traits\PlayableTrait;
}
```
#### Then if we execute
####
    php [path/to/php-to-c-extension]/build_extensions.php src/dummy.php
####Once the extension dummy.so is built, we will have method move() and play() available for class Vehicle.
####And if we do the following in our user code 
```php 
<?php 
$vehicle = new Dummy\Vehicle();
$vehicle->play();
$vehicle->move();
```
####We will see the "I am playing.I am moving." printed 

###Example 08 
####When we need to call certain methods in the base class, we will need to use the parent keyword, below is an example:
####Let's create a file named dummy.php and it looks like this:
```php 
<?php 
namespace Dummy;

trait StoppableTrait 
{
  public function stop()
  {
    echo "I am stopping now.";
  }
}

class Vehicle 
{
  use StoppableTrait;

  public function __construct()
  {
    echo "I am a new vehicle.";
  }
}

class Car extends Vehicle 
{
  public function __construct() 
  {
    parent::__construct();
    echo "I am also a new car.";
  }

  public function stop()
  {
    parent::stop();
    echo "don't worry i am a car.";
  }
}
```
####In the code above, we know that the base class Vehicle has a method named stop() defined by the trait.
####And now if we do 
```php 
$car = new Dummy\Car();
$car->stop();
```
####We should see the following printed 
####"I am a new vehicle.I am also a new car.I am stopping now.don't worry i am a car."

###Example 09
####We can use the self keyword just like normal php to build a php extension, below is an example:
####Let's create a file named dummy.php and it looks like this:
```php 
<?php 
namespace Dummy;

trait StoppableTrait 
{
  public function stop()
  {
    echo "I am stopping now.";
  }
}

class Vehicle 
{
  private static $singleton;

  use StoppableTrait;

  public function __construct()
  {
    echo "I am a new vehicle.";
  }

  public static function getInstance()
  {
    if (!isset(self::$singleton)) {
      self::$singleton = new self();
    }  
    return self::$singleton;
  }
}

class Car extends Vehicle 
{
  public function __construct() 
  {
    parent::__construct();
    echo "I am also a new car.";
  }

  public function stop()
  {
    parent::stop();
    echo "don't worry i am a car.";
  }
}
```
####Notice the getInstance() method in class Vehicle, we use the self keyword to create a singleton.
####And now once we have dummy.so built, if we do the following in the user code:
```php 
<?php 
$car1 = Dummy\Car::getInstance();
$car2 = Dummy\Car::getInstance();

if ($car1 === $car2) {
  print "Two cars are identical\n";
} 

print $car1->stop()."\n";
```
####We should see the following printed 
####"I am a new vehicle.Two cars are identical
####I am stopping now."

###Example 10 
####We can use ternary operator as a shortcut to write conditional statemements and variable assignmens 
####Let's create a file named dummy.php and it looks like this:
```php 
<?php
namespace Dummy;

class Number 
{
  private $number;

  public function __construct($number)
  {
    $this->number = $number;
  }

  public function isPositive()
  {
    $result = ($this->number > 0)?true:false;
    return $result;
  }
}
``` 
####Then once we dummy.so built, in the user code if we do the following:
```php 
<?php 
$number = new Dummy\Number(10);
if ($number->isPositive() === TRUE) {
  echo "This is a positive number.";
}
```
####We then should see the following printed on the screen.
####"This is a positive number." 

###Example 11 
####Late static binding is introduced in PHP 5.3 and used to reference the called class in a context of static inheritance.
####Below is an example of late static binding 
####Let's create a file named src/dummy.php and it looks like this:
```php  
namespace Dummy;
class Model 
{ 
  protected static $name = "model";
  public static function find() 
  { 
    echo static::$name; 
  } 
} 

class Product extends Model 
{ 
  protected static $name = 'Product'; 
} 
``` 
####Then once we have dummy.so built and if we do the following:
```php 
Dummy\Product::find();
```
####We should have "Product" printed on the screen.

###Example 12 
####Below let's do a simple benchmark to see how fast the php extension will be 
####We will be using bubble sort as an example to demonstrate the time difference. Here is our code for src/dummy.php:
```php 
<?php 
namespace Dummy;

class Sorter
{
  public function bubbleSort($arr)
  {
    $size = count($arr);
    for ($i=0; $i<$size; $i++) {
      for ($j=0; $j<$size-1-$i; $j++) {
        if ($arr[$j+1] < $arr[$j]) {
          $tmp = $arr[$j];
          $arr[$j] = $arr[$j+1];
          $arr[$j+1] = $tmp;
        }
      }
    }

    return $arr;
  }
}
```
####As you can see, this is a very typical bubble sort implementation.
####We will then do: 
```sh
$ php [path/to/php-to-c-extension]/build_extensions.php src/dummy.php
```
####Then we will have dummy.so built.
####Now we will be writing our userland testing code test.php
```php 
<?php 
if (!class_exists("Dummy\Sorter")) {
  require_once "src/dummy.php";
}

function microtime_float()
{
  list($usec, $sec) = explode(" ", microtime());
  return ((float)$usec + (float)$sec);
}

$arr = array();
for ($i = 10000; $i >= 1; $i--) {
  $arr[]  = $i;
}

$time_start = microtime_float();

$st = new Dummy\Sorter();
$arr = $st->bubbleSort($arr);

$time_end = microtime_float();
$time = $time_end - $time_start;

print "Time spent on sorting: ".$time." seconds.\n";
```
####The code above is pretty straightforward, it first detect if we have the Dummy\Sorter class defined, if it is defined, that means
####the dummy.so extension is loaded, otherwise, we will just require the pure php version of Dummy\Sorter class. 
####We then generate an array of 10000 integers and ask Dummy\Sorter to bubble sort it.
####This is also the beauty of having the ability to write our extension in php itself, since we can seamlessly compare the performance.
####Now if we just do: 
```sh 
php test.php
```
####We will be just using the pure php version, in my intel core i3 laptop with 4 core cpu running fedora 21, it shows the following:
####Time spent on sorting: 16.802139997482 seconds.
####Now let's test the php extension see how it performs. We first create php.ini and then inside we have:
```sh 
extension=dummy.so
```
####Then if we do php -c php.ini test.php, we will be using the dummy.so to do the bubble sort for us, in my laptop it shows the following:
####Time spent on sorting: 3.9628620147705 seconds.
####As you can see, the php extension dummy.so that we built is about 3 times faster than the pure php version. And we are seamlessly using the same Dummy\Sorter class!
