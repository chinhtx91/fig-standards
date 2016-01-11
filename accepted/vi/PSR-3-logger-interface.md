Logger Interface
================

Tài liệu này mô tả một quy chuẩn cho các thư viện người dùng khai thác.

Mục tiêu chính là cho phép các thư viện có thể nhận được một `Psr\Log\LoggerInterface`
đối tượng và ghi lại nhật kí 1 cách đơn giản và phổ biến. Frameworks
và CMSs có tùy chỉnh cần PHẢI mở rộng giao diện người dùng cho mục đích riêng,
tuy nhiên, NÊN tương thích với tài liệu này. Điều này đảm bảo
các thư viện ứng dụng của người sử dụng bên thứ 3 có thể ghi lại các bản ghi tập trung.

Các ừ khóa "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", và "OPTIONAL" trong tài liệu này đã được giải thích trong 
[RFC 2119][].

từ `implementor` trong văn bản này  có nghĩa là 1 người nào đó đã thực hiện `LoggerInterface` trong 1 đăng nhập đã liên kết 
thư viện or framework.
Những người dùng của những lần đăng nhập được gọi là `user`.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

1. Đặc điểm kỹ thuật
-----------------

### 1.1 Khái niệm cơ bản.

- The `LoggerInterface` đã thể hiện 8 phương thức ghi lại nhật kí gồm 8 mức độ
  [RFC 5424][] levels (debug, info, notice, warning, error, critical, alert,
  emergency).

- Phương thức thứ 9, `log`, đã được chấp nhận 1 mức độ đăng nhập lập luận đầu tiên.
- Phương thức này với 1 mức độ đăng nhập cố định PHẢI có kết quả như 1 phương thức đặc biệt.\
- Phương thức này with không được quy định mức độ vì đặc tả PHẢI loại bỏ 1 `Psr\Log\InvalidArgumentException`
  nếu việc thực hiện không biết được thông tin mức độ. Những  người dùng KHÔNG NÊN sử dụng câp độ tùy chỉnh 
 nếu không biết chắc chắn những hỗ trợ hiện tại của nó

[RFC 5424]: http://tools.ietf.org/html/rfc5424

### 1.2 Lời nhắn

- Mỗi phương thức được chấp nhận cùng với 1 lời nhắn, hoặc 1 đối tượng
  `__toString()` method. Những người thực hiện PHẢI có xử lí đặc biệt để cho các đối tượng được thông qua.
Trong trường hợp không có, Phải đặt nó trong 1 chuỗi.

- Lời nhắn phải chứa (placeholders) which những người thực hiện PHẢI thay thế với giá trị trong mảng.

  (placeholders) những cái tên PHẢI tương ứng với những khóa trong mảng.

  (placeholders) những tên PHẢI được phân cách với 1 đấu mở '{' và 1 dấu đóng '}'
 .không được có khoảng trắng giữa các the
  delimiters and the (placeholders) name.

  (placeholders) các tên được cấu tạo NÊN chỉ gồm các kí tự `A-Z`, `a-z`,
  `0-9`, underscore `_`, and period `.`. Các kí tự khác chỉ rành riêng cho thay đổi trong tương lai của the (placeholders) specification.

  Những người thực hiện PHẢI sử dụng (placeholders) để triển khai various escaping strategies
  and thay đổi các phiên đăng nhập cho hiển thị. Những người sự dụng KHÔNG NÊN  pre-escape placeholder
  values since they can not know in which context the data will be displayed.

  Tiếp theo là ví dụ của placeholder thực hiện
  đã cung cấp để người dùng tham khảo:

  ```php
  /**
   * Interpolates context values into the message placeholders.
   */
  function interpolate($message, array $context = array())
  {
      // build a replacement array with braces around the context keys
      $replace = array();
      foreach ($context as $key => $val) {
          $replace['{' . $key . '}'] = $val;
      }

      // interpolate replacement values into the message and return
      return strtr($message, $replace);
  }

  // a message with brace-delimited placeholder names
  $message = "User {username} created";

  // a context array of placeholder names => replacement values
  $context = array('username' => 'bolivar');

  // echoes "User bolivar created"
  echo interpolate($message, $context);
  ```

### 1.3 Ngữ cảnh

- Mỗi phương thức được chấp nhận 1 mảng dữ liệu. Điều này chính là giữ lại 
  thông tin không liên quan đó làm rõ những điều không phù hợp trong chuỗi.
