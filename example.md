Title
Short description follows
28/12/2021 20:27
29/12/2021
example, tags, blog post
-----
This page is a demonstration of the elements that can be formatted using Simple.css. Each section includes a code block on how to include it in your site’s design.

This may be a little basic for some people, but I wanted to barrier for entry to be as low as possible for this project.

## Basic Typography

All the typography of Simple.css uses `rem` for sizing. This means that accessibility is maintained for those who change their browser font size. The `body` element has a size of `1.15rem` which makes all the standard font sizes slightly larger. This equates to `18.4px` for paragraph text, instead of the standard `16px`.

The heading elements also have an increased top margin in order to break blocks of text up better.

#Heading 1 `2.8rem`
##Heading 2 `2.25rem`
###Heading 3 `1.8rem`
####Heading 4 `1.44rem`
#####Heading 5 `1.15rem`
######Heading 6 `.92rem`

```
<h2>This is a H2 header<h2>

<p>This is some paragraph text.</p>
```

## Links

Links are formatted very simply on Simple.css (shock horror). They use the accent CSS variable and are underlined. There is a :hover effect that removes the underline. Here is an [example link](https://example.com "Alt text").

## Other typography elements

There are a number of other typography elements that you can use with Simple.css. Some of the common ones are:

 - All the standard stuff, like **bold**, *italic* and ***both***.
 - Adding `inline code` using the `code` element.

# Images

In Simple.css, images within the `main` element are always full width and have rounded edges to them. The `figcaption` element is also formatted in Simple.css. Here are examples of images with and without a caption:

![Dog with a tablet](https://simplecss.org/assets/images/dog-ipad.jpg)

![A black swan](https://simplecss.org/assets/images/goose.jpg)
