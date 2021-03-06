---
title: The open function explained
author: yasoob
type: post
date: 2014-01-15T13:19:30+00:00
url: /2014/01/15/the-open-function-explained/
publicize_facebook_url:
  - https://facebook.com/509724922449953_10203066813301172
publicize_twitter_user:
  - yasoobkhalid
publicize_twitter_url:
  - http://t.co/zAT4L1EhKF
categories:
  - python
tags:
  - image python
  - jpeg python
  - open
  - opening a file in python
  - python image
  - python open image file
  - with statement

---

This is a guest post by Philipp.

[open][1] opens a file. Pretty simple, eh? Most of the time, we see it being used like this:

```
f = open('photo.jpg', 'r+')
jpgdata = f.read()
f.close()
```

The reason I am writing this article is that most of the time, I see open used like this: There are three errors in the above code. Can you spot them all? If not, read on. By the end of this article, you'll know what's wrong in the above code, and, more importantly, be able to avoid these mistakes in your own code.

Let's start with the basics: The return of open is a file handle, given out from the operating system to your Python application. You will want to return this file handle once you're finished with the file, if only so that your application won't reach the limit of the number of open file handle it can have at once.

Explicitly calling `close` closes the file handle, but only if the read was successful. If there is any error just after `f = open(...)`, `f.close()` will not be called (depending on the Python interpreter, the file handle may still be returned, but that's another story). To make sure that the file gets closed whether an exception occurs or not, pack it into a [`with`][2] statement:

```
with open('photo.jpg', 'r+') as f:
    jpgdata = f.read()
```

The first argument of `open` is the filename. The second one (the \*mode\*) determines \*how\* the file gets opened.

- If you want to read the file, pass in `r`
- If you want to read and write the file, pass in `r+`
- If you want to overwrite the file, pass in `w`
- If you want to append to the file, pass in `a`

While there are a couple of other valid mode strings, chances are you won't ever use them. The mode matters not only because it changes the behavior, but also because it may result in permission errors. For example, if we were to open a jpg-file in a write-protected directory, `open(.., 'r+')` would fail.

The mode can contain one further character; we can open the file in binary (you'll get a string of bytes) or text mode (a string of characters). In general, if the format is written by humans, it tends to be text mode. jpg image files are not generally written by humans (and are indeed not readable to humans), and you should therefore open them in binary mode by adding a `b` to the text string (if you're following the opening example, the correct mode would be `rb`).

If you open something in text mode (i.e. add a `t`, or nothing apart from r/r+/w/a), you must also know which encoding to use - for a computer, all files are just bytes, not characters. Unfortunately, `open` does not allow explicit encoding specification in Python 2.x. However, the function [`io.open`][3] is available in both Python 2.x and 3.x (where it is an alias of `open`), and does the right thing.

You can pass in the encoding with the `encoding` keyword. If you don't pass in any encoding, a system- (and Python-) specific default will be picked. You may be tempted to rely on these defaults, but the defaults are often wrong, or the default encoding cannot actually express all characters (this will happen on Python 2.x and/or Windows). So go ahead and pick an encoding. `'utf-8'` is a terrific one.

When you write a file, you can just pick the encoding to your liking (or the liking of the program that will eventually read your file). How do you find out which encoding a file you read has? Well, unfortunately, there is no sureproof way to detect the encoding - the same bytes can represent different, but equally valid characters in different encodings. Therefore, you must rely on metadata (for example, in HTTP headers) to know the encoding. Increasingly, formats just define the encoding to be UTF-8.

Armed with this knowledge, let's write a program that reads a file, determines whether it's JPG (hint: These files start with the bytes `FF D8`), and writes a text file that describe the input file.

```
import io
 
with open('photo.jpg', 'rb') as inf:
    jpgdata = inf.read()
 
if jpgdata.startswith(b'\xff\xd8'):
    text = u'This is a jpeg file (%d bytes long)\n'
else:
    text = u'This is a random file (%d bytes long)\n'
 
with io.open('summary.txt', 'w', encoding='utf-8') as outf:
    outf.write(text % len(jpgdata))
```

 [1]: http://docs.python.org/dev/library/functions.html#open
 [2]: http://freepythontips.wordpress.com/2013/07/28/the-with-statement/
 [3]: http://docs.python.org/2/library/io.html#io.open