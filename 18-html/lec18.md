# Data 6 Fall 2025: Web Scraping with Beautiful Soup

Based on lessons by [Melanie Walsh's _Intro to Cultural Analytics](https://melaniewalsh.github.io/Intro-Cultural-Analytics), [Alison Parrish](http://www.decontextualize.com/) and [Jinho Choi](https://github.com/emory-courses/data-science/blob/master/course/data_aggregation/data_aggregation.ipynb)

# Inspecting HTML's anatomy with Developer Tools

Thanks to Alison Parrish, we have a very simple example of HTML to begin with. It is about kittens. [Here's the rendered version](http://static.decontextualize.com/kittens.html), and [here's the HTML source code](https://raw.githubusercontent.com/ledeprogram/courses/master/databases/data/kittens.html).

## Developer Tools in your browser
First we're going to use Developer Tools in Chrome to take a look at how `kittens.html` is organized. Click on the "rendered ve#rsion" link above.
* Chrome, Firefox: Right-click (or ctrl-click) anywhere on the page, then click "Inspect"
* Safari: Right-click (or ctrl-click) anywhere on the page, then click "Inspect Element"

This will the browser's Chrome's Developer Tools. Your screen should look (something) like this:

<a href="http://static.decontextualize.com/snaps/kittens-dev-tools.png"><img src="http://static.decontextualize.com/snaps/kittens-dev-tools.png" alt="kittens-dev-tools"/></a>

In the upper panel, you see the web page you're inspecting. In the lower panel, you see a version of the HTML source code, with little arrows next to some of the lines. (The little arrows allow you to collapse parts of the HTML source that are hierarchically related.) As you move your mouse over the elements in the top panel, different parts of the source code will be highlighted. Chrome is showing you which parts of the source code are causing which parts of the page to show up. Pretty spiffy!

This relationship also works in reverse: you can move your mouse over some part of the source code in the lower panel, which will highlight in the top panel what that source code corresponds to on the page. We'll be using this later to visually identify the parts of the page that are interesting to us, so we can write code that extracts the contents of those parts automatically.


## The structure of `kittens.html`

Here's what the source code of kittens.html looks like:

```
<!doctype html>
<html>
  <head>
    <title>Kittens!</title>
  </head>
  <body>
    <h1>Kittens and the TV Shows They Love</h1>
    <div class="kitten">
      <h2>Fluffy</h2>
      <div><img src="http://placekitten.com/100/100"></div>
      <ul class="tvshows">
        <li><a href="http://www.imdb.com/title/tt0106145/">Deep Space Nine</a></li>
        <li><a href="http://www.imdb.com/title/tt0088576/">Mr. Belvedere</a></li>
      </ul>
      Last check-up: <span class="lastcheckup">2014-01-17</span>
    </div>
    <div class="kitten">
      <h2>Monsieur Whiskeurs</h2>
      <div><img src="http://placekitten.com/150/100"></div>
      <ul class="tvshows">
        <li><a href="http://www.imdb.com/title/tt0106179/">The X-Files</a></li>
        <li><a href="http://www.imdb.com/title/tt0098800/">Fresh Prince</a></li>
      </ul>
      Last check-up: <span class="lastcheckup">2013-11-02</span>
    </div>
  </body>
</html>
```

This is pretty well organized HTML, but if you don't know how to read HTML, it will still look like a big jumble. Here's how I would characterize the structure of this HTML, reading in my own idea of what the meaning of the elements are.

* We have two "kittens," both of which are contained in `<div>` tags with class `kitten`.
* Each "kitten" `<div>` has an `<h2>` tag with that kitten's name.
* There's an image for each kitten, specified with an `<img>` tag.
* Each kitten has a list (a `<ul>` with class `tvshows`) of television shows, contained within `<li>` tags.
* Those list items themselves have links (`<a>` tags) with an `href` attribute that contains a link to an IMDB entry for that show.

## Summary: HTML Elements

In summary, HTML is composed of **HTML elements**, which have:
* **Tags**, provided as the first string within angled brackets.
* (optional) **attributes**, provided as `attr=value` key-value pairs within the angled brackets.
* (optional) a string and/or other elements under the tag. These will be between the matching angled brackets, e.g., `<li>` and `</li>`.

You will hear **tag** and **element** used interchangeably. This is because every **HTML element must have a tag.**

### **TASK**: Discussion

1. What's the parent tag of `<a href="http://www.imdb.com/title/tt0088576/">Mr. Belvedere</a>`? 

_Your answer here_

2. Both `<div class="kitten">` tags share a parent tag---what is it? What attributes are present on both `<img>` tags?

_Your answer here_

# Scraping and Web requests

We've examined `kittens.html` a bit now. What we'd like to do is write some code that is going to extract information from the HTML, like "what is the last checkup date for each of these kittens?" or "what are Monsieur Whiskeur's favorite TV shows?" To do so, we need to:
1. **Scrape** the HTML from the internet
2. **Parse** the HTML by creating a representation of it in our program that we can manipulate with Python

Let's do the first task: **scraping**. It's left to us to actually *get* the HTML from somewhere. In most cases, we'll want to download the HTML directly from the actual web. For that, we'll use the `get` method from the Python library `requests` ([link](https://2.python-requests.org/en/master/)):


```python
# first let's import the requests library
import requests 
```

If it worked, you won't get an error message.

Now let's use the "get" method to make an http request (the eponymous "requests") to get the contents of kittens.html.


```python
resp = requests.get("http://static.decontextualize.com/kittens.html") 
resp
```

Note that "resp" is a **Python object**, and not plain text. The HTTP Response Status Code 200 means "OK"---as in, the webpage was able to give us the data we wanted.

_Side note_: The "get" method makes things easy by guessing at the document's character encoding. We're not really going to talk about character encoding until next class, but since we can, let's check this page's encoding real quick.


```python
resp.encoding
```

Okay, back on task. Now let's create a **string** with the contents of the web page in text format we use the "text" method for this.


```python
html_str = resp.text
html_str
```

That looks like a mess but it's apparent that we've obtained the data as desired.

If we wanted to "pretty print", note that the `html_str` has a lot of whitespace:
* `\t`: Tab
* `\n`: Newline

Calling `print` will render this whitespace in our display.


```python
# pretty print
print(html_str)
```

## Task: Why is web scraping hard?

It turns out that web scraping is heavily restricted on today's internet. What do you think are reasons why websites would **not** want programs periodically scraping their data?

_Your answer here_

In this class, we will usually handle the web requests and scraping for you. 

# The Beautiful Soup library

Now, onto the next step:

> 2. Parse the HTML by creating a representation of it in our program that we can manipulate with Python.

What representation should we choose? As mentioned earlier, HTML is hard to parse by hand. (Don't even try it. In particular, [don't parse HTML with regular expressions](http://stackoverflow.com/a/1732454).)

**[Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/)** is a Python library that parses (even poorly formatted) HTML and allows us to extract and manipulate its contents. More specifically, it gives us some Python objects that we can call methods on to poke at the data contained therein. So instead of working with strings and bytes, we can work with Python objects, methods and data structures.

Note that BeautifulSoup can parse any HTML that is provided as a string. We've already gotten an HTML string from our previous web request, so now we need to create a Beautiful Soup object from that data. 


```python
# just run this cell
from bs4 import BeautifulSoup

document = BeautifulSoup(html_str, "html.parser")
type(document)
```

The `BeautifulSoup` function creates a new Beautiful Soup object. It takes two parameters: the string containing the HTML data, and a string that designates which underlying parser to use to build the parsed version of the document. (Leave this as `"html.parser"`.) I've assigned this object to the variable `document`.

The `document` object supports a number of interesting methods that allow us to dig into the contents of the HTML. Primarily what we'll be working with are:
* `Tag` objects, and
* `ResultSet` objects, which are essentially just lists of `Tag` objects.

## Finding a tag with `find`

As we've previously discussed, HTML documents are composed of tags. To represent this, Beautiful Soup has a type of value that represents tags. We can use the `.find()` method of the `BeautifulSoup` object to find a tag that matches a particular tag name. For example:


```python
h1_tag = document.find('h1')
type(h1_tag)
```

A `Tag` object has several interesting attributes and methods. The `string` attribute of a `Tag` object, for example, returns a string representing that tag's contents:


```python
h1_tag.string
```

You can access the attributes of a tag by treating the tag object as though it were a **dictionary**. To get the **value** associated with a particular **attribute**. Use the square-bracket syntax, providing the attribute name as key/string.

### Get the `src` attribute of the first `<img>` tag in the document


```python
img_tag = document.find('img')
img_tag['src']
```

**Note**: You might have noticed that there is more than one `<img>` tag in `kittens.html`! If more than one tag matches the name you pass to `.find()`, it returns only the first matching tag. (A better name for `.find()` might be `.find_first()`, but we digress.)

### **TASK**: Find the last check-up date of the first kitten.


```python
# YOUR SOLUTION HERE
first_span_tag = ...
...
```

## Finding multiple tags with `find_all`

It's very often the case that we want to find not just one tag that matches particular criteria, but ALL tags matching those criteria. For that, we use the `.find_all()` method of the `BeautifulSoup` object. For example, to find all `h2` tags in the document:


```python
h2_tags = document.find_all('h2')
type(h2_tags)
```

But what's in the Result Set?

In order to find out, we're going to need to use a loop. Specifically, a **for** loop.


```python
for tag in h2_tags:
    print(tag.string) 
```

This conveniently gives you a variable, `tag`, that updates with the appropriate value each time you iterate.

## Finding tags with particular attributes

In any case, both the `.find()` and `.find_all()` methods can search not just for tags with particular names, but also for **tags that have particular attributes**.

For that, we use the **optional** `attrs` keyword argument. `attrs` is a dictionary that associates attribute names as keys and the desired attribute value as values.

```
document.find_all(tag_name, attrs=None)
```

To find all `span` tags with a `class` attribute of `lastcheckup`:


```python
checkup_tags = document.find_all('span', attrs={'class': 'lastcheckup'})
for tag in checkup_tags:
    print(tag.string)
```

## Read more in the BeautifulSoup Documentation

**Before we move on:**

Beautiful Soup's `.find()` and `.find_all()` methods are actually more powerful than we're letting on here. [Check out the details in the official Beautiful Soup documentation.](http://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all)

## Use `find`/`find_all` on tags, too

Let's say that we wanted to print out a list of the name of each kitten, along with a list of the names of that kitten's favorite TV shows. In other words, we want to print out something that looks like this:

    Flufzfy: Deep Space Nine, Mr. Belvedere
    Monsieur Whiskeurs: The X-Files, Fresh Prince
    
In order to do this, we need to find *not just* tags with particular names, but tags with **particular hierarchical relationships** with other tags:

1. Identify all of the kittens, then
1. Find the shows that belong to that kitten.

This kind of search is made easy by the fact that you can use `.find()` and `.find_all()` methods not just on the entire document, but on **individual tags***. When you use these methods on tags, they search for matching tags that are specifically **children of** the tag that you call them on.

---

In our kittens example, we can see that information about individual kittens is grouped together under `<div>` tags with a `class` attribute of `kitten`. So, to find a list of all `<div>` tags with `class` set to `kitten`, we might do this:


```python
# just run this cell
kitten_tags = document.find_all("div", attrs={"class": "kitten"})
# kitten_tags # uncomment if you want to print
```

Now, we'll loop over that list of tags and find, inside each of them, the `<h2>` tag that is its child:



```python
# just run this cell
for kitten_tag in kitten_tags:
    h2_tag = kitten_tag.find('h2')
    print(h2_tag.string)
```

### String method demo practice

Now, we'll go one extra step. Looping over all of the kitten tags, we'll find not just the `<h2>` tag with the kitten's name, but all `<a>` tags (which contain the names of the TV shows that we were looking for).

_Note*: The list method `lst.append(elt)` returns nothing and directly modifies the original list `lst`, appending `elt` to the end. (This behavior is unlike `np.append`, which returns a new array.)


```python
# trace through this code carefully!

for kitten_tag in kitten_tags:
    h2_tag = kitten_tag.find('h2')
    
    a_tag_strings = []
    for tag in kitten_tag.find_all('a'):
        a_tag_strings.append(tag.string)
        
    print(h2_tag.string + ":", ", ".join(a_tag_strings))
```

### Task: Find last check-up for all kittens

Using the code above as a template, write code that prints out a list of kitten names along with the last check-up date for that kitten.

HINT: Look up above for the code that deals with check-up dates. 


```python
# YOUR CODE HERE
```

<details>
  <summary>Click for Solution</summary>

Various options. One reasonable approach:

```
for kitten_tag in kitten_tags:
    h2_tag = kitten_tag.find('h2') # keep this to find the kittens
    checkup_tag = kitten_tag.find('span', attrs={'class': 'lastcheckup'})

    print(h2_tag.string + ", " + checkup_tag.string)
```

</details>



### Task: Find links to all kittens' favorite shows

Using the code above, write code that prints a list of kitten names with links to that kitten's favorite shows. I.e., you should end up with something of the format:

`Name: [kitten name]
 URLS: www.asdasdsa.com, www.sdkalskdjsa.com`
 
Hint: Look up above for how you access the attribute of a tag. 


```python
# YOUR CODE HERE
```

<details>
  <summary>Click for Solution</summary>

Various options. One reasonable approach:

```
for kitten_tag in kitten_tags:
    h2_tag = kitten_tag.find('h2') # keep this to find the kittens
    url_strings = []
    for url in kitten_tag.find_all('a'):
        url_strings.append(url['href'])
    urls_joined = ", ".join(url_strings)
    print("Name: " + h2_tag.string)
    print("URLs: " + urls_joined)
```

</details>



## Find sibling tags: `find_next_sibling()`

Often, the tags we're looking for don't have a distinguishing characteristic, like a `class` attribute, that allows us to find them using `.find()` and `.find_all()`, and the tags also aren't in a parent-child relationship. This can be tricky! Take the following HTML snippet, for example:    


```python
cheese_html = """
<h2>Camembert</h2>
<p>A soft cheese made in the Camembert region of France.</p>

<h2>Cheddar</h2>
<p>A yellow cheese made in the Cheddar region of... France, probably, idk whatevs.</p>
"""

cheese_document = BeautifulSoup(cheese_html, "html.parser")
```

If our task was to create a list of the name of the cheese followed by the description that follows in the `<p>` tag directly afterward, we'd be out of luck. Fortunately, Beautiful Soup has a `.find_next_sibling()` method, which allows us to search for the next tag that is a *sibling* of the tag you're calling it on (i.e., the two tags share a parent), that also matches particular criteria. So, for example, to accomplish the task outlined above:


```python
cheese_dict = {}
for h2_tag in cheese_document.find_all('h2'):
    cheese_name = h2_tag.string
    cheese_desc_tag = h2_tag.find_next_sibling('p')
    cheese_dict[cheese_name] = cheese_desc_tag.string

cheese_dict
```

You now know most of what you need to know to scrape web pages effectively. Good job!

But before you're done, we should talk about what to do when things go wrong.

## When things go wrong with Beautiful Soup

A number of things might go wrong with Beautiful Soup. Here are just a few.

### Tag does not exist with `find`

You might, for example, search for a tag that doesn't exist in the document:


```python
footer_tag = document.find("footer")
```

Beautiful Soup doesn't return an error if it can't find the tag you want. Instead, it returns `None`:


```python
type(footer_tag)
```

If you try to call a method on the object that Beautiful Soup returned anyway, you might end up with an error like this:


```python
footer_tag.find("p")
```

You might also inadvertently try to get an attribute of a tag that wasn't actually found. You'll get a similar error in that case:


```python
footer_tag['title']
```

Whenever you see something like `TypeError: 'NoneType' object is not subscriptable`, it's a good idea to check to see whether your method calls are indeed finding the thing you were looking for.

### No tags exist with `find_all`

The `.find_all()` method will return an empty list if it doesn't find any of the tags you wanted:


```python
footer_tags = document.find_all("footer")
print(footer_tags)
```

If you attempt to access one of the elements of this regardless...


```python
print(footer_tags[0].string)
```

...you'll get an `IndexError`.

# [Out-of-scope material] List comprehension

Consider the following code:


```python
checkup_tags = document.find_all('span', attrs={'class': 'lastcheckup'})
[tag.string for tag in checkup_tags]
```



The second line in the code cell above is a helpful shorthand: it creates a list with each of the `tag.string`s in `checkup_tags`.

In more official terms, it's called a *list comprehension*, and it helps with a very common task in both data analysis and computer programming: when you want to apply an operation to every item in a list (e.g., scaling the numbers in a list by a fixed factor), or create a copy of a list with only those items that match a particular criterion (e.g., eliminating values that fall below a certain threshold). 

A list comprehension has a few parts:

* a source list, or the list whose values will be transformed or filtered;
* a predicate expression, to be evaluated for every item in the list;
* (optionally) a membership expression that determines whether or not an item in the source list will be included in the result of evaluating the list comprehension, based on whether the expression evaluates to True or False; and
* a temporary variable name by which each value from the source list will be known in the predicate expression and membership expression.
These parts are arranged like so:

> `[` *predicate expression* `for` *temporary variable name* `in` *source list* `if` *membership expression* `]`

The words for, in, and if are a part of the syntax of the expression. They don't mean anything in particular (and in fact, they do completely different things in other parts of the Python language). You just have to spell them right and put them in the right place in order for the list comprehension to work.

You are welcome to use list comprehension in this course, but we won't test it.

# Further reading

* [Chapter 11](https://automatetheboringstuff.com/chapter11/) from Al Sweigart's [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) is another good take on this material (and discusses a wider range of techniques).
* [The official Beautiful Soup documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) provides a systematic walkthrough of the library's functionality. If you find yourself thinking, "it really should be easy to do the thing that I want to do, why isn't it easier?" then check the documentation! Leonard's probably already thought of a way to make it easier and implemented a feature in the code to help you out.
* Beautiful Soup is the best scraping library out there for quick jobs, but if you have a larger site that you need to scrape, you might look into [Scrapy](http://scrapy.org/), which bundles a good parser with a framework for writing web "spiders" (i.e., programs that parse web pages and follow the links found there, in order to make a catalog of an entire web site, not just a single web page).
