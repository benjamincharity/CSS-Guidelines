# CSS Guidlines

---

**Purpose: Maintainability, scalability, ease of code hand-off.**

*Expect this document to change over time as my understanding deepens and new
methods present themselves.*

---


## Contents

* [CSS document anatomy](#css-document-anatomy)
  * [General](#general)
  * [One file vs. many files](#one-file-vs-many-files)
  * [Table of contents](#table-of-contents)
* [Source order](#source-order)
* [Anatomy of rulesets](#anatomy-of-rulesets)
* [Naming conventions](#naming-conventions)
  * [Classes for CSS and JS shared rules](#classes-for-css-and-js-shared-rules)
  * [JS hooks](#js-hooks)
  * [Internationalization](#internationalization)
* [Comments](#comments)
  * [Comments on steroids](#comments-on-steroids)
    * [Quasi-qualified selectors](#quasi-qualified-selectors)
    * [Object/extension pointers](#objectextension-pointers)
* [Writing CSS](#writing-css)
* [Building new components](#building-new-components)
* [OOCSS](#oocss)
* [Layout](#layout)
* [Font sizing](#font-sizing)
  * [H tags](#h-tags)
* [Shorthand](#shorthand)
* [IDs](#ids)
* [Selectors](#selectors)
  * [Over qualified selectors](#over-qualified-selectors)
  * [Selector performance](#selector-performance)
* [CSS selector intent](#css-selector-intent)
* [`!important`](#important)
* [Shame.scss](#shamescss)
* [Magic numbers and absolutes](#magic-numbers-and-absolutes)
* [Conditional stylesheets](#conditional-stylesheets)
* [Debugging](#debugging)
* [Credits](#credits)

---


## CSS Document Anatomy

No matter the document, we must always try and keep a common formatting. This
means consistent commenting, consistent syntax and consistent naming.


### General

CSS should be written in
[Expanded Form](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#id15):

    .widget {
      rule: value;
    }

SCSS is our prefered CSS syntax.


### One file vs. many files

Our files should be organized by modules.  Much of this was inspired by
[Clean out your Sass junk-drawer](http://gist.io/4436524) from Dale Sande.



    layouts/
      _main.scss
      ...
    home/
      _extends.scss
      _home.scss
      _welcome.scss
    extends/
      _buttons.scss
      _forms.scss
      _links.scss
      ...
    global/
      _extends.scss
      _navbar.scss
      _navpanel.scss
    _dev.scss
    _global_config.scss
    _global_functions.scss
    _global_reset.scss
    _shame.scss
    main.scss


### Files that should be found in every project

Some global SCSS files should appear in every project.

    _dev.scss              // contains any styling useful during development
    _global_reset.scss     // level the browser playing field and start with a blank canvas
    _global_functions.scss // level the browser playing field and start with a blank canvas
    _global_config.scss    // set all site-wide variables and third-party overrides
    _global_functions.scss // create reusable functions here to separate function
                              from style
    _shame.scss


### Table of contents

This file should reference each included SCSS file with direct links to open the
file in vim and a brief description:

    // SIGN-UP
    // app/sign-up.scss
    // Define the styles for the embeddable sign up widget.
    app/sign-up.scss

    // MODAL
    // app/modal.scss
    // Define all global styles for modal windows
    app/modal.scss

    // etc...

All links must be relative to the current directory. This allows Vim users to
simply place their cursor over the path and type 'gf' to open it.


### File titles and headers

Without being able to search for files, the table of contents wouldn't be nearly
as useful. At the top of each SCSS file reference the name as you defined it in
the table of contents.

We will also include several other pieces of helpful information: a) primary
contributors (name/email), and editor settings.


    // ex: set tabstop=8 expandtab:
    //
    //
    // $RESET
    //
    //
    // @author Your Name <you@youremail.com>
    // @doc
    //  Any documentation needed goes here...
    // @end



The `$` prefixing the name of the section allows us to run a find
for `$[SECTION-NAME]` and **limit our search scope to section titles only**.


## Source order

All stylesheets should be in order of specificity. This ensures that you take full
advantage of inheritance and CSS' first <i>C</i>; the cascade.

A well ordered set of stylesheets will look something like this:

1. **Elements** – unclassed `h1`, unclassed `ul` etc.
2. **Objects and abstractions** — generic, underlying design patterns.
3. **Components** – full components constructed from objects and their extensions.
4. **Style trumps** – error states etc.

This means that—as you go down the document—each section builds upon and
inherits sensibly from the previous one(s). There should be less undoing of
styles, less specificity problems and all-round better architected stylesheets.


## Anatomy of rulesets

    [selector]{
      [property]: [value];
    }


* Use hyphen delimited class names (except for BEM notation,
  [see below](#naming-conventions))
* Soft tabs with a 2 space indent
* 1 space after the colon `:` following the property
* Multi-line
* Declarations in alphabetical order
* Align vendor-prefixed rules so their values are aligned.
* Always include the final semi-colon in a ruleset
* `0` values should not include the unit (px/em/etc)

A brief example:

    .widget{
      background-color: #c0ffee;
      border: 1px solid #bada55;
      -webkit-border-radius : 4px;
      -moz-border-radius    : 4px;
      -ms-border-radius     : 4px;
      -o-border-radius      : 4px;
      border-radius         : 4px;
      padding: 10px;
    }

    .widget__heading{
      color: #bada55;
      font-size: 16px;
      font-size: 1.5rem;
      font-weight: bold;
      line-height: 1em;
    }

Here we can see that `.widget-heading` must be a child of `.widget` as we have
indented the `.widget__heading` ruleset one level deeper than `.widget`. This is
useful information to developers that can now be gleaned just by a glance at the
indentation of our rulesets.

One exception to our multi-line rule might be in cases of the following:

    .t10 { width:10% }
    .t20 { width:20% }
    .t25 { width:25% }
    .t30 { width:30% }
    .t33 { width:33.333% }
    .t40 { width:40% }
    .t50 { width:50% }
    .t60 { width:60% }
    .t66 { width:66.666% }
    .t70 { width:70% }
    .t75 { width:75% }
    .t80 { width:80% }
    .t90 { width:90% }

In this example from [inuit.css's table grid system](https://github.com/csswizardry/inuit.css/blob/master/base/_tables.scss#L96-L120)
it makes more sense to single-line our CSS.


## Naming conventions

Always use hyphen delimited classes (e.g. `.foo-bar`, not
`.foo_bar` or `.fooBar`), however in certain circumstances we can use BEM (Block,
Element, Modifier) notation.

[<abbr title="Block, Element, Modifier">BEM</abbr>](https://bem.info/method/)
is a methodology for naming and classifying CSS selectors in a way to make them
much more strict, transparent and informative.

The naming convention follows this pattern:

    .block {...}
    .block__element {...}
    .block--modifier {...}

* `.block` represents the higher level of an abstraction or component.
* `.block__element` represents a descendent of `.block` that helps form `.block`
  as a whole.
* `.block--modifier` represents a different state or version of `.block`.

An analogy of how BEM classes work might be:

    .person {...}

    .person--woman {...}

    .person__hand {...}
    .person__hand--left {...}
    .person__hand--right {...}

Here we can see that the basic object we’re describing is a person, and that a
different type of person might be a woman. We can also see that people have
hands; these are sub-parts of people, and there are different variations,
like left and right.

We can now namespace our selectors based on their base objects and we can also
communicate what job the selector does; is it a sub-component (`__`) or a
variation (`--`)?

So, `.page-wrapper` is a standalone selector; it doesn't form part of an
abstraction or a component and as such it named correctly. `.widget__heading`,
however, _is_ related to a component; it is a child of the `.widget` construct
so we would rename this class `.widget__heading`.

BEM looks a little uglier, and is a lot more verbose, but it grants us a lot of
power in that we can glean the functions and relationships of elements from
their classes alone. Also, BEM syntax will typically compress (gzip) very well
as compression favors/works well with repetition.

Regardless of whether you need to use BEM or not, always ensure classes are
sensibly named; keep them as short as possible but as long as necessary. Ensure
any objects or abstractions are very vaguely named (e.g. `.ui-list`, `.media`)
to allow for greater reuse. Extensions of objects should be much more explicitly
named (e.g. `.user-avatar-link`). **Don't worry about the amount or length of
classes;** gzip will compress well written code _incredibly_ well.

Let me say that one more time....

**Do NOT worry about the amount or length of classes.**

### Classes in HTML

In a bid to make things easier to read, separate classes is your HTML with two
(2) spaces, thus:

    <div class="foo--bar  bar__baz">

This increased whitespace should allow for easier spotting and reading
of multiple classes.


### Classes for CSS and JS shared rules

Use the `is_` prefix for CSS state rules that are triggered by JavaScript.

HTML:

    <button class="button--submit  is_enabled">Submit</button>

CSS:

    .button--submit {
      opacity: .7;
      pointer-events: none;

      &.is_enabled {
        opacity: 1;
        pointer-events: all;
      }
    }


### JS hooks

**Never use a CSS _styling_ class as a JavaScript hook.** Attaching JS behaviour
to a styling class means that we can never have one without the other.

If you need to bind to some markup use a JS specific CSS class. This is simply a
class namespaced with `.js-`, e.g. `.js-toggle`, `.js-drag-and-drop`. This means
that we can attach both JS and CSS to classes in our markup but there will never
be any troublesome overlap.

    <button class="is_active  js-active-button">
      Submit
    </button>

The above markup holds two classes; one to which you can attach some styling for
sortable table columns and another which allows you to add the sorting
functionality.


### Internationalization

Whether you grew up using <i>colour</i> instead of <i>color</i>—for 
the sake of consistency, it is better to always use US-English in CSS. CSS, as 
with most (if not all) other languages, is written in US-English, so to mix
syntax like `color:red;` with classes like `.colour-picker{}` lacks consistency.


## Comments

When more than one or two lines are needed, use a docBlock-esque commenting
style limited to 80 characters in length:

    //
    //
    //  This is a docBlock style comment
    //
    //  This is a longer description of the comment, describing the code in more
    //  detail. We limit these lines to a maximum of 80 characters in length.
    //
    //  We can have markup in the comments, and are encouraged to do so:
    //
    //  <div class=foo>
    //      <p>Lorem</p>
    //  </div>
    //
    //  We do not prefix lines of code with an asterisk as to do so would inhibit
    //  copy and paste.
    //
    //

Document and comment your code as much as you possibly can, what may
seem or feel transparent and self explanatory to you may not be to another dev.
**Write a chunk of code then write about it.**


### Comments on steroids

There are a number of more advanced techniques you can employ with regards
comments, namely:

* Quasi-qualified selectors
* Object/extension pointers


#### Quasi-qualified selectors

Never qualify your selectors; that is to say, never write
`ul.nav{}` if you can just use `.nav`. Qualifying selectors a) decreases selector
performance, b) inhibits the potential for reusing a class on a different type of
element and c) it increases the selector's specificity. These are all things that
should be avoided at all costs.

However, sometimes it is useful to communicate to the next developer(s) where
you intend a class to be used. Let’s take `.product-page` for example; this
class sounds as though it would be used on a high-level container, perhaps the
`html` or `body` element, but with `.product-page` alone it is impossible to
tell.

By quasi-qualifying this selector, adding the element that the class should be
applied to, we can make it easier to find within the DOM.

    // <html>
    .product-page {...}

We can now see exactly where to apply this class but with none of the
specificity or non-reusability drawbacks.

Other examples might be:

    // <ol>
    .breadcrumb {...}
    // <p>
    .intro {...}
    // <ul>
    .image-thumbs {...}

Here we can see where we intend each of these classes to be applied without
impacting the specificity of the selectors.


#### Object/extension pointers

When working in an object oriented manner you will often have two chunks of CSS
(one being the skeleton (the object) and the other being the skin (the
extension)) that are very closely related, but that live in different places.

The object should be defined as a [placeholder selector](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_).
This allows the style to be defined, but it will not be rendered on it's own;
but rather as part of the style declaration that includes it.

    %error {
      background-color: red;
      border: 1px solid DarkRed;
      color: white;
    }

Then to use:

    .error--serious {
      @extend %error;
      border-width: 3px;
    }

Here we have established a concrete relationship between two very separate
pieces of code.

---


## Writing CSS

The previous section dealt with how we structure and form our CSS; they were
very quantifiable rules. The next section is a little more theoretical and deals
with our attitude and approach.


### Building new components

When building a new component write markup **before** CSS. This means you can
visually see which CSS properties are naturally inherited and thus avoid
reapplying redundant styles.

By writing markup first you can focus on data, content and semantics and then
apply only the relevant classes and CSS _afterwards_.


### OOCSS

Work in an OOCSS manner; split components into structure (objects) and
skin (extensions). As an **analogy** (note, not example) take the following:

    .room {...}

    .room--kitchen {...}
    .room--bedroom {...}
    .room--bathroom {...}

We have several types of room in a house, but they all share similar traits;
they all have floors, ceilings, walls and doors. We can share this information
in an abstracted `.room{}` class. However we have specific types of room that
are different from the others; a kitchen might have a tiled floor and a bedroom
might have carpets, a bathroom might not have a window but a bedroom most likely
will, each room likely has different coloured walls. OOCSS teaches us to
abstract the shared styles out into a base object and then _extend_ this
information with more specific classes to add the unique treatment(s).

So, instead of building dozens of unique components, try and spot repeated
design patterns across them all and abstract them out into reusable classes;
build these skeletons as base ‘objects’ and then peg classes onto these to
extend their styling for more unique circumstances.

If you have to build a new component split it into structure and skin; build the
structure of the component using very generic classes so that we can reuse that
construct and then use more specific classes to skin it up and add design
treatments.


### Layout

All components you build should be left **totally free** of widths; they should
always remain fluid and their widths should be governed by a parent/grid system.

Rather than use a grid system that requires extra markup or an abundance of
classes, we create our grid using custom mixins applied to various containers.

In `grid.scss` define columns and gutters as so:

    //
    // Base font size = 16px // Defined in base.scss
    // Site = 1000px, Column = 150px, Gutter = 20px   (6 column grid)
    //

    $column: 15%;
    $gutter: 2%;

    $onecol   : $column;
    $twocol   : ($column * 2) + $gutter;
    $threecol : ($column * 3) + ($gutter * 2);
    $fourcol  : ($column * 4) + ($gutter * 3);
    $fivecol  : ($column * 5) + ($gutter * 4);
    $sixcol   : ($column * 6) + ($gutter * 5);

Leave any notes above the grid in a comment block. For instance, a new developer
may not know how the 1000px wide design file translates to a 15% column. When
sizing in ems or rems it is also helpful to include the base font size here.


### Font sizing

We can use a combination of methods for sizing UIs. Percentages, pixels, ems, rems
and nothing at all.

Grid systems should, ideally, be set in percentages. Because we use grid systems
to govern widths of columns and pages, you can leave components totally free of
any dimensions (as discussed above).

Font sizes should be set in rems with a pixel fallback. This gives the accessibility
benefits of ems with the confidence of pixels. Here is a handy Sass mixin to
work out a rem and pixel fallback for you (assuming you set your base font
size in a variable somewhere):

    @mixin font-size($font-size){
      font-size: $font-size +px;
      font-size: $font-size / $base-font-size +rem;
    }

Only use pixels for items whose dimensions were defined _before_ the came into
the site. This includes things like images and sprites whose dimensions are
inherently set absolutely in pixels.

**NOTE:** As browsers have become increasingly better at determining correct px
sizes when zooming, I have noticed myself needing REMs less and less.

### H tags

If you find yourself needing to reuse header styles across your site, attach
your styles to something _other_ than the element.

Rather than simply using .hN notation we use abstract classes made up of the first 
six letters of the Greek alphabet:

    h1, .alpha   { ...}
    h2, .beta    { ...}
    h3, .gamma   { ...}
    h4, .delta   { ...}
    h5, .epsilon { ...}
    h6, .zeta    { ...}

This allows us to reuse the styles without _always_ being locked into specific 
elements.


## Shorthand

**Shorthand CSS needs to be used with caution.**

It might be tempting to use declarations like `background: red;` but in doing so
what you are actually saying is ‘I want no image to scroll, aligned top-left,
repeating X and Y, and a background color of red’. Nine times out of ten this
won't cause any issues but that one time it does is annoying enough to warrant
not using such shorthand. Instead use `background-color:red;`.

Similarly, declarations like `margin: 0;` are nice and short, but
**be explicit**. If you actually only really want to affect the margin on
the bottom of an element then it is more appropriate to use `margin-bottom: 0;`.

Be explicit in which properties you set and take care to not inadvertently unset
others with shorthand. E.g. if you only want to remove the bottom margin on an
element then there is no sense in setting all margins to zero with `margin: 0;`.

Shorthand can be good, but is most often abused.


## IDs

A quick note on IDs in CSS before we dive into selectors in general.

**NEVER use IDs in CSS.**

They can be used in your markup for JS and fragment identifiers but use only
classes for styling. There shouldn't be a single ID in any stylesheets!

Classes come with the benefit of being reusable (even if we don't want to, we
can) and they have a nice, low specificity. Specificity is one of the quickest
ways to run into difficulties in projects and keeping it low at all times is
_imperative_. An ID is **255** times more specific than a class, so never ever use
them in CSS _ever_.

If CSS specificity is still a bit confusing, check out this visualizer:
[Specificity Calculator](http://specificity.keegan.st/)


## Selectors

Keep selectors a) short, b) efficient and c) portable.

Heavily location-based selectors are bad for a number of reasons. For example,
take `.sidebar h3 span{}`. This selector is too location-based and thus we
cannot move that `span` outside of a `h3` outside of `.sidebar` and maintain
styling.

Selectors which are too long also introduce performance issues; the more checks
in a selector (e.g. `.sidebar h3 span` has three checks, `.content ul p a` has
four), the more work the browser has to do.

Make sure styles are not dependent on location, and make sure selectors are nice 
and short.

Selectors as a whole should be kept short (e.g. one - two class(es) deep) but the class
names themselves should be as long as they need to be. A class of `.user-avatar`
is much easier to quickly read and comprehend than `.usr-avt`.

**Remember:** classes are neither semantic or insemantic; they are sensible or
insensible! Stop stressing about ‘semantic’ class names and pick something
sensible and futureproof.


### Over-qualified selectors

As discussed above, qualified selectors are bad news.

An over-qualified selector is one like `div.promo`. You could probably get the
same effect from just using `.promo`. Of course sometimes you will _want_ to
qualify a class with an element (e.g. if you have a generic `.error` class that
needs to look different when applied to different elements e.g.:

    .error {
      color: red;
    }
    div.error {
      padding: 14px;
    }

But generally avoid it where possible.

Another example of an over-qualified selector might be `ul.nav li a {...}`. As
above, we can instantly drop the `ul` and because we know `.nav` is a list, we
therefore know that any `a` _must_ be in an `li`, so we can get `ul.nav li a
{...}` down to just `.nav a {...}`.


### Selector performance

Whilst it is true that browsers will only ever keep getting faster at rendering
CSS, efficiency is something you could do to keep an eye on. Short, unnested
selectors, not using the universal (`* {}`) selector as the key selector, and
avoiding more complex CSS3 selectors should help circumvent these problems.


## CSS selector intent

Instead of using selectors to drill down the DOM to an element, it is often best
to put a class on the element you explicitly want to style. Let's take a
specific example with a selector like `.header ul {...}`…

Let's imagine that `ul` is indeed the main navigation for our website. It lives
in the header as you might expect and is currently the only `ul` in there;
`.header ul {...}` will work, but it's not ideal or advisable. It's not very future
proof and certainly not explicit enough. As soon as we add another `ul` to that
header it will adopt the styling of our main nav and the chances are it
won't want to. This means we either have to refactor a lot of code _or_ undo a
lot of styling on subsequent `ul`'s in that `.header` to remove the effects of
the far reaching selector.

Your selector's intent must match that of your reason for styling something;
ask yourself **‘am I selecting this because it’s a `ul` inside of `.header` or
because it is my site’s main nav?’**. The answer to this will determine your
selector.

Make sure your key selector is never an element/type selector or
object/abstraction class. You never really want to see selectors like
`.sidebar ul {...}` or `.footer .media {...}` in our theme stylesheets.

Be explicit; target the element you want to affect, not its parent. **Never**
assume that markup won't change. **Write selectors that target what you want,
not what happens to be there already.**

For a full write up please see the article
[Shoot to kill; CSS selector intent](http://csswizardry.com/2012/07/shoot-to-kill-css-selector-intent/)


## `!important`

Unless needed to override third party styles outside of our control, you should
never use `!important`.

Anytime a 'hack' such as this is employed, it should be included in your
`_shame.scss` (outlined below).

Always leave one space between the style value and the `!important` declaration
to improve readability.

    .foo {
      background-color: #bada55 !important;
    }


## Shame SCSS

The idea of shame.css is that you have a totally new stylesheet reserved just
for your hacky code. The code you have to write to get the release out on time,
but the code that makes you ashamed.

By putting your hacks and quick-fixes in their own file you do a few things:

1. You make them stick out like a sore thumb.
2. You keep your ‘main’ codebase clean.
3. You make developers aware that their hacks are made very visible.
4. You make them easier to isolate and fix.
5. $ git blame _shame.scss.


### The Rules

Obviously you need some kind of rules and criteria:

- If it's a hack, it goes in _shame.scss.
- Document all hacks fully:
    - What part of the codebase does it relate to?
    - Why was this needed?
    - How does this fix it?
    - How might you fix it properly, given more time?
- Do not blame the developer; if they explained why they had to do it then
  their reasons are probably (hopefully) valid.
- Try and clean _shame.scss up when you have some down time.
    - Even better, get a tech-debt story in which you can dedicate actual
      sprint time to it.

Original article by [CSS-Wizardry](http://twitter.com/csswizardry):
[shame.css](http://csswizardry.com/2013/04/shame-css/)


## Magic numbers and absolutes

A magic number is a number which is used because ‘it just works’. These are bad
because they rarely work for any real reason and are not usually very
futureproof or flexible/forgiving. They tend to fix symptoms and not problems.

For example, using `.dropdown-nav li:hover ul { top: 37px; }` to move a dropdown
to the bottom of the nav on hover is bad, as `37px` is a magic number. `37px` only
works here because in this particular scenario the `.dropdown-nav` happens to be
`37px` tall.

Instead you should use `.dropdown-nav li:hover ul { top: 100%; }` which means no
matter how tall the `.dropdown-nav` gets, the dropdown will always sit 100% from
the top.

_Every_ time you hard code a number think twice (maybe even a third time); if
you can avoid it by using keywords or ‘aliases’ (i.e. `top: w100%` to mean ‘all
the way from the top’) or&mdash;even better&mdash;no measurements at all then
you probably should.

Every hard-coded measurement you set is a commitment you (or other developers on
the project) might not necessarily want to keep.


## Conditional stylesheets

IE stylesheets can, by and large, be totally avoided. The only time an IE
stylesheet may be required is to circumvent blatant lack of support (e.g. PNG
fixes).

As a general rule, all layout and box-model rules can and _will_ work without an
IE stylesheet if you refactor and rework your CSS. This means you never want to
see `<!--[if IE 7]> element{ margin-left:-9px; } < ![endif]-->` or other such
CSS that is clearly using arbitrary styling to just ‘make stuff work’.

Use Modernizr.js to test for and target different support sets rather than 
device or software sniffing.


## Debugging

If you run into a CSS problem **take code away before you start adding more** in
a bid to fix it.

Remember: _The problem exists in CSS that is already written, more CSS isn't
the right answer!_

Delete chunks of markup and CSS until your problem goes away, then you can
determine which part of the code the problem lies in.

It can be tempting to put an `overflow: hidden;` on something to hide the effects
of a layout quirk, but overflow was probably never the problem; **fix the
problem, not its symptoms.**


## Overrides & Fallbacks

Use [Modernizr.js](http://modernizr.com/) to test for features when needed. Use
the classes provided by Modernizr to set fallbacks or overrides for specific
feature sets.

Place these styles at the end of the related stylesheet. For example, if you are
editing a styleheet called `_home.scss` and you need to write a specific style
for touch enabled devices, place the override at the bottom of `_home.scss`

Or, better yet, include a separate file in your `home/` scss directory called
`_overrides.scss` and include your specific overrides there. Then in your main
SCSS file, include this overrides file after the primary `_home.scss`.


## CSS Resets

We use a custom reset file that was originally based off of [Eric Meyer's original
css reset](http://meyerweb.com/eric/tools/css/reset/) and then tweaked over the 
years.

This file is maintained on Github: [Resets](https://github.com/benjamincharity/Resets)


## Credits

Credit goes out to these guys as the inspiration and source of knowledge for
much of this document.

- Harry Roberts - [http://twitter.com/csswizardry](https://twitter.com/csswizardry)
- Chris Coyier - [http://twitter.com/real_css_tricks](http://twitter.com/csswizardry)
- Kyle Kneath - [http://twitter.com/kneath](http://twitter.com/kneath)
- Dale Sande - [http://twitter.com/anotheruiguy](http://twitter.com/anotheruiguy)
