---
title: WP I18n
description: A standard OOP implementatin of internationalization for WordPress
layout: default
project_type: php
package_name: dhii/wp-i18n
package_url: https://packagist.org/packages/dhii/wp-i18n
---

**Package**: [`{{ page.package_name }}`][package-url]  
**Standards**: [Dhii I18n][]

## Details
This package wraps around some of the [WordPress internationalization][wp-i18n] functions to provide an OOP interface. By implementing the [Dhii I18n][] standard, it can be used with every compatible consumer. It allows you to use WordPress i18n, while being not tied to WordPress. Without this, it would be impossible to write completely framework-agnostic modules that would still work with WordPress. Furthermore, it allows you to avoid having to specify the text domain everywhere: you would normally have no more than one text domain per module anyway, and having to specify it **while consuming** would couple you to it, and therefore it would couple you to WordPress. Use this implementation to break free from this horrible constrant!

## Usage
The best way is to abstract internationalization entirely in your code; at Dhii, we do it by relying on an [abstract method `__()`][i18n-method]. Then, in a more specialized, perhaps concrete class, use the [`StringTranslatorConsumingTrait`][]. This will cause the `__()`method to use the assigned translator to actually translate the strings. For implementations that want to internationalize, but don't actually need to localize, [`StringTranslatingTrait`][] provides a no-op implementation that will still interpolate values, but will not translate.

## Examples
```php
use Dhii\I18n\StringTranslatorInterface;
use Dhii\I18n\StringTranslatorConsumingTrait;
use Dhii\I18n\StringTranslatorAwareTrait;
use Dhii\Wp\I18n\FormatTranslator;

/**
 * A generic greeter.
 *
 * Doesn't know how to translate, but concerns itself only with greeting, while at the same time
 * internationalizing the greeting message. Not coupled to WordPress.
 */
class AbstractGreeter
{
    /**
     * Retrieves a greeting based on a name.
     *
     * @param string $name Name of the person to greet.
     *
     * @return string The greeting string.
     */
    public function getGreeting($name)
    {
        $message = $this->__('Hello, %1$s!!', [$name]);
    
        return $message;
    }

    /**
     * Translates a string, and replaces placeholders.
     *
     * @since [*next-version*]
     * @see sprintf()
     *
     * @param string $string  The format string to translate.
     * @param array  $args    Placeholder values to replace in the string.
     * @param mixed  $context The context for translation.
     *
     * @return string The translated string.
     */
    abstract protected function __($string, $args = [], $context = null);
}

/**
 * A specialized greeter implementation.
 *
 * Uses a translator to localize the greetings. Still not coupled to WordPress.
 */
class MyGreeter
{
    use StringTranslatorConsumingTrait;
    use StringTranslatorAwareTrait;
    
    public function __construct(StringTranslatorInterface $translator)
    {
        $this->_setTranslator($translator);
    }
}

// Glue code
$translator = new FormatTranslator('my-text-domain');
$greeter = new MyGreeter($translator);

$greeting = $greeter->getGreeting('Sarah'); // Perhaps: "Bonjour, Sarah!"
```

## Limitations
At present, the standard does not support plural forms, and therefore this implementation doesn't support them either.


[Dhii I18n]:                                    https://packagist.org/packages/dhii/i18n-interface
[`AddCapableOrderedList`]:                      https://github.com/Dhii/set/blob/develop/src/AddCapableOrderedList.php
[wp-i18n]:                                      https://codex.wordpress.org/I18n_for_WordPress_Developers
[i18n-method]:                                  https://github.com/Dhii/map/blob/develop/src/MakeCapableMapTrait.php#L156
[`StringTranslatorConsumingTrait`]:             https://github.com/Dhii/i18n-helper-base/blob/develop/src/StringTranslatorConsumingTrait.php
[`StringTranslatingTrait`]:                     https://github.com/Dhii/i18n-helper-base/blob/develop/src/StringTranslatingTrait.php
[package-url]:                                  {{ page.package_url }}
