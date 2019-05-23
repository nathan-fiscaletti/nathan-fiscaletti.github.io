---
title: "Creating a custom numeric base in PHP"
published: true
comments: true
---

Recently I wanted to create URL's that used words instead of integers for specific posts, however, these URL's needed to conform to three points.

  1. Be completely unique.
  2. Be repeatable with the same seed.
  3. Be reversible back to their seed.

The idea of a numeric base (or "radix") is pretty simple, you have `x` number of digits to work with (in the most common base, base-10, you have 10 digits to work with: 0-9.) When you reach `x` in the current numeral place, that numeral loops back to `0`and the numeral place to the left has `1` added to it. Although my description of a base may not be the best, you can read more about it here: [Number Bases - Math is Fun](https://www.mathsisfun.com/numbers/bases.html)

Most numeric bases up to base-36 use the alphabet to supplement the missing numerals. For example, base-16 uses `[0-1][A-F]`, this way it has the 16 "digits" it requires to create a number in base-16 format.

This is accomplished in PHP using a library I wrote that not only allows for this type of normal "base" expression, but also allows for some more complicated usage that technically speaking go outside of the constraints of a "base".

[Find the library here](https://gist.github.com/nathan-fiscaletti/5c999a60d17ee1bff0a44bb6365c1e6b)

## General Usage

Using the library is incredibly simple. To create a base simply use the following:

```php
$base5 = new Base(5);
```

You can then convert to and from the base using `->parse(int)` and `->toBase10($parsedValue)`:
    
```php
$b5value = $base5->parse(99);
echo '99 in base5 is '.$b5value.PHP_EOL;

echo 'converted back to base-10 is: '.$base5->toBase10($b5value).PHP_EOL;
```

You can also convert to other bases using the following:
    
```php
$base2 = new Base(2);
$base3 = new Base(3);
$base5 = new Base(5);

// Convert from base10 to base2
$b2val = $base2->parse(1325);
echo "b10(1325)\t==\tb2($b2val)".PHP_EOL;

// Convert from base2 to base5
$b5val = $base2->convert($b2val, $base5);
echo "b2($b2val)\t==\tb5($b5val)".PHP_EOL;

// Convert from base5 to base3
$b3val = $base5->convert($b5val, $base3);
echo "b5($b5val)\t==\tb3($b3val)".PHP_EOL;
```

You can create any base up to base-36, after that you must supply a custom dictionary.

## Custom Dictionary

So, my thought was to supplement all digits within my base with a dictionary of words taken from a text file. My base would be the number of words stored in the file. If the file had `88` words in it, it would be base-88, however wouldn't rely on any numbers or the alphabet and instead would use each word from the document as a "digit".

---

Let's say I have the following text document 
    
```
hello
there
friend
```

This would be base-3, where `hello` would represent `0`, `there`, would represent `1`, and `friend` would represent `2`.

So, using the following code I could convert a number to base-3 using this special dictionary.
    
```php
$dictionary = file('words.txt', FILE_IGNORE_NEW_LINES);
$base = new Base(count($dictionary));
$base->setLibrary($dictionry);
echo $base->parse(954);
```

Which would result in `ThereHelloFriendFriendThereHelloHello`

---

You can convert back to base 10 using the `->toBase10` member function of the `$base` instance.

For example, with this library you can use separate dictionaries for different places of the resulting number. In this example, I use one dictionary for the `1s` place, but another for all other numbers. 
    
```php
$dictionary1 = file("./adjectives.txt", FILE_IGNORE_NEW_LINES);
$dictionary2 = file("./nouns.txt", FILE_IGNORE_NEW_LINES);

$baseCustom = new Base(count($dictionary1));
$baseCustom->setLibrary($dictionary1);

// 0 = 1s place, 1 = 10s place, ...
$baseCustom->putLibrary($dictionary2, 0);

echo $baseCustom->parse(9984134);
```

> Note: When using multiple dictionaries like this, they must conform to the same base. (Have the same number of "digits" defined in them.)

This "multiple dictionaries" feature was added specifically because I wanted to be able to have a format of "AdjectiveAdjectiveNoun" for my final URLs.

So, a resulting URL might look something like `https://website.com/AbidingEsotericAnthropology` instead of `https://website.com/99653422`

This would allow me to maintain uniqueness, reversibility and repeatability while still having easy to remember URLs. In order to avoid the URLs becoming long strings of words, you would need to have a higher base. For example, I use a base of around 5,000. Meaning I can have up to 25 million (5,000²) combinations before I would even exceed two words. 

## Constraints

If you use a custom dictionary they are constrained to three things. 

  * They must use only lower case entries as the library uses capitalization to denote a new "digit".
  * Each entry in the list must be unique
  * If you are using multiple dictionaries for different places in the number, the dictionaries must be of equal lengths.
