---
title: "C++ Primer : Chapter 17"

categories:
    - cpp

tags:
    - [C++, Programming Language, tuple, regular expression, random, IO]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-20
---

# Specialized Library Facilities

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## The tuple Type
A `std::tuple` is a template that is similar to a `std::pair`
- like `std::pair`, it can have members whose types vary from one `tuple` type to another
- unlike `std::pair`, it can have any number of members
- a `std::tuple` is most useful when we want to combine some data into a single object but do not want to bother to define a data structure to represent those data 
- it's defined in the `<tuple>` header

### Operations On tuples
The following table shows the operations of `std::tuple`

||Operations on `std::tuple`|
|:---|:---|
|`tuple<T1, T2, ..., Tn> t;`|`t` is a `std::tuple` with as many members there are types `T1` ... `Tn`. the members are value initialized|
|`tuple<T1, T2, ..., Tn> t(v1, v2, ..., vn);`|`t` is a `std::tuple` with types `T1`...`Tn` in which each members is initialized from the corresponding initializer, vi. this constructor is `explicit`|
|`make_tuple(v1, v2, ..., vn)`|returns a `std::tuple` initialized from the given initializers. the type of the `std::tuple` is inferred from the types of the initializers.|
|`t1 == t2`|Two tuples are equal if they have the same number of members and if each|
|`t1 != t2`|pair of members are equal. Uses each member's underlying `==` operator. once a member is found to be unequal, subsequent members are not tested.|
|`t1 relop t2`|relational operations on `std::tuple`s using dictionary ordering. the `std::tuple`s must have the same number of members. members of `t1` are compared with the corresponding members from `t2` using the `<` operator|
|`get<i>(t)`|returns a reference to the `i`th data member of `t`; if `t` is an Ivalue, the result is an lvalue reference; otherwise, it is an rvalue reference. all members of a `std::tuple` are `public`; as usual, the index starts at `0`|
|`tuple_size<TupleType>::value`|a class template that can be instantiated by a `std::tuple` type and has a `public constexpr static` data member named value of type size `t` that is number of members in the specified `std::tuple` type.|
|`tuple element<i, TupleType>::type`|a class template that can be instantiated by an integral constant and a `std::tuple` type and has a `public` member named `type` that is the type of the specified members in the specified `std::tuple` type.|

- the following codes show the example of the use of `std::tuple`
    ```c++
    #include <tuple>
    auto item = std::make_tuple(3, "haha", 7.02);
    std::get<0>(item) /= 3;         // the first element is now 1
    ```

## The bitset Type
If you want to handle the collections of bits that are larger than the longest integral type, **std::bitset** class which is defined in the `<bitset>` header would be a good solution

## Regular Expressions
A **regular expression** is a way of describing a sequence of characters

### Regular Expression Library Components
The C++ regular-expression library (RE library) is defined in the `<regex>` header
- the following table shows the components of the library

    ||Regular Expression Library Components|
    |:---|:---|
    |`regex`|class that represents a regular expression|
    |`regex_match`|matches a sequence of characters against a regular expression|
    |`regex_search`|finds the first subsequence that matches the regular expression|
    |`regex_replace`|replaces a regular expression using a given format|
    |`sregex_iterator`|iterator adaptor that calls `regex_search` to iterate through the matches in a string|
    |`smatch`|container class that holds the results of seraching a string|
    |`ssub_match`|results for a matched subexpression in a string|

- the following table shows arguments regarding `std::regex_search` and `std::regex_match`

    ||Arguments to `std::regex_search` and `std::regex_match`|
    |:---|:---|
    ||**Note: These operations return `bool` indicating whether a match was found**|
    |`(seq, m, r, mft)`|look for the regular expression in the `std::regex` object `r` in the character sequence `seq`. `m` is a *match* object, which is used to hold details about the match|
    |`(seq, r, mft)`|`m` and `seq` must have the compatibal types. `mft` is an optional `std::regex_constants::match_flag_type` value. these values affect the match process|

