---
title: Regular Expressions in Python
slug: python-regular-expressions

publish_timestamp: Sept. 16, 2020
url: https://www.codingforentrepreneurs.com/blog/python-regular-expressions/

---


Regular expressions, aka `regex`, is incredibly common to help us parse data. Before we discuss how, let's consider a practical example by using US Phone numbers. The following are *all* valid written phone number formats:

- +1-555-555-3121
- 1-555-555-3121
- 555-555-3121
- +1(555)-555-3121
- +15555553121

It's amazing that all of these numbers are the exact same just formatted slightly different. So how would we search a whole document for all possible derivations of phone number format?


"Machine learning!" you say. Well, that would probably work but it's overcomplicating this particular challenge. Instead, we can use pattern matching, aka regular expressions, to simplify the challenge.


Regular expressions are intimidating and take some time to wrap your head around. So I created this guide as a way to unpack how to effectively use Regular Expressions in Python. Many of these `regex` patterns and concepts overlap to other languages especially since Python regex was inspired by `Perl`.


Let's look at some code.


> Watch this tutorial on [youtube](https://www.youtube.com/watch?v=TpOboJe6uIo), in [this project](https://www.codingforentrepreneurs.com/projects/30-days-python-38), or at the bottom of this page.



```python
my_phone_number = "555-867-5309"
```

How do we get all the numbers (not the dashes `-`) from the above string? Let's first talk about the harder and more amateur way to do it:


```python
numbers = []
for char in my_phone_number:
    number_val = None
    try:
        number_val = int(char)
    except:
        pass
    if number_val != None:
        numbers.append(number_val)

numbers_as_str = "".join([f"{x}" for x in numbers])
numbers_as_str
```




    '5558675309'



Here's another way your intuition make take you:


```python
numbers_as_str2 = my_phone_number.replace("-", "")
numbers_as_str2
```




    '5558675309'



Finally, Python Strings (`str`) have a built-in method `.isdigit()` that can be applied to verify if the string contains a number or not. Here's how it's done:


```python
numbers_as_str3 = "".join([f"{x}" for x in my_phone_number if x.isdigit()])
numbers_as_str3
```




    '5558675309'



All of these methods are valid in that they achieve the goal but there's a more practical and robust way to do this. 

And that's with regular expressions. Let's see our first regex example:


```python
import re # the built-in regex library

pattern = r"\d+"
matches = re.findall(pattern, my_phone_number)
matches
```




    ['555', '867', '5309']



The `re.findAll` method actually illustrates a much better result --> each group of numbers has been parsed out by default. 

To me, it's easier to infer that `['555', '867', '5309']` is a phone number over something like `5558675309`. That's because I'm from the USA and that's how we typically group numbers.

We still haven't gotten to the core reason as to why we use regex. Let's think of another example.


```python
my_other_phone_numbers = "Hi there, my home number is 555-867-5309 and my cell number is +1-555-555-0007."

pattern = r"\d+"
matches = re.findall(pattern, my_other_phone_numbers)
matches
```




    ['555', '867', '5309', '1', '555', '555', '0007']



The numbers `['555', '867', '5309', '1', '555', '555', '0007']` are much more challenging to distinguish a list of phone numbers within a string. The length of that string was only 79 characters (including spaces/punctuation). Imagine if we had thousands of characters? 

What to do? The answer, again is regex. And this is where regex really shines.

The reason for this is we're looking for a specific pattern to parse in our text; not just digits. We actually want to ignore digits that don't match this pattern. Say, for instance, I gave you a time and my phone number:


```python
meeting_str = "Hey, give me a call at 8:30 on my cell at +1-555-555-0007."
```

If we try to only extract digits, we'll get a few extra we don't need. Take a look:


```python
pattern = r"\d+"
matches2 = re.findall(pattern, meeting_str)
matches2
```




    ['8', '30', '1', '555', '555', '0007']



So what we need to do is improve our regular expression pattern. Let's see how:


```python
phone_pattern = r"\+\d{1}-\d{3}-\d{3}-\d{4}"
matches3 = re.findall(phone_pattern, meeting_str)
matches3
```




    ['+1-555-555-0007']



Whoa. Now, you've really lost me. What the heck is `r"\+\d{1}-\d{3}-\d{3}-\d{4}"`?

To match any digit, you use the string `r"\d"`. The `r` in the front signifies this is a regular expression. The `\d` is the pattern to match any number digit. I'll explain the curly braces parts in a minute but let's dive into the `\d` a bit more.


```python
numbers_with_decimals = r"\d+\.\d+"
matches4 = re.findall(numbers_with_decimals, "123.122")
no_matches = re.findall(numbers_with_decimals, "12")
print(matches4, no_matches)
```

    ['123.122'] []


The last two patterns we saw something strange with the characters `+` and `.`. That's because regex treats these characters differently than English does. So if our regex pattern needs to use `+` or `.` we have to escape them with `\+` and `\.` respectively.

So going back to the pattern `r"\+\d{1}-\d{3}-\d{3}-\d{4}"`, let's break it into 4 chunks:

- Chunk 1. `\+\d{1}-`
- Chunk 2. `\d{3}-`
- Chunk 3. `\d{3}-`
- Chunk 4. `\d{4}`

These 4 chunks represent the international format for phone numbers in the US. This phone number isn't real but the format is.

What we're trying to do with regex is to create a pattern that encapsulates the general meaning behind the data we're using. Let's break down these 4 chunks.


##### Chunk 1 `\+\d{1}-`
This starts with `\+` which, in regex, converts to anything that starts with the character "+". If the string we were parsing was "Hey my number is 1-444-444-444" no match would be found. It *must* start with a `+` in this case.


The `\+` is proceeded by `\d` which, as you may recall, matches any and all digits; not letters, not spaces, not dashes, just digits.

`\d` is proceeded by `{1}`. Whenever you see braces (aka curly brackets) inside a regex pattern with a number, like `{1}` or `{3}` it means the previous pattern can only be `{n}` `n` length long. 

- `r"\d{4}"` a pattern for 4 digits
- `r"\d{4032}"` a pattern for 4032 digits
- `r"\d{2}"` a pattern for 2 digits

And finally, we see a `-` at the end of this chunk. In this case, that means this chunk ends with a dash (`-`). We don't need to escape the dash as we did with the `+` since regex doesn't use a dash `-` for any other special purpose. More on `+` later.


##### Chunk 2 `\d{3}-`

This should be pretty obvious now. This chunk matches any `\d` digit that is `{3}` 3 characters long with a trailing `-` dash.

##### Chunk 3 `\d{3}-`

Identical to chunk 2: This chunk matches any `\d` digit that is `{3}` 3 characters long with a trailing `-` dash.

##### Chunk 4 `\d{4}`

Almost identical to chunk 2 and 3 with 2 exceptions: This chunk matches any `\d` digit that is `{4}` 4 characters long and that's it.

Once you combine these 4 chunks you get a pattern that must start with (`\+`) a plus and (`\d{1}`) 1 digit and end with 4 digits (`\d{4}`).

Pretty cool huh? But how do we make this pattern optionally start with `+`?


```python
phone_pattern2 = r"\+?\d{1}-\d{3}-\d{3}-\d{4}"
area_code_only_number = "1-555-867-5309"
print(re.findall(phone_pattern2, area_code_only_number))

international_num = "+1-555-867-5309"
print(re.findall(phone_pattern2, international_num))
```

    ['1-555-867-5309']
    ['+1-555-867-5309']


You might notice that Chunk 1 started with `\+\d{1}-` and became `\+?\d{1}-`. The only difference is we added a `?` right after the `\+` which made the plus `+` optional. If we escaped `?` like `\+\?\d{1}-` then none of the patterns would have worked. Go ahead and try it for yourself.

### `?` makes things optional

Now, let's make the `-` optional as well since many people write their numbers like `+15558655309`.


```python
phone_pattern3 = r"\+?\d{1}-?\d{3}-?\d{3}-?\d{4}"

dashless = "+15558655309"
print(re.findall(phone_pattern3, dashless))

dashless_plusless = "15558655309"

print(re.findall(phone_pattern3, dashless_plusless))

some_dashes = "1555-8655309"

print(re.findall(phone_pattern3, some_dashes))

some_dashes_and_plus = "+1555-865-5309"

print(re.findall(phone_pattern3, some_dashes_and_plus))

```

    ['+15558655309']
    ['15558655309']
    ['1555-8655309']
    ['+1555-865-5309']


Now, we're talking. A much, much better extractor. How about numbers with parentheses `(555)-867-5309`? Yet another, ~fun~ way to write phone numbers.

Like the plus `+`, parentheses are special characters that regex uses. So we have to escape both open parentheses `\(` and close parentheses `\)`. Before we add the parentheses, let's make it easier to see how our chunks are working:


```python
chunk_1 = "\+?\d{1}-?"
chunk_2 = "\d{3}-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

phone_pattern4 = f"{chunk_1}{chunk_2}{chunk_3}{chunk_4}"

phone_pattern4_regex = re.compile(phone_pattern4)
phone_pattern4_regex
```




    re.compile(r'\+?\d{1}-?\d{3}-?\d{3}-?\d{4}', re.UNICODE)



Using `re.compile` allows us to use standard python string substitution to make our pattern. To use it, we just run:


```python
phone_pattern4_regex.findall(some_dashes_and_plus)
```




    ['+1555-865-5309']



> Note that when we use `re.compile` on a pattern (like our `phone_pattern_4`) we can change `re.findall(pattern, string)` to `<our-compiled-instance>.findall(string)` or `re.findall(<our-compiled-instance>, string)`

Now, let's adjust our chunks to account for parentheses in `chunk_2`:


```python
chunk_1 = "\+?\d{1}-?"
new_chunk_2 = "\(?" + "\d{3}" + "\)?" + "-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

phone_pattern5 = f"{chunk_1}{new_chunk_2}{chunk_3}{chunk_4}"

phone_pattern5_regex = re.compile(phone_pattern5)

some_parentheses = "+1-(555)-867-5309"

print(phone_pattern5_regex.findall(some_parentheses))

no_parentheses = "+1-555-867-5309"

print(phone_pattern5_regex.findall(no_parentheses))

```

    ['+1-(555)-867-5309']
    ['+1-555-867-5309']


We're really close to having a finished parser here. There's one other aspect we need to adjust. `chunk_1` is almost entirely optional. This is certainly true if you're already in the US. So let's fix that:


```python
new_chunk_1 = "\+?\d{0,1}-?"
new_chunk_2 = "\(?" + "\d{3}" + "\)?" + "-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

phone_pattern6 = f"{new_chunk_1}{new_chunk_2}{chunk_3}{chunk_4}"

phone_pattern6_regex = re.compile(phone_pattern6)

with_areacode_only = "(555)-867-5309"

print(phone_pattern6_regex.findall(with_areacode_only))

international_again = "+1-(555)-867-5309"

print(phone_pattern6_regex.findall(international_again))
```

    ['(555)-867-5309']
    ['+1-(555)-867-5309']


`new_chunk_1` shows us something cool about the length matching `{0,1}` we can use a length of 0 or 1 since that's the most accurate here. 

### Are we done then?

Well, no. The above works well but it's not accounting for various features of phone numbers. According to what [wikipedia](https://en.wikipedia.org/wiki/North_American_Numbering_Plan) says about North American phone numbers, when need to adjust `chunk_2` and `chunk_3` to only allow for:

- First digit between `[2-9]`
- Second and third digits between `[0-9]`

Luckily, this introduces a new feature of regular expressions for us. Remember the good old `\d` syntax? That captures all numbers... but what if we only want certain digits? Let's take a look:



```python
chunk_1 = "\+?\d{0,1}-?"
chunk_2 = "\(?" + "\d{3}" + "\)?" + "-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

chunk_2_new = "\(?" + "[2-9]{3}" + "\)?" + "-?"
chunk_3_new = "[0-9]{3}-?"

nanp_number_pattern = f"{chunk_1}{chunk_2_new}{chunk_3_new}{chunk_4}"

invalid_digits_format = "(155)-067-5309"

print("invalid_digits_format", re.compile(nanp_number_pattern).findall(invalid_digits_format))


valid_digits_format = "(215)-827-5309"

print("valid_digits_format", re.compile(nanp_number_pattern).findall(valid_digits_format))
```

    invalid_digits_format []
    valid_digits_format []


Now we can see, if we want to allow ony certain digits, we just pass a list `[]` with the digits we want (this is true for letters as well but more on that later).

Here's a few examples worth trying:

- Only even numbers: `[02468]`
- Only odd numbers: `[13579]`
- Only `1`, `4`, and `9`: `[149]`
- Only numbers between `4` and `8`: `[4-8]`

So `\d` is the shortcut of `[0-9]`. So `chunk_4 = "\d{4}"` can also be written as `chunk_4 = "[0-9]{4}"`.

Unfortunately, we need to update `chunk_2` and `chunk_3` a bit more. The first digit (not all 3) needs to be between 2-9 and the second 2 needs to be between 0-9. Let's see how that's done:


```python
chunk_1 = "\+?\d{0,1}-?"
chunk_2 = "\(?" + "\d{3}" + "\)?" + "-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

chunk_2_new = "\(?" + "[2-9]{1}" + "[0-9]{2}" + "\)?" + "-?"
chunk_3_new = "[2-9]{1}" + "[0-9]{2}" + "-?"


nanp_number_pattern2 = f"{chunk_1}{chunk_2_new}{chunk_3_new}{chunk_4}"

invalid_digits_format2 = "(108)-002-5309"

print("invalid_digits_format2", re.compile(nanp_number_pattern2).findall(invalid_digits_format2))


valid_digits_format2 = "(205)-203-5309"

print("valid_digits_format2", re.compile(nanp_number_pattern2).findall(valid_digits_format2))

```

    invalid_digits_format2 []
    valid_digits_format2 ['(205)-203-5309']


### The or `|` operator

Sometimes you need to allow for 2 different kinds of patterns in any given regex. Let's just keep on our phone pattern and allow `555` or `[4-9]`


Let's say I wanted to restrict the area code (ie `chunk_2`) to only match 2 different area codes. I'll do `212` and `213`


```python
chunk_1 = "\+?\d{0,1}-?"
chunk_2 = "\(?" + "\d{3}" + "\)?" + "-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"

chunk_2_new = "\(?" + "(?:213|212)" + "\)?" + "-?"
chunk_3_new = "[2-9]{1}" + "[0-9]{2}" + "-?"

area_code_matching_pattern = f"{chunk_1}{chunk_2_new}{chunk_3_new}{chunk_4}"

ny_city_number = "(212)-342-3223"

print("ny_city_number", re.compile(area_code_matching_pattern).findall(ny_city_number))

la_city_number = "213-323-1233"

print("la_city_number", re.compile(area_code_matching_pattern).findall(la_city_number))


chicago_city_number = "312-323-1233"

print("chicago_city_number", re.compile(area_code_matching_pattern).findall(chicago_city_number))

```

    ny_city_number ['(212)-342-3223']
    la_city_number ['213-323-1233']
    chicago_city_number []


The magic for this one is in `chunk_2_new` with the `(?:213|212)` portion. This combines two new concepts (1) a group and (2) the *or operator* `|`. We'll talk bout groups more later but for now, let's focus in on the `213|212` portion. That line `|` means or. So the patter will match `212` *or* `213`.

Can we make it a little more dynamic? The answer, like many things, is yes.


```python
dynamic_or_pattern = "(?:213|[3-9]{1}[2-9]{2})"

print("dynamic_or_pattern", re.compile(dynamic_or_pattern).findall("234"))

print("dynamic_or_pattern", re.compile(dynamic_or_pattern).findall("213"))

print("dynamic_or_pattern", re.compile(dynamic_or_pattern).findall("322"))
```

    dynamic_or_pattern []
    dynamic_or_pattern ['213']
    dynamic_or_pattern ['322']


As you can see, regular expressions can be pretty complex when we need them to be. The `dynamic_or_pattern` is starting to get us down a complicated rabbit hole that's a bit outside the context of this guide.

Let's bring it back up a bit.


## Groups

We regex, we can group parts of a pattern so they are easier to identify. A phone number is identified as:

```
<country-code>-<area-code>-<exchange-code>-<line-number>
```

This represents:

```
1-212-555-5123
```
- `1` is the country code
- `212` is the area code
- `555` is the exchange code
- `5123` is the line number


The technical terms for a phone number don't matter here. What matters is each chunk has a proper name that we want to assign to it. 

Before we assign names, let's turn our chunks into groups. I am going to use simple chunks to show you how:


```python
chunk_1 = "\d{1}-?"
chunk_2 = "\d{3}-?"
chunk_3 = "\d{3}-?"
chunk_4 = "\d{4}"


example = "1-212-555-5123"
pattern = f"{chunk_1}{chunk_2}{chunk_3}{chunk_4}"

print('example', re.compile(pattern).findall(example))
```

    example ['1-212-555-5123']



```python
group_1 = "(\d{1}-?)"
group_2 = "(\d{3}-?)"
group_3 = "(\d{3}-?)"
group_4 = "(\d{4})"


example = "1-212-555-5123"
grouped_pattern = f"{group_1}{group_2}{group_3}{group_4}"

matched = re.compile(grouped_pattern).match(example)
print('group', matched.group())
print('groups', matched.groups())
```

    group 1-212-555-5123
    groups ('1-', '212-', '555-', '5123')


Groups are simply chunks of regex with parentheses around them `()`. Of course, do not escape these parentheses otherwise it's not longer a group. Naturally, the entire pattern is also 1 big group by default as well and this is true with or without parentheses.

Now, I only want my groups to have digits (ie no dashes `-`). That is a simple fix. We just surround the parentheses `()` around the part of the pattern we want to extract most:


```python
group_1 = "(\d{1})-?"
group_2 = "(\d{3})-?"
group_3 = "(\d{3})-?"
group_4 = "(\d{4})"


example = "1-212-555-5123"
grouped_pattern = f"{group_1}{group_2}{group_3}{group_4}"

matched = re.compile(grouped_pattern).match(example)
print('group', matched.group())
print('groups', matched.groups())
```

    group 1-212-555-5123
    groups ('1', '212', '555', '5123')


Pretty neat huh? 

So, how do we access these groups by their type? One way is to select the group based on it's pattern index value + 1 (index 0 is the entire group). 

`group_1` is in index 0 of the entire pattern. The matched group will be in group 1 as you see here:


```python
group_1 = "(\d{1})-?"
group_2 = "(\d{3})-?"
group_3 = "(\d{3})-?"
group_4 = "(\d{4})"


example = "1-212-555-5123"
grouped_pattern = f"{group_1}{group_2}{group_3}{group_4}"

matched = re.compile(grouped_pattern).match(example)
country_code = matched.group(1)
print('country_code', country_code)

area_code = matched.group(2)
print('area_code', area_code)

exchange_code = matched.group(3)
print('exchange_code', exchange_code)

line_number = matched.group(4)
print('line_number', line_number)
```

    country_code 1
    area_code 212
    exchange_code 555
    line_number 5123


This method works perfectly fine but we might need something a bit more robust. Thus...

### Named Groups

Named groups allow you to add a keyword to each group in your regex expression. Let's take a look:


```python
named_group_1 = "(?P<country_code>\d{1})-?"
named_group_2 = "(?P<area_code>\d{3})-?"
named_group_3 = "(?P<exchange_code>\d{3})-?"
named_group_4 = "(?P<line_number>\d{4})"


example = "1-212-555-5123"
named_group_pattern = f"{named_group_1}{named_group_2}{named_group_3}{named_group_4}"

matched = re.compile(named_group_pattern).match(example)
print('named_groups', matched.groupdict())
# you can also use matched['country_code']
```

    named_groups {'country_code': '1', 'area_code': '212', 'exchange_code': '555', 'line_number': '5123'}



```python
long_example =  "Hi there, my home number is 1-555-867-5309 and my cell number is 1-555-555-0007."
all_matches = re.compile(named_group_pattern).finditer(long_example)
for i, m in enumerate(all_matches):
    print(f"Phone {i+1}", m.groupdict())
```

    Phone 1 {'country_code': '1', 'area_code': '555', 'exchange_code': '867', 'line_number': '5309'}
    Phone 2 {'country_code': '1', 'area_code': '555', 'exchange_code': '555', 'line_number': '0007'}


> What is `finditer`?  It's similar to `findall` but it allows us to see each match's `groupdict()`. `findall` does work but it strips the named group. Each iteration of `finditer` is an instance exactly like `re.match(pattern, example)`

## What about letters?

Everything we've done so far is using digits with `\d` or `[0-9]`. Letters is almost identical but, since letters can be capitalized, you can use `[a-z]` or `[A-Z]` in place of `[0-9]`.

Let's see an example:


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[a-z]"

print(re.findall(pattern, my_text))
```

    ['e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd', 'h', 'a', 'v', 'e', 'a', 's', 'c', 'o', 'r', 'e', 'o', 'f', 'o', 'w', 'c', 'o', 'o', 'l', 'i', 's', 't', 'h', 'a', 't']


As you can see, it only takes the lowercase letters; no spaces, no numbers, no punctuation.


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[A-Z]"

print(re.findall(pattern, my_text))
```

    ['H', 'I', 'H']


Now only uppercase letters.


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[a-zA-Z]"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd', 'I', 'h', 'a', 'v', 'e', 'a', 's', 'c', 'o', 'r', 'e', 'o', 'f', 'H', 'o', 'w', 'c', 'o', 'o', 'l', 'i', 's', 't', 'h', 'a', 't']


Now all letters


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[0-9a-zA-Z]"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd', 'I', 'h', 'a', 'v', 'e', 'a', 's', 'c', 'o', 'r', 'e', 'o', 'f', '1', '0', '0', '1', '0', '0', 'H', 'o', 'w', 'c', 'o', 'o', 'l', 'i', 's', 't', 'h', 'a', 't']


Now all letters and numbers


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[0-9a-zA-Z ]"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', ' ', 'I', ' ', 'h', 'a', 'v', 'e', ' ', 'a', ' ', 's', 'c', 'o', 'r', 'e', ' ', 'o', 'f', ' ', '1', '0', '0', '1', '0', '0', ' ', 'H', 'o', 'w', ' ', 'c', 'o', 'o', 'l', ' ', 'i', 's', ' ', 't', 'h', 'a', 't']


Now letters and spaces because I added a space to `r"[0-9a-zA-Z ]"`


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[0-9a-zA-Z \/\\]"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', ' ', 'I', ' ', 'h', 'a', 'v', 'e', ' ', 'a', ' ', 's', 'c', 'o', 'r', 'e', ' ', 'o', 'f', ' ', '1', '0', '0', '/', '1', '0', '0', ' ', 'H', 'o', 'w', ' ', 'c', 'o', 'o', 'l', ' ', 'i', 's', ' ', 't', 'h', 'a', 't']


Now some punctuation. With regex, I have to escape backslashes `\` with a backslash `\` resulting in `\\`. I also have to escape forward slash `/` with a backslash `\` resulting in `\/`.

Just like `\d`, I can use `\w` to accomplish something cool:


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"\w"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd', 'I', 'h', 'a', 'v', 'e', 'a', 's', 'c', 'o', 'r', 'e', 'o', 'f', '1', '0', '0', '1', '0', '0', 'H', 'o', 'w', 'c', 'o', 'o', 'l', 'i', 's', 't', 'h', 'a', 't']


`\w` is equivalent to `[a-zA-Z0-9]`


```python
my_text = "Hello world. I have a score of 100/100. How cool is that?"

pattern = r"[\w \/\\]"

print(re.findall(pattern, my_text))
```

    ['H', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', ' ', 'I', ' ', 'h', 'a', 'v', 'e', ' ', 'a', ' ', 's', 'c', 'o', 'r', 'e', ' ', 'o', 'f', ' ', '1', '0', '0', '/', '1', '0', '0', ' ', 'H', 'o', 'w', ' ', 'c', 'o', 'o', 'l', ' ', 'i', 's', ' ', 't', 'h', 'a', 't']


## Metacharacters

Here's a few metacharacters that you can use:

- `^` - the start of a string
- `[^0-9]` This matches everything except `[0-9]` because of `^`; `[^a-z]` matches anything that's not a lowercase number.
- `$` - the end of a string
- `+` - if 1 or more happens
- `*` - if 0 or more happens
- `?` - makes the value before `?` optional (as discussed above)
- `|` - the or operator (from above as well)



```python
first_letter_uppercase_match_pattern = "^[A-Z]"

print("uppercase_match", re.compile(first_letter_uppercase_match_pattern).match("Another"))

print("uppercase_not_match", re.compile(first_letter_uppercase_match_pattern).match("not Another"))
```

    uppercase_match <re.Match object; span=(0, 1), match='A'>
    uppercase_not_match None



```python
non_number_extraction_pattern = r"[^0-9]"

long_example =  "Hi there, my home number is 1-555-867-5309 and my cell number is 1-555-555-0007."

matched = re.findall(non_number_extraction_pattern, long_example)

print("non_number_extraction_pattern", "".join(matched))
```

    non_number_extraction_pattern Hi there, my home number is --- and my cell number is ---.



```python
ends_with_period_or_q_mark = r"(\.|\?)$"

this_fails = "This is not going to work!"

this_works = "This will work."

this_works_2 = "But will this?"


print(f"\"{this_fails}\"",  re.search(ends_with_period_or_q_mark, this_fails) != None)

print(f"\"{this_works}\"", re.search(ends_with_period_or_q_mark, this_works) != None)

print(f"\"{this_works_2}\"", re.search(ends_with_period_or_q_mark, this_works_2) != None)

```

    "This is not going to work!" False
    "This will work." True
    "But will this?" True


### Regex Substitution
Learn to use `re.sub` to dynamically replace values based on a matched pattern.


```python
group_1 = "(\d{1})-?"
group_2 = "(\d{3})-?"
group_3 = "(\d{3})-?"
group_4 = "(\d{4})"
grouped_pattern = f"{group_1}{group_2}{group_3}{group_4}"

sample = "Let's hide my phone numbers like 1-212-555-5123."
replacement = "***-***-****"
re.sub(grouped_pattern, replacement, sample)
```




    "Let's hide my phone numbers like ***-***-****."



#### Named Regex Substitution

To use a named group in a sub, you have to use the format `\g<named-group-var-name>` so `r="(?P<domain>[\w\.]+)"` becomes `\g<domain>`

Let's see:


```python
email_pattern = "(?P<username>\w+)@(?P<domain>[\w\.]+)"

re.sub(email_pattern, "****@\g<domain>", "j@gmail.com")
```




    '****@gmail.com'



#### Group Regex Substitution

Let's see the above example as an unnamed group, you have to use the format `\g<group_index>`. The possible `group_index` values for the pattern `r="(\w+).(\w+)"` is `\g<0>`, `\g<1>`, and `\g<2>`. `\g<0>` is for the entire pattern. `\g<1>` is for the first sub-group. 


Let's see:


```python
email_pattern = "(\w+)@([\w\.]+)"

re.sub(email_pattern, "****@\g<1>", "j@gmail.com")
```




    '****@j'



### Repeating patterns

`\w` matches all characters. Appending the `+` to it, will repeat that pattern until it's broken such as a space or punctuation.


```python
repeating_pattern = "\w+"
sentence = "This is going to work!"
re.findall(repeating_pattern, sentence)
```




    ['This', 'is', 'going', 'to', 'work']



Notice how the above pattern removes all spaces and punctuation. It's very practical. Compare this without the `+`:


```python
repeating_pattern = "\w"
sentence = "This is going to work!"
re.findall(repeating_pattern, sentence)
```




    ['T',
     'h',
     'i',
     's',
     'i',
     's',
     'g',
     'o',
     'i',
     'n',
     'g',
     't',
     'o',
     'w',
     'o',
     'r',
     'k']



Let's use the `*` now. 


```python
repeating_pattern = "\w*"
sentence = "This is going to work!"
re.findall(repeating_pattern, sentence)
```




    ['This', '', 'is', '', 'going', '', 'to', '', 'work', '', '']



So the `*` will return something even if there is no match. So `\w` matches all characters, adding `*` will still match all characters and return just `""` if no matches found.

Here's some additional reference:

- `\d`: Matches any decimal digit; this is equivalent to the class [0-9].
- `\D`: Matches any non-digit character; this is equivalent to the class [^0-9].
- `\s`: Matches any whitespace character; this is equivalent to the class [ \t\n\r\f\v].
- `\S`: Matches any non-whitespace character; this is equivalent to the class [^ \t\n\r\f\v].
- `\w`: Matches any alphanumeric character; this is equivalent to the class [a-zA-Z0-9_].
- `\W`:Matches any non-alphanumeric character; this is equivalent to the class [^a-zA-Z0-9_].

Naturally, the python docs for [regex](https://docs.python.org/3/howto/regex.html) are incredibly useful too.

[[ youtube id=TpOboJe6uIo ]]
