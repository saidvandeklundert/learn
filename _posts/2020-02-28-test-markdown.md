---
layout: post
title: Sample blog post
subtitle: Each post also has a subtitle
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test]
comments: true
---

This is a demo post to show you how to write blog posts with markdown.  I strongly encourage you to [take 5 minutes to learn how to write in markdown](https://markdowntutorial.com/) - it'll teach you how to transform regular text into bold/italics/headings/tables/etc.

```python
# Enable the main of this script to timeout and log:
def sig_handler(signum, frame):
    """
    Writes and error to the log and exits the program.
    """
    LOG.error(f"setup.py taking too long, terminated during stage {STAGE} at frame {frame}")
    sys.exit(1)
```

```rust
fn main() {
    //Scalar types
    let ch: char = 'z';
    let b: bool = true;
    let e: i32 = -2323;
    let float: f32 = 3.4;
    assert_eq!(ch, 'z');
    assert_eq!(b, true);
    assert_eq!(e, -2323);
    assert_eq!(float, 3.4);

    // Tuple:
    let tuple: (f32, f32) = (6.35, 15.123);
    assert_eq!(tuple.0, 6.35);
    assert_eq!(tuple.1, 15.123);
    // Array:
    let arr: [u8; 2] = [12, 233];
    assert_eq!(arr[1], 233);
    //Slice:
    let s = String::from("hello world");
    let hello = &s[0..5];
    let world = &s[6..11];
    assert_eq!(hello, "hello");
    assert_eq!(world, "world");
}
```

**Here is some bold text**

## Here is a secondary heading

Here's a useless table:

| Number | Next number | Previous number |
| :------ |:--- | :--- |
| Five | Six | Four |
| Ten | Eleven | Nine |
| Seven | Eight | Six |
| Two | Three | One |


How about a yummy crepe?

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg)

It can also be centered!

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg){: .mx-auto.d-block :}

Here's a code chunk:

~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

And here is the same code with syntax highlighting:

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code yet again but with line numbers:

{% highlight javascript linenos %}
var foo = function(x) {
  return(x + 5);
}
foo(3)
{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.
