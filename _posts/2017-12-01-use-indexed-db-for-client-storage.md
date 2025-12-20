---
title: Use IndexedDB (instead of Local Storage) for Client Side Storage
categories:
- DevOps
- Containers
tags:
- local storage
- IndexedDB
- browser storage
- Dexie
- cookie
- client side storage
description: Save data on the client side with IndexedDB
date: 2017-12-01 19:20
author_profile: true
classes: wide
---

# Use Case

I was working on a data dashboard, there are a number of filters, including rules (is or is not), keywords, and date range etc, user can change these filters to set up a view that they will use on daily basis, that means the filter set up should be saved somewhere. The legacy code used local storage, but it has some limitations, one is that every time user update the filter you have to use `JSON.parse()` to parse the string, and if you want to change the schema of the data, you have to write code to sort of "cache busting" the local storage.

# Local Storage VS IndexedDB

(The actual stats [are different for various browsers](https://www.html5rocks.com/en/tutorials/offline/quota-research/), this is just an overview)

|           | LocalStorage           | IndexedDB  |
| --------- |:-------------:| -----:|
|  Storage Size | Up to 10MB | update to 50MB |
|  Storage Format | **strings** only of key value | built on transactional database model, you can save complex structured data as value |
|  Block UI | synchronous |  mostly asynchronous |
|  API | simple, such as setItem, getItem | more complicated |

To know more about core concepts of IndexedDB see [here](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Concepts_Behind_IndexedDB).

What I want to achieve are 1) easy to update the storage, ideally when user changes the filter, the criteria is saved 2) easy to update the schema as the requirement for client storage evolves 3) easy to expand later if there are more filter with different type.

It seems that IndexedDB is a better option in my case.


# Implementation (wrapper library and redux)

However, if you look at the native api of the IndexedDB [here](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB), it is overwhelming, it feels like you are writing XHR from scratch by hand, no one does that now. Thanks to Open Source, there are a few wrapper libraries for IndexedDB, I used [Dexie](http://dexie.org/) because the api looks intuitive and well maintained.

Scenario one is to save these dashboard filters and update them, this is how I implemented it:

In `db.js`, I initialize the db like this:

```js
const db = new Dexie('CriteriaDB'); // settings for items page

db.version(1).stores({
  filters: "++id, name, rule, options",
});

db.on("populate", () => {
  // populate will only be executed once
  // add filter options, bulk add or add separately
  // db.filters.bulkAdd(filterOptions)

  db.filters.add({
    id: 1,
    name: 'a',
    rule: 'is',
    options: [1,2,3,4]
  })
})

export default db;
```

To update the data in the storage, I used the Dexie api, which is similar to database query, something like this:

```js
db.filters.update(action.name, {
  options: options
})

// or get
db.filters.get(action.name)
```

Scenario two is to update the schema of the storage, as mentioned [here](http://dexie.org/docs/Tutorial/Design#database-versioning) and [here](http://dexie.org/docs/Version/Version.upgrade()):

```js
db.version(2).stores({
  filters: null, // change it to null to remove this table in new version
  newStuff: "++id, name" // add a new table
}).upgrade(t => {
  // do stuff here, update, add, remove
  t.newStuff.add({
    id:1,
    name: 'Some name'
  })
})
```

When the page refreshs, you should be able to see the IndexedDB version is updated. One thing bugs me a bit is the chrome dev tool has some kind of lagging issue, sometimes you have to close and reopen the dev tool to see changes.