- the following table shows the match flags

    ||**Match Flags**|
    |:---|:---|
    ||**Defined in `std::regex_constants::match_flag_type`**|
    |`match_default`|equivalent to `format_default`|
    |`match_not_bol`|don't treat the first character as the beginning of the line|
    |`match_not_eol`|don't treat the last character as the end of the line|
    |`match_not_bow`|don't treat the first character as the beginning of a word|
    |`match_not_eow`|don't treat the last character as the end of a word|
    |`match_any`|if there is more than one match, any match can be returned|
    |`match_not_null`|Don't match an empty sequence|
    |`match_continuous`|the match must begin with the first character in the input|
    |`match_prev_avail`|the input sequence has characters before the first|
    |`format_default`|replacement string uses the ECMAScript rules|
    |`format_sed`|replacement string uses the rules from POSIX `sed`|
    |`format_no_copy`|don't output the unmatched parts of the input|
    |`format_first_only`|replace only the first occurrence|

### Regular Expression Classes and the Input Sequence Type
The following table shows which classes support a certain input sequence type

||Regular Expression Library Classes|
|:---:|:---:|
|**If Input Sequence Has Type**|**Use Regular Expression Classes**|
|`std::string`|`std::regex`, `std::smatch`, `std::ssub_match` and `std::sregex_iterator`|
|`const char*`|`std::regex`, `std::cmatch`, `std::csub_match` and `std::cregex_iterator`|
|`std::wstring`|`std::wregex`, `std::wsmatch`, and `std::wsregex_iterator`|
|`const wchar_t*`|`std::wregex`, `std::wcmatch` and `std::wcregex_iterator`|

