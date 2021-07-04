---
layout: post
title: "Next.js - basics"
date: 2021-07-04 18:00:00 +0100
categories: react
---

[Next.js](https://nextjs.org/) is framework based on ReactJS. It simplifies many blotware of React code and give some features (e.g. server-side-rendering) out-of-the-box

# Next.js instalation

`npx create-next-app name-your-app` - new Next.js app

`npm run dev` - build dev app and run dev server with it

# Routing and pages
Routing is declare with file tree in `page/` direction

## Static URLs

- `page/index.js` - http://your.app/
- `page/admin/index.js` - http://your.app/admin/

## Dynamic URLs
- `page/admin/[slug].js` - http://your.app/admin/blablabla/
- `page/[username]/index.js` - http://your.app/blablabla/ 
- `page/[username]/[slug].js` - http://your.app/blablabla/ohohoh


> Static URLs has priority then Dynamic. In this example is not possible to open this page `page/[username]/index.js` using url: `your.app/admin`. In our static route with this pattern already exist so *admin/index.js* has priority


## Page component structure
Example file: `page/admin/[slug].js`

It's simple react component, without import react module

```javascript
export default function AdminPostEditPage({}){
    return(
        <main>
            <h1>Post edit page</h1>
        </main>
    )
}
```

## Links
Example file: `page/index.js`

```
import Link from 'next/link';

export default function Home() {
  return(
      <>
        <Link
        href={{
          pathname: '/[username]',
          query: {username: 'daemon666'},
        }}>
          <a>Daemon's profile</a>
        </Link>
      </>
  )
}
```

![Next.js link]({{ BASE_PATH }}/assets/post_images/next_routing.png "Next.js link")