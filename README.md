# pathname
Use pathname as API similar to window.URLSearchParams

# Problem

When creating a complex SPA, usually you do lots of internal linking. 

You want to keep your links organized. Maybe even across back-end and front-end? Especially, if your apps features URL-persisted filters, you'll cry when trying to rename a search param.

After some time, your developers will start to create helpers and other one-off abstractions making the situation even worse.

# Ideas

The solution is to build a dictionary of URLs.

It will work like this:

```javascript
export const URLS = {
  ROOT: '/',
  CUSTOMERS: '/customers',
  CUSTOMER: '/customer/:id'
  // and so on.  
};
```

Sweet, but does not handle query params or path params.

The real API you want will work like this:

```javascript
export const URLS = {
  ROOT: createURL('/'),
  CUSTOMERS: createURL('/customers'),
  CUSTOMER: createURL('/customer/:id') // maybe add some types for query params
  // and so on.  
};
```

Tip: you could provide URLS via your bundler so that you don't have to import them. Back to the business:

```javascript
<a href={URLS.CUSTOMER({id: customer.id})}>Customer details</a>
```

in real life:

```javascript
<a href={URLS.CUSTOMER({
    pathParams: {
        id: customer.id
    },
    queryParams: {
        someSidebar: true
    },
    hashParams: CUSTOMER_REGIONS.DETAILS // build it in a way that you can pass string too! Unless URL gets long, you'd store params in queryParams
})}>Customers</a>

// => /customer/5?someSidebar=true#details
```

Note that in the above example it'd be sweet to convert booleans to 1 or 0 to save precious bytes.

You could then create URL parser based on qs to map your state to URL, or map your URL to state.

# Implementation

I would like to write such library, and add proper testing to it.

But we've got window.URL and window.URLSearchParams. Also, we've got awesome qs or query-string to facilitate the hardest operation. 
It looks like we've got all the stuff to duct tape the perfect URL management library.

The only thing we're missing is the pathname part. I want to build simple library to facilitate this step.

## Why pathname?

It feels native.

```javascript
new URL('http://korytnicki.pl/s/z/y/m/o/n').pathname
```

# API

```javascript
import createPathname from 'pathname'; // or require it 
```

**createPathname**

Creates a pathname you can manipulate.
```javascript
const pathname = createPathname('/customer/:id', {id: 2});
```

```javascript
const pathname = createPathname('/customer/:id');
```

returns pathname object.

**pathname.toString**

Creates string you can use. Optionally, of course, pass options object. Only one param is accepted now (trailingSpace, defaults to true).


```javascript
createPathname('/customer/:id').toString(); // /customer
createPathname('/customer/:id').toString({trailingSpace: true}); // /customer/
createPathname('/customer/:id', {id: 2}).toString(); // /customer/2
createPathname('/customer/:id', {id: 2}).toString({trailingSpace: true}); // /customer/2/
createPathname('/customer/:id/', {id: 2}).toString({trailingSpace: true}); // /customer/2//
```

Note it's toString like you'd expect it. 

```javascript
const customerURL = createPathname('/customer/:id', {id: 2}) + '?' + new URLSearchParams({hasPopup: true});
// /customer/id/2?hasPopup=true
```

**pathname.set**

sets params. Accepts objects or key and value.

```javascript
createPathname('/customer/:id').set('id', 2);
createPathname('/customer/:id')
    .set('id', 2)
    .set('id', 5);
// customer/5
createPathname('/customer/:id').set({id: 2});
// customer/2
createPathname('/customer/:id', {id: 5})
    .set({id: 2})
    .toString(); // /customer/id/2
```

**pathname.keys**

returns array of keys

**pathname.values**

returns array of values

**pathname.entries**

returns array of entries

Just play with it.
Consider copy-pasting it as a helper in your `/helpers` directory.