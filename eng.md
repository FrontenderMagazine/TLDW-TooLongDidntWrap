# TLDW:TooLongDidntWrap

![Illustration][Illustration]

Another day, another CSS rant article from a JavaScript snob. :) I’ll keep this
one brief, though.

This time I’m talking about word-wrapping behavior. Specifically, trying to get
CSS to force text to fit within the container box, even if it has to wrap at
unnatural (non-word boundary) locations. This is sorta the reverse task of that
examined in my other article, [“Ellipse My Text…”][1].

Example:

<p style="font-family: sans-serif; font-size: 15px; padding:10px; background-color: yellow; width: 200px;">http://www.urbandictionary.com/define.php?term=the%20meaning%20of%20life</p>

The browser breaks the long URL as best as it can, but it’s not good enough.

## Manual Text Break

One hack’ish option is to force an element into your long text at certain
intervals, to give the browser more of a hint/chance at breaking the text.

You can use JavaScript to take a piece of text, say a tweet or news item, and
every X number of characters, you insert your hint element. There’s some
possible complications to consider, though:

1. You likely only want to break up long words (that is, you don’t need to 
insert something that ends up breaking a two character word in half), so you 
don’t just want to insert dumbly at each “multiple of X” character position. 
You’d probably want to say something like, find any string of non-space 
characters longer than say 10 characters, and insert there.A JS regex that you 
might use for that might be: `text.split(/(\s*\S{10}\s*)/g).join("...")` where 
`"..."` would be your break element.
2. If you have more than just text, such as html markup, you have to be more 
careful not to insert your break elements inside of the html tag portions, as 
you could break tag or attribute names, invalidating them, so you’d probably 
need to first loop over the return values from `split(..)` and look at the 
surrounding context to decide whether you should insert in that position or not 
(that kind of code is left as an exercise for the reader).

But, if we decide to go this route, which element should we insert?

### (space)

A space may seem like an obvious choice, but it’s no good because if you have a
space where a break isn’t actually happening, the space will show up visibly,
which you probably don’t want. And definitely don’t use `&nbsp;`, because that’s
literally a “non-breaking space”, which is the opposite of what you want.

You could insert a `<span class='break'> </span>` element, and style it to have
no visible width.
 
    <style>
    p { font-family:sans-serif; font-size:15px; padding:3px; background-color:yellow; width:85px; }
    .break { font-size:0px; }
    </style>
 
    <p>Now is the ti<span> </span>me for all good men...</p>

<p style="font-family: sans-serif; font-size: 15px; padding: 3px; background-color: yellow; width: 85px;">Now is the time for all good men…</p>

But that’s awfully ugly, and you may have some issues in older/legacy browsers
([if you even care][2] about them!).

### Opportunity comes knocking

How about the [“Word Break Opportunity”][3] tag `<wbr>`? Seems perfect. It’s a
no-visible-render element that helps the browser know where it’s ok to break if
it wouldn’t have otherwise decided to break there.

One issue you may notice is that it doesn’t necessarily get the browser to
insert a hyphen where it does the breaking in the middle of a word. You may or
may not want that.

Another problem some years back was that Safari did not support this `<wbr>`
element. Safari however did have very early support for the `&shy;` element
(that other browsers back then did not support, but many more do now), which is
a “soft hyphen” used in [suggesting line-break opportunities][4] to the browser.
If the browser breaks at a `&shy;`, it should show the `-` hyphen, and if not,
it remains invisible. Just what we’d like.

Because of legacy support issues, doing this used to require you to browser
sniff and choose either `<wbr>` or `&shy;` accordingly. Nowadays, the support
seems much better, and you can probably just choose which one you want depending
on if you want a hyphen or not. For instance, a hyphen showing in a line-broken
URL is probably not what you want, but hyphens in regular text are much nicer
for readers, especially those accustomed to print-layout type displays (online
newspapers, magazines, etc).

## CSS rescues us!

Manually breaking text used to be our only option. But thankfully, CSS actually
already has a great answer for us on this, so if supporting the older non-CSS3
browsers isn’t as important to you, you can let CSS do the work for you.

The `word-wrap:break-word` functionality will do what we want, that it will
allow the browser to [break in the middle of a word][5] where it feels like is
best to do so.

    <style>
    #container { word-wrap:break-word; background-color:yellow; padding:10px; width:200px; }
    </style>
 
    <div id="container">
   http://www.urbandictionary.com/define.php?term=the%20meaning%20of%20life
    </div>

<div style="word-wrap: break-word; background-color: yellow; padding: 10px; width: 200px;">http://www.urbandictionary.com/define.php?term=the%20meaning%20of%20life</div>

**NOTE:** This feature strictly has been around awhile, but it’s recently been 
renamed in the CSS3 spec to `overflow-wrap:break-word`, and only the newest 
browsers are supporting the rename. So, tread lightly using it, and probably 
specify both rules for awhile.

Another [related feature][6] is `word-break`. Semantically, we’d probably have
preferred for `word-break` to handle the word-breaking we’re discussing, instead
of `word-wrap` or `overflow-wrap`, but… oh well. In any case, `word-break`
appears to control word-breaking in internationalized languages, so it may or
may not be something to consider.

## What do you think?

Did CSS get it right? The slight semantic hiccup between `word-break` and `word-
wrap` isn’t too big of a deal. But we still don’t seem to have as much control
over the hyphenation like we did with `&shy;`.

Do we need more in the CSS standard, or should we just be happy we’ve got this
much? Let us know in the discussion thread below!

[1]: http://html5hub.com/ellipse-my-text/#i.17vvxi5jknd0vx
[2]: http://html5hub.com/browser-versions-are-dead/#i.xadptq2lifhdr1
[3]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/wbr
[4]: https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens#Suggesting_line_break_opportunities
[5]: https://developer.mozilla.org/en-US/docs/Web/CSS/word-wrap
[6]: https://developer.mozilla.org/en-US/docs/Web/CSS/word-break

[Illustration]: img/Screen-Shot-2014-04-10-at-3.58.13-PM-660x380.png