Mảng có thể chứa bất cứ thứ gì. Nhũng người thực hiện PHẢI chăc chắn họ xử lí đữ liệu
  cũng như much lenience as possible. 1 gia trị được đưa ra PHẢI không có lỗi phát sinh 
  an exception nor raise any php error, warning or notice.

- If an `Exception` object is passed in the context data, it MUST be in the
  `'exception'` key. Logging exceptions is a common pattern and this allows
  implementors to extract a stack trace from the exception when the log
  backend supports it. Implementors MUST still verify that the `'exception'`
  key is actually an `Exception` before using it as such, as it MAY contain
  anything.

### 1.4 Helper classes and interfaces

- The `Psr\Log\AbstractLogger` class lets you implement the `LoggerInterface`
  very easily by extending it and implementing the generic `log` method.
  The other eight methods are forwarding the message and context to it.

- Similarly, using the `Psr\Log\LoggerTrait` only requires you to
  implement the generic `log` method. Note that since traits can not implement
  interfaces, in this case you still have to implement `LoggerInterface`.

- The `Psr\Log\NullLogger` is provided together with the interface. It MAY be
  used by users of the interface to provide a fall-back "black hole"
  implementation if no logger is given to them. However conditional logging
  may be a better approach if context data creation is expensive.

- The `Psr\Log\LoggerAwareInterface` only contains a
  `setLogger(LoggerInterface $logger)` method and can be used by frameworks to
  auto-wire arbitrary instances with a logger.

- The `Psr\Log\LoggerAwareTrait` trait can be used to implement the equivalent
  interface easily in any class. It gives you access to `$this->logger`.

- The `Psr\Log\LogLevel` class holds constants for the eight log levels.

2. Package
----------

The interfaces and classes described as well as relevant exception classes
and a test suite to verify your implementation is provided as part of the
[psr/log](https://packagist.org/packages/psr/log) package.

3. `Psr\Log\LoggerInterface`
----------------------------

```php
<?php

namespace Psr\Log;

/**
 * Describes a logger instance
 *
 * The message MUST be a string or object implementing __toString().
 *
 * The message MAY contain placeholders in the form: {foo} where foo
 * will be replaced by the context data in key "foo".
 *
 * The context array can contain arbitrary data, the only assumption that
 * can be made by implementors is that if an Exception instance is given
 * to produce a stack trace, it MUST be in a key named "exception".
 *
 * See https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md
 * for the full interface specification.
 */
interface LoggerInterface
{
    /**
     * System is unusable.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function emergency($message, array $context = array());

    /**
     * Action must be taken immediately.
     *
     * Example: Entire website down, database unavailable, etc. This should
     * trigger the SMS alerts and wake you up.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function alert($message, array $context = array());

    /**
     * Critical conditions.
     *
     * Example: Application component unavailable, unexpected exception.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function critical($message, array $context = array());

    /**
     * Runtime errors that do not require immediate action but should typically
     * be logged and monitored.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function error($message, array $context = array());

    /**
     * Exceptional occurrences that are not errors.
     *
     * Example: Use of deprecated APIs, poor use of an API, undesirable things
     * that are not necessarily wrong.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function warning($message, array $context = array());

    /**
     * Normal but significant events.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function notice($message, array $context = array());

    /**
     * Interesting events.
     *
     * Example: User logs in, SQL logs.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function info($message, array $context = array());

    /**
     * Detailed debug information.
     *
     * @param string $message
     * @param array $context
     * @return null
     */
    public function debug($message, array $context = array());

    /**
     * Logs with an arbitrary level.
     *
     * @param mixed $level
     * @param string $message
     * @param array $context
     * @return null
     */
    public function log($level, $message, array $context = array());
}
```

4. `Psr\Log\LoggerAwareInterface`
---------------------------------

```php
<?php

namespace Psr\Log;

/**
 * Describes a logger-aware instance
 */
interface LoggerAwareInterface
{
    /**
     * Sets a logger instance on the object
     *
     * @param LoggerInterface $logger
     * @return null
     */
    public function setLogger(LoggerInterface $logger);
}
```

5. `Psr\Log\LogLevel`
---------------------

```php
<?php

namespace Psr\Log;

/**
 * Describes log levels
 */
class LogLevel
{
    const EMERGENCY = 'emergency';
    const ALERT     = 'alert';
    const CRITICAL  = 'critical';
    const ERROR     = 'error';
    const WARNING   = 'warning';
    const NOTICE    = 'notice';
    const INFO      = 'info';
    const DEBUG     = 'debug';
}
```
