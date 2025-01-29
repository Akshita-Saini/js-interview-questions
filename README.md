# js-interview-questions
A compilation of JS interview questions and notes


# Web Storage APIs

## Why do we need Browser Storage?
1. Improve User Experience
2. Better performance
3. Offine support
4. We can store:
   a. User generated content
   b. App's state
   c. Cached assets
   d. Auth tokens
   e. Analytics

## How does Browser Storage work?
1. Using JS we're going to store and retrieve data stored locally in user's device.
2. Browsers manage the implementation and security details.
3. We should treat it as data that can disappear anytime.
4. The data will persist between browsing sessions.
5. On most APIs, we won't require explicit permissions from the user.
6. It also works for PWAs and hybrid apps.
7. Data is NOT shared with the server or other webapps.(cookies are the only exception)

## Some Important Concepts
1. What is the origin?
   **Protocol + Host + Port**
   Browser storage is origin-based.

## APIs for Browser Data Storage
1. Cookies - not suitable
2. Web storage: local storage and session storage - synchronous APIs (block the thread) - becoming an ANTI-PATTERN - something that should be avoided
3. WebSQL - depricated
4. Application cache - depricated
5. **IndexedDB** - object based (NoSql db) for client-side. Typically stores JS object. Can also store bytes.
6. **FileSystem Access** - Subset: Origin Private FS - Not fully compatible for every browser - based on Chromium based browsers - Chrome, Edge, Opera, Brave etc. - Open and work with the real file system. Will require user permission.
7. Files and directories - to be deprecated (files system created for origin which is not the real user file system - also Chrome only)
8. **Cache Storage** - part of the service worker spec - storing HTTP responses

## Data Storage APIs Comparison
1. **IndexedDB**: stores JS objects and binary data - a **keyPath** within the object - keyPath is the name of a property within your object that you want to use as a key. Data is grouped in **Object Stores** and object stores are grouped in **Databases**. Stored upto the available quota.
2. **Cache storage**: HTTP responses, could be PDFs, CSS/JS files etc - key is HTTP request. Grouped in **Caches**. Stored upto the available quota.
3. **Web storage**: Strings - data has to be stringified to be stored here - key is string. Store keys in root level. There is no grouping. **Session storage: 12MB Local storage: 5MB**
4. **FileSystem access**: Files, no key, only file path. Store at root level. There is no grouping.

**Quota: You could probably store upto 1GB per origin without any issue.**

It's possible to make your own DB - using web assembly and Indexed DB or FileSystem API. 

## Web Storage
1. Simple API
   a. It stores only one string per key.
   b. The key is also a string.
2. Synchronous API
   a. Perf issues
   b. Not available on workers or service workers: meaning you can't read or write to it from a sw or w.
3. We should try to avoid using it today.
4. You can emulate it's behaviour using IndexedDB.
5. It offers the same API for local and session storage.
   
```
localStorage.setItem("key", "value");
const data = localStorage.getItem("key");

localStorage.removeItem("key");
localStorage.clear();

```
**Session storage**: Lives only within the browser session. Persists data within a browser's session. Includes page reloads and restores. What's a session on mobile? When you go back to home, is it the same session? Quota is between 5MB to 12MB. 
**Local storage**: Keeps the data forever. It persists the data between navigation and browser sessions. Quota is 5MB. Strings are stored in UTF-16 in Chromium based browers. So it's in practise 2.5MB. 2 bytes per character, instead of 1.

To increase perf, quota and reachability we're going to be using indexedDB.

All browser data storage is public to the user. Anyone can see and change the data. Native apps, user etc.

## Differences between Promise.all, Promise.allSettled, Promise.any and Promise.race
Let's break down the differences between Promise.all, Promise.allSettled, Promise.any and Promise.race. These are all methods used to handle multiple promises concurrently in JavaScript, but they behave differently:

1. Promise.all(promises)