### regex Classes
`std::regex` class is initialized with the **pattern**
- the **pattern** consists of one or more **subexpressions**
    * by default, the regular-expression language used by `std::regex` objects is ECMAScript
    * you can find the whole details regarding it [**here**](https://en.cppreference.com/w/cpp/regex/ecmascript)
- the `std::regex` class is used to call `regex_match`, `regex_search`, `regex_replace`, or to get `sregex_iterator`
    * after processing it, the `std::smatch` object is returned
    * and you can get `std::ssub_match` from `std::smatch` object
- the following table shows the operations on `std::regex`

    ||`std::regex` (and `std::wregex`) Operations |
    |:---:|:---:|
    |`regex r(re)`|`re` represents a regular expression and can be a `std::string`, a pair of iterators denoting a range of characters, a pointer to a null-terminated character array, a character pointer with a count, or a braced list of characters|
    |`regex r(re, f)`|`f` are flags that specify how the object will execute. `f` is set from the values listed below. if `f` is not specified, it defaults to `ECMAScript`|
    |`r1 = re`|replace the regular expression in `r1` with `re`. `re` represents a regular expression and can be another `std::regex`, a `std::string`, a pointer to a null-terminated character array, or a braced list of characters|
    |`r1.assign(re, f)`|same effect as using the assignment operator. `re` and optional flag `f` same as corresponding arguments to `std::regex` constructors|
    |`r1.mark_count()`|returns the number of subexpressions in `r`|
    |`r.flags()`|returns the flags set for `r`|
    ||*Note: Constructors and assignment operations mahy throw expcetions of type `std::regex_error`*|
    ||**Flags Specified When a `std::regex` Is Defined**|
    |``|**Defined in `std::regex` and `std::regex_constants::syntax_option_type`**|
    |`icase`|ignore case during the match|
    |`nosub`|don't store subexpression matches|
    |`optimize`|favor speed of execution over speed of construction|
    |`ECMAScript`|use grammar as specified by ECMA-262|
    |`basic`|use POSIX basic regular-expression grammar|
    |`extended`|use POSIX extended regular-expression grammar|
    |`awk`|use grammar from the POSIX version of `awk` language|
    |`grep`|use grammar from the POSIX version of `grep`|
    |`egrep`|use grammar from the POSIX version of `egrep`|

    * note that we can specify one or more flags that affect how the `std::regex` operates
- a regular expression is "compiled" at **run time** when a `std::regex` object is initialized with or assigned a new pattern
    * therefore, construcing a `std::regex` objedct and assigning a new regular expression to an existing `std::regex` can be **time-consuming**
        + which means, you should try to avoid creating more `std::regex` objects than needed
        + moreover, if you use a regular expression in a loop,
            - you should create it outside the loop
    * if we make a mistake in writing a regular expression, then at **run time** the library will throw an exception of type `std::regex_error`
        + `std::regex_error` has `.what()` as other exception classes, and it also has `.code()` method which returns a numeric code corresponding to the type of error that was encountered
        + the following table shows the error conditions

        ||Regular Expression Error Conditions|
        |:---:|:---:|
        ||**Defined in `std::regex` and in `std::regex_constants::error_type`**|
        |`error_collate`|invalid collating element request|
        |`error_ctype`|invalid character class|
        |`error_escape`|invalid escape character or trailing escape|
        |`error_backref`|invalid back reference|
        |`error_brack`|mismatched bracket `([` or `])`|
        |`error_paren`|mismatched parentheses `((` or `))`|
        |`error_brace`|mismatched brace `({` or `})`|
        |`error_badbrace`|invalid range inside a `{ }`|
        |`error_range`|invalid character range (e.g., `[z-a]`)|
        |`error_space`|insufficient memory to handle this regular expression|
        |`error_badrepeat`|a repetition character (`*, ?, +`, or `{`) was not preceded by a valid regular expression|
        |`error_complexity`|the requested match is too complex|
        |`error_stack`|insufficient memory to evaluate a match|


### Subexpressions
The following codes show some examples of **subexpressions**
```c++
std::string pattern;
pattern = "[[:alpha:]]";              // single alphabetic character  
pattern = "[[:alpha:]]+";             // one or more alphabetic character(s)  
pattern = "[[:alpha:]]*";             // zero or more alphabetic character  
pattern = "[[:alpha:]]?";             // zero or one alphabetic character (used to make the component optional)

pattern = "[^c]";                     // single character except for 'c'  
pattern = "[^c]ei";                   // any such letter that is followed by "ei"
pattern = "[-. ]";                    // - or . or space

pattern = "\\d";                      // single digit
pattern = "\\d{n}";                   // sequence of n digits

pattern = "(\\()?(\\d{3})(\\))?([-. ])?(\\d{3})([-. ])?(\\d{4})";       // example of the pattern which contains multiple subexpressions to target phone number 

std::regex r(pattern);                // initializes regex class with the pattern
```
- note that the ECMAScript uses the special characters like C++
    * for instance, if you need set .(dot) as a pattern, you need to use \\(backslash) to represent it
        + however, if you use the special character in `[ ]` (which matches any of characters inside square brackets), they have no speical meaning in there
    * moreover, the backslash is also the special character in C++, you need to put additional backslash to represent it
- if there are many subexpressions, they are usually parenthesized  

### Use of sregex_iterator
If you want to get **all** matches, using `std::sregex_iterator` is a good solution
```c++
std::string target = "input sequence";
std::string pattern = "some pattern";
std::regex r(pattern);

for (std::sregex_iterator cur(target.begin(), target.end(), r), end; cur != end; cur++) {
    std::cout << "prefix : " << cur->prefix().str() <<  "\nmatched : " << cur->str() << "\nsuffix : " << cur->suffix().str() << std:: endl; 
}
```
- it's an iterator adaptor bound to the input sequence and `std::regex` object
- when you construct the iterator with the proper arguments, it implicitly calls `std::regex_search` to find the first match
    * if you default construct it, it would be the "off-the-end" iterator
- dereferencing it returns `std::smatch` object that contains the most recent search result
    * the following table shows the operations on `std::smatch`

        ||`std::smatch` Operations|
        |:---:|:---:|
        ||**These Operations also apply to the `std::cmatch`, `std::wsmatch` and the cooresponding `std::csub_match`, `std::wssub_match` and `std::wcsub_match` types**|
        |`m.ready()`|returns `true` if `m` has been set by a call to `std::regex_search` or `regex_match`; `false` otherwise; operations on `m` are undefined if `.ready()` returns `false`|
        |`m.size()`|returns `0` if the match failed; otherwise, returns `1` + the number of subexpressions in the most recently matched regular expression.|
        |`m.empty()`|`true` if `m.size()` is `0`.|
        |`m.prefix()`|an `std::ssub_match` representing the sequence before the match.|
        |`m.suffix()`|an `std::ssub_match` representing the part after the end of the match.|
        |`m.format(dest, fmt, mft)`|Produces formatted output using the format string `fmt`, the|
        |`m.format(fmt, mft)`|match in `m`, and the optional `match_flag_type` flags in `mft`. the first version writes to the output iterator `dest` and takes `fmt` that is either a `std::string` or a pair of pointers denoting a range in a character array. the second version returns a `std::string` that holds the output and takes `fmt` that is a `std::string` or a pointer to a null-terminated character array. `mft` defaults to `format_default`.|
        |`regex_replace`|Iterates through `seq`, using `regex_search` to find successive|
        |`(dest, seq, r, fmt, mft)`|matches to `regex` `r`. uses the format string `fmt` and optional|
        |`regex_replace`|`match_flag_type` flags in `mft` to produce its output.|
        |`(seq, r, fmt, mft)`|the first version writes to the output iterator `dest`, and takes a pair of iterators to denote `seq`. the second returns a `std::string` that holds the output and `seq` can be either a `std::string` or a pointer to a null-terminated character array. in all cases, `fmt` can be either a `std::string` or a pointer to a null-terminated character array, and `mft` defaults to `match_default`.|
        ||**In the operations that take an index, n defaults to zero and must be less than `m.size()`.**|
        ||**The first submatch (the one with index 0) represents the overall match.**|
        |`m.length(n)`|returns the size of the `n`th matched subexpression.|
        |`m.position(n)`|returns the distance of the `n`th subexpression from the start of the sequence.|
        |`m.str(n)`|returns the matched string for the `n`th subexpression.|
        |`m[n]`|returns `std::ssub_match` object corresponding to the `n`th subexpression.|
        |`m.begin(), m.end()`|returns iterators across the `std::sub_match` elements in `m`.|
        |`m.begin(), m.cend()`|as usual, `.cbegin()` and `.cend()` return `const_iterator`s.|

    * the following table shows the operations on `std::ssub_match`

        ||Submatch Operations|
        |:---:|:---:|
        ||**Note: These operations apply to `std::ssub_match`, `std::csub_match`, `std::wssub_match`, `std::wcsub_match`**|
        |`matched`|a `public bool` data member that indicates whether this `std::ssub_match` was matched|
        |`first`|`public` data members that are iterators to the start and one past the end of|
        |`second`|the matching sequence. ff there was no match, then `first` and `second` are equal.|
        |`length()`|returns the size of this match. returns `0` if `matched` is `false`|
        |`str()`|returns a `std::string` containing the matched portion of the input. returns the empty `std::string` if `matched` is `false`|
        |`s = ssub`|converts the `std::ssub_match` object `ssub` to the `std::string` `s`. equivalent to `s = ssub.str()`. the conversion operator is not `explicit`|

- if you increment it, it would find the next one

### Use of regex_replace
If you want to use some of the matched results with the different format, you can achieve this by using `std::regex_replace`
```c++
std::string pattern = "(\\()?(\\d{3})(\\))?([-. ])?(\\d{3})([-. ])?(\\d{4})";       // example of the pattern which contains multiple subexpressions to target phone number 
std::string format = "$2.$5.$7";        // reformat numbers to ddd.ddd.dddd

std::regex r(pattern);
std::string target = "(123) 456-7890";
std::cout << std::regex_replace(target, r, format) << std::endl;

std::smatch results;
std::regex_search(target, results, r);
std::cout << results.format(format) << std::endl;
```
- note that `&n` represents the match result from the nth subexpression
    * unlike indexing, it counts from 1
- `std::regex_replace()` and `.format()` from `std::smatch` object returns the same output



## Random Numbers
Prior to the new standard, both C and C++ relied on a simple C library function named `rand`
- that function produces pseudorandom integers that are uniformly distributed in the range from 0 to a system-dependent maximum value that is at least 32767
    * the `rand` function has several problems :
        + many, if not most, programs need numbers in a different range from the one produced by `rand`
        + some applications require random floating-point numbers
        + some programs need numbers that reflect a nonuniform distribution
- to solve these problems, the library defines the `<random>` header which contains a set of cooperating classes :
    * **random-number engines**
        + an engine is a type that generates a sequence of random `unsigned` integers
    * **random-number distributions**
        + a distribution is a type that uses an engine to return numbers according to a particular probability distribution
- hence, the C++ programs should not use the library `rand` function
    * instead, they should use the engine with the proper distribution

### Random-Number Engines and Distributions
The random-number engines are function-object classes that define a call operator that takes no arguments and returns a random `unsigned` number
- random-number engines
    * we can generate raw random number by calling an object of a random-number engine type
    * there are several engines defined in the library
        + each compiler designates one of these engines as the `std::default_random_engine` type
        + the list of engines would be covered later in this chapter
    * the following table shows the operations of the engine

    ||Random Number Engine Operations|
    |:---|:---|
    |`Engine e;`|default constructor; uses the default seed for the engine type|
    |`Engine e(s);`|uses the integral value `s` as the seed|
    |`e.seed(s)`|reset the state of the engine using the seed `s`|
    |`e.min()`|the smallest number this generator will generate|
    |`e.max()`|the largest number this generator will generate|
    |`Engine::result_type`|the `unsigned` integral type this engine generates|
    |`e.discard(u)`|advance the engine by `u` steps; `u` has type `unsigned long long`|

    + note that the seed is `std::uint_fast32_t` which is usually just a 32-bit `int`. Every value in the range [0 ~ 2^32) should produce different results
    + using `time()` function defined in the `<ctime>` header as a seed usually doesn't work if the program is run repeatedly as part of an automated process
- random-number distributions
    * to get the properly transformed random number, we need to use the distribution object
    * there are several distributions defined in the library
        + the list of distributions would be covered later in this chapter as well
        + with the exception of the `std::bernouilli_distribution`, the distribution types are templates
        + each of these templates takes a single type parameter that names the result type that the distribution will generate
        + some distribution templates can be used to generate only floating-point numbers; others can be used to generate only integers
        + the distribution templates define the default template parameter
            - the default for the integral distribution is `int`
            - the default for the floating-point is `double`
        + the constructor for each distribution has parameters that are specific to the kind of distribution
            - some of these parameters specify the range of the distribution, such as `std::uniform_int_distribution<IntT> u(m, n)`
            - and these ranges are alwasy **inclusive**, unlike iterator ranges
    * the following table shows the operations of the distribution

    ||Distribution Operations|
    |:---|:---|
    |`Dist d`|default consturctor; makes `d` ready to use|
    |``|other constructors depend on the type of `Dist`; and the distruction consturctors are `explicit`|
    |`d(e)`|successive calls with the same `e` produce a sequence of random numbers according to the distribution type of `d`; `e` is a random-number engine object|
    |`d.min()`|return the smallest number `d(e)` will generate|
    |`d.max()`|return the largest number `d(e)` will generate|
    |`d.reset()`|reestablish the state of `d` so that subsequent uses of `d` don't depend on values `d` has already generated|

- when we refer to a **random-number generator**, we mean the combination of a distribution object with an engine

### Things to Consider
- a given random-number generator always produces the same sequence of numbers
    * a function with a **local** random-number generator should make that generator (both the engine and distribution objects) `static`
    * otherwise, the function will generate the identical seuqnce on each call
- because engines return the same sequence of numbers
    * it's essential that we declare engines **outside** of loops
    * otherwise, we'd create a new engine on each iteration and generate the same values on each iteration
    * similarly, distributions may retain state and should also be defined outside loops

### List of Engines and Distributions
You can see the list of engines and distributions on p.882 ~ p.885
- or [here](https://en.cppreference.com/w/cpp/numeric/random)


## The IO Library Revisited

### Formatted Input and Output
The library defines a set of **manipulators** that **modify the format** state of a stream
- manipulators are used for two broad categories of output control :
    * controlling the presentation of numeric values
    * controlling the amount and placement of padding
- most of the manipulators that change the format state provide set/unset pairs
- it's usually best to **undo** whatever changes are made as soon as those changes are no longer needed
    * manipulators that change the format state of the stream usually leave the format state changed for all subsequent IO
- `std::endl` is the typcial example of manipulators
    * which means, you use the other manipulators in the same way we use the `std::endl`

### List of Manipulators
The library defines manipulators in two header files
- `<iostream>`

    |||Manipulators Defined in `<iostream>`|
    |:---|:---|:---|
    ||`boolalpha`|display `true` and `false` as strings|
    |*|`noboolalpha`|display `true` and `false` as `0`, `1`|
    ||`showbase`|generate prefix indicating the numeric base of integral values|
    |*|`noshowbase`|do not generate notational base prefix|
    ||`showpoint`|always display a decimal point for floating-point values|
    |*|`noshowpoint`|display a decimal point only if the value has a fractional part|
    ||`showpos`|display + in nonnegative numbers|
    |*|`noshowpos`|Do not display + in nonnegative numbers|
    ||`uppercase`|print `0x` in hexadecimal, `E` in scientific|
    |*|`nouppercase`|print `0x` in hexadecimal, `e` in scientific|
    |*|`dec`|display integral values in decimal numeric base|
    ||`hex`|display integral values in hexadecimal numeric base|
    ||`oct`|display integral values in octal numeric base|
    ||`left`|add fill characters to the right of the value|
    ||`right`|add fill characters to the left of the value|
    ||`internal`|add fill characters between the sign and the value|
    ||`fixed`|display floating-point values in decimal notation|
    ||`scientific`|display floating-point values in scientific notation|
    ||`hexfloat`|display floating-point values in hex (new to C++ 11)
    ||`defaultfloat`|reset the floating-point format to decimal (new to C++ 11)|
    ||`unitbuf`|flush buffers after every output operation|
    |*|`nounitbuf`|restore normal buffer flushing|
    |*|`skipws`|skip whitespace with input operators|
    ||`noskipws`|do not skip whitespace with input operators|
    ||`flush`|flush the `std::ostream` buffer|
    ||`ends`|insert null, then flush the `std::ostream` buffer|
    ||`endl`|insert newline, then flush the `std::ostream` buffer|
    |*|*indicates the default stream state*||

    * unless you need to control the presentation of a floating-point number, it's usually best to let the library choose the notation
- `<iomanip>`

    ||Manipulators Defined in `<iomanip>`|
    |:---|:---|
    |`setfill(ch)`|fill whitespace with `ch`|
    |`setprecision(n)`|set floating-point precision to `n`|
    |`setw(w)`|read or write value to `w` characters|
    |`setbase(b)`|output integers in base `b`|

    * `std::setw`, like `std::endl`, does not change the internal state of the output stream
        + it determines the size of only the next output
    
### Unformatted I/O Operations
The library provides a set of low-level operations that support **unformatted IO**
- Single-Byte

    ||Single-Byte Low-Level IO Operations|
    |:---|:---|
    |`is.get(ch)`|put the next byte from the `std::istream` `is` in character `ch`. returns `is`|
    |`os.put(ch)`|put the character `ch` onto the `std::ostream` `os`. returns `os`.|
    |`is.get()`|returns next byte from `is` as an `int`.|
    |`is.putback(ch)`|put the character `ch` back on `is`; returns `is`.|
    |`is.unget()`|move `is` back one byte; returns `is`.|
    |`is.peek()`|return the next byte as an `int` but doesn't remove it.|

- Multi-Byte

    ||Multi-Byte Low-Level IO Operations|
    |:---|:---|
    |`is.get(sink, size, delim)`|reads up to size bytes from `is` and stores them in the character array beginning at the address pointed to by `sink`. reads until encountering the `delim` character or until it has read size bytes or encounters end-of-file. If `delim` is present, it is left on the input stream and not read into `sink`.|
    |`is.getline(sink, size, delim)`|same behavior as the three-argument version of get but reads and discards `delim`.|
    |`is.read(sink, size)`|reads up to size bytes into the character array `sink`. Returns `is`.|
    |`is.gcount()`|returns number of bytes read from the stream `is` by the last call to an unformatted read operation.|
    |`os.write(source, size)`|writes size bytes from the character array source to `os`. returns `os`.|
    |`is.ignore(size, delim)`|reads and ignores at most size characters up to and including `delim`. unlike the other unformatted functions, `.ignore()` has default arguments: `size` defaults to `1` and `delim` to end-of-file.|

    * it's a common error to intend to remove the delimiter from the stream but to 
    forget to do so
- if you can use the more type-safe, higher-level operations supported by the library, it's usually better to do so

## Random Access to a Stream
The library provides a pair of functions to *seek* to a given location and to *tell* the current location in the associated stream
- random IO is an inherently system-dependent
    * to understand how to use these features, you must consult your system's documentation
- the `std::istream` and `std::ostream` types usually do **not support** random access
    * other types (`std::fstream` or `std::sstream`) **do support**
- to support random access, the IO types maintain a **marker** that determines where the next read (getting) or write (putting) will happen
    * but note that there is only **single** marker, we must do a `seek` to reposition the marker whenever we switch between reading and writing so that it's safe to do so
- the following table shows the random IO functions

    ||Seek and Tell Functions|
    |:---|:---|
    |`tellg()`|returns the current position of the marker in an input stream|
    |`tellp()`|returns the current position of the marker in an output stream|
    |`seekg(pos)`|repositions the marker in an input or output stream to the given|
    |`seekp(pos)`|absolute address in the stream. `pos` is usually a value returned by a previous call to the corresponding `tellg()` or `tellp()` function.|
    |`seekp(off, from)`|repositions the marker for an input or output stream integral number|
    |`seekg(off, from)`|`off` characters ahead or behind `from`. `from` can be one of|
    ||`beg`, seek relative to the beginning of the stream|
    ||`cur`, seek relative to the current position of the stream|
    ||`end`, seek relative to the end of the stream|

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}