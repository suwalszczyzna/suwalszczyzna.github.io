---
layout: post
title: "CSS - Responsive typography"
date: 2021-09-07 18:00:00 +0100
categories: frontend css
---
Responsive web design is the most important thing in web development nowadays. The basic thing on frontend is typography. Which units do you use in you projects? `Px`, `cm`, `mm`, `rem` or `em`? How about `vw`?


This blog post is about rem and em. The most responsive units in css, virtually they was created especially for responsive designs.


- [REM and EM - what's the difference?](#rem-and-em---whats-the-difference)
- [Dealing with media queries](#dealing-with-media-queries)
      - [Create css variables.](#create-css-variables)
      - [Change html font-size](#change-html-font-size)
- [VW unit and clamp() combo](#vw-unit-and-clamp-combo)
- [What is clamp function?](#what-is-clamp-function)
- [Codepen example](#codepen-example)


# REM and EM - what's the difference?

Difference is simple:

-- **rem** is relative to font-size of the **ROOT** element on page. Literally, it is 'html' tag.

For example:

You set `font-size` on your html element to `15px`. Now when you will use `rem` it will depend on this 15px.
* 1 rem = 15px
* 2 rem = 30px etc.


```css
html {
    font-size: 15px;
}

p {
    font-size: 1rem; // 15px
    line-height: 1.6rem; // 24px   <- You can use rem/em to assign width and height too and it still depend on font-size of html element! :)
}

h1 {
    font-size: 2rem; // 30px
}
```

-- **em** is relative to font-size of a **parent** element. I think it is clear

```css
.card {
    font-size: 13px;
}

p {
    font-size: 1rem; // 13px
    line-height: 1.5rem; // 19,5px
}

h1 {
    font-size: 2rem; // 26px
}
```

# Dealing with media queries
Responsive units won't help if we abuse media queries. It's really easy to make too much of them. But what selector should we change? 


There is couple option:

#### Create css variables.

```css
:root {
  --font-xl: 3rem; // for headings
  --font-l: 1.8rem; // for leads
  --font-m: 1.3rem; // for paragaphs
}

@media(min-width: 60rem){
  :root{
    --font-l: 2.5rem;
    --font-m: 1.8rem;
  }
}
```
As you see above in `min-width` we used rem too. 60rem in this case mean 960px. I haven't assigned any value of font-size in html element so its default value is 16px. `60 * 16px = 960px`. Voila! :) 


#### Change html font-size

This way can generate some problems because it will change all your responsive sizes in all projects. You have to use it intentionally.


```css
@media(min-width: 60rem){
  html {
    font-size: 20px;
  }
}


p {
  font-size: 2rem; 
  // 32px if viewport with is under 960px. Above them is 40px;
}
```


> Be careful - if you use relative units in media query definition and change html font-size don't set font-size in html element by yourself (outside media query)

# VW unit and clamp() combo
VW - viewport width unit is kinda tricky. If you use it normally, e.g:
```css
p {
  font-size: 15vw;
}
```
it brings you many problems. On huge monitors, your font will be enormous big but on mobile will be tiny. It is useless used alone. 

But if we aid to this operation `clamp` function we'll be able to apply limitations of growing and shrink.

# What is clamp function? 

It's simple. Clamp function accepts minimal, actual and maximum value: `clamp(MIN, VAL, MAX)`


Examples:
* `clamp(1rem, 2.5vw, 2rem);` - it will change between 1 and 2 rem. Not more and less.
* `clamp(1.5rem, 2.5vw, 4rem);`
* `clamp(3.5rem, 10vw + 1rem, 8rem);`

# Codepen example
I have prepared codepen to play with all of today seen tricks

<p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="css,result" data-slug-hash="abwpowm" data-preview="true" data-user="damiansuwala" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/damiansuwala/pen/abwpowm">
  Responsive typography</a> by damiansuwala (<a href="https://codepen.io/damiansuwala">@damiansuwala</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>