Behavior: Takes an array of promises and returns a single promise that resolves when all of the input promises resolve. If any of the input promises reject, the resulting promise immediately rejects with the reason of the first rejected promise.   
Success: Resolves with an array of the results of all the input promises, in the same order as the input array.   
Failure: Rejects immediately if any of the input promises reject. No results from the other promises (even if they resolved) are returned in this case.   
Use Case: Ideal when you need all promises to succeed before proceeding. For example, fetching data from multiple APIs where all the data is required.
JavaScript
   
```
Promise.all([fetch('/api/users'), fetch('/api/products')])
  .then(results => {
    // Both fetches succeeded
    const users = results[0];
    const products = results[1];
    // ... use both users and products data
  })
  .catch(error => {
    // At least one fetch failed
    console.error("Error fetching data:", error);
  });

```
2. Promise.allSettled(promises)

Behavior: Takes an array of promises and returns a single promise that resolves after all of the input promises have either resolved or rejected. It doesn't short-circuit like Promise.all.   
Success: Resolves with an array of result objects. Each result object has a status property (either "fulfilled" or "rejected") and either a value (if fulfilled) or a reason (if rejected). The results are in the same order as the input promises.   
Failure: Never rejects. It always resolves after all promises settle.
Use Case: Useful when you need to know the outcome of all promises, regardless of whether they succeed or fail. For example, reporting on the status of multiple asynchronous operations.
JavaScript
```
Promise.allSettled([fetch('/api/users'), fetch('/api/products')])
  .then(results => {
    results.forEach(result => {
      if (result.status === 'fulfilled') {
        console.log("Success:", result.value);
      } else {
        console.error("Failure:", result.reason);
      }
    });
  });

```
3. Promise.any(promises)

Behavior: Takes an array of promises and returns a single promise that resolves as soon as any of the input promises resolves. It short-circuits as soon as a promise resolves. It ignores rejections until all promises have rejected.
Success: Resolves with the value of the first resolved promise.
Failure: Rejects with an AggregateError if all of the input promises reject. The AggregateError contains an array of the rejection reasons.
Use Case: Useful when you only need the result of the first successful promise and don't care about the others. For example, trying multiple servers and using the response from the first one that responds.
JavaScript

```
Promise.any([fetch('/server1'), fetch('/server2'), fetch('/server3')])
  .then(result => {
    // The first server to respond successfully
    console.log("Response from:", result);
  })
  .catch(error => {
    // All servers failed
    console.error("All servers failed:", error.errors); // error.errors is the array of reasons
  });

```

4. Promise.race(promises)

Behavior: Takes an array of promises and returns a single promise that resolves or rejects as soon as any of the input promises resolves or rejects. It short-circuits as soon as the first promise settles (either resolves or rejects).   
Success/Failure: Resolves or rejects with the value or reason of the first promise to settle (either resolve or reject).
Use Case: Useful when you want to wait for the first promise to finish, regardless of its outcome. For example, implementing a timeout:
JavaScript

```
Promise.race([
  fetch('/api/data'),
  new Promise((_, reject) => setTimeout(() => reject('Timeout'), 5000)) // Timeout after 5 seconds
])
  .then(data => {
    // Fetch succeeded before timeout
    console.log("Data received:", data);
  })
  .catch(reason => {
    // Fetch failed or timed out
    console.error("Error or timeout:", reason);
  });

```

Key Differences Summarized:

| Feature         | `Promise.all`         | `Promise.allSettled`   | `Promise.any`          | `Promise.race`         |
|-----------------|-----------------------|------------------------|-----------------------|-----------------------|
| Resolves when   | All resolve           | All settle             | Any resolves          | Any settles            |
| Rejects when    | Any rejects          | Never rejects         | All reject           | Any settles (first)   |
| Result          | Array of values       | Array of result objects | Value of first resolve | Value/reason of first |
| Short-circuit   | On first rejection    | No                     | On first resolve       | On first settle        |


Understanding these differences is crucial for effectively managing asynchronous operations in JavaScript. Choose the method that best suits your specific needs based on how you want to handle the outcomes of multiple promises.







