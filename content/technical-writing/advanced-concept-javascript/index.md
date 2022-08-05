---
title: "Advanced JavaScript Concepts that Helped Me Get Better at Coding"
date: 2022-06-09T14:46:15+02:00
draft: false
slug: "technical-writing/advanced-concept-javascript"
description: "Javascript concepts to get better at coding."
keywords:
  ["javascript", "concept", "coding", "code", "programming", "web", "api"]
toc: true
---

![illustration-article](./illustration-article.png)

JavaScript is one of the most dynamic languages out there. Multiple features are added to make the language more manageable and practical each year.

In this blog post, we are going through some recent features added to the language and some tricks that you could use in a daily coding journey that will help make you better at JavaScript coding.

Writing this blog post allows me to share some of the concepts that I learned over the last years condensed into one place.

## Destructuring assignment

Destructuring assignment allows changing the variable's name extracted from an object through destructuring.

```
const { address: addressLine } = {
  address: "20B Rue Lafayette",
  postcode: "75009",
};
console.warn(addressLine); // 20B Rue Lafayette
const [first, second] = [1, 2, 3, 4];
console.warn(first, second); // 1 2
```

Destructuring assignment can be very handy when you change the variable name to give more context to a particular part of the code.

However, it would be best not to use destructuring assignment every single time as it is preferable that you name your variable accordingly right the first time.

But it would help if you instead want to provide additional context to a specific part of the code.

## Optional chaining

Optional chaining exists in other languages such as Swift. It permits access to property or methods from objects without throwing if they, for some reason, are not available. JavaScript is not a typed language, so accessing the unavailable property is a common source of issues with the traditional:

```
const contactInfos = { address: "20B Rue Lafayette" };
console.warn(contactInfos.user.phoneNumber)
// Cannot read properties of undefined (reading 'phoneNumber')
```

However, using the optional chaining through the `?` keyword allows us to avoid throwing.

```
const contactInfos = { address: "20B Rue Lafayette" };
console.warn(contactInfos.user?.phoneNumber); // undefined
```

Then, by receiving undefined as a default value, we have the entire liberty to fall back on something else that keeps the application healthy.

## Coalescing operator

The coalescing operator is very nice to use with the optional chaining for setting a default value. Thus, you could use the code below to fall back on a default phone number which is an empty string:

```
const contactInfos = { address: "20B Rue Lafayette" };
console.warn(contactInfos.user?.phoneNumber ?? ""); // ""
```

The coalescing operator is an operator that returns his right operator when the left operator is either `null` or `undefined`. The coalescing operator has been created to address a common issue with JavaScript regarding truthy and falsy values.

Indeed, for example, the number `0` is considered a falsy value in the JavaScript run time. However, sometimes the `0` value in your application logic should be viewed as a valid value that we would use.

For example:

```
const contactInfos = { address: "20B Rue Lafayette", addressNumber: 0 };
console.warn(contactInfos.addressNumber || undefined); // undefined

console.warn(contactInfos.addressNumber ?? undefined); // 0
```

## Conditionally add a property to an object

Have you ever wanted to add a property to a JavaScript object but only if some requirements are met?

If you encountered that situation, the code you may have produced could be very verbose as two `if` statements are required. However, a JavaScript trick exists that allows us to eliminate the additional statements we would have written.

```
const moreInfos = { info: "Please go to the desk." };
return {
  address: "20B Rue Lafayette",
  postcode: "75009",
  ...(moreInfos !== undefined && { moreInfos }),
};
```

Here it is! We are only adding the `moreInfos` property only when the variable value is different from `undefined`.

We are leveraging the spread syntax (`…`) to spread the object `{ moreInfos }` and using the `AND` operator that permits us to return the second value of the condition when the first is evaluated to `true`.

For instance:

```
console.warn(true && "1") // "1"
console.warn(false && "1") // false
```

However, that syntax should be used cautiously, as sometimes, the expression is not very readable depending on the complexity of the condition and the verbosity of the object we want to spread.

## Reference for non-primitive value

Like any programming language, it contains primitive and non-primitive values. In JavaScript, primitive values are Number, String, Boolean, Undefined, Symbol, and BigInt. Other data types are passed by reference, which means that the variable is given instead of just a copy.

For example:

```
const contactInfos = { address: "20B Rue Lafayette" };
function setAddress(infos) {
  infos.address = "90B Rue laffite";
}
console.warn(contactInfos); // { address: "20B Rue Lafayette" }
setAddress(contactInfos);
console.warn(contactInfos); // { address: "90B Rue laffite" }
```

Since you pass the variable itself, the address in memory is passed. Then, wherever you have the address of that variable, you can update it, and the changes will propagate in other parts of the code where this variable has the same address as the variable you just modified.

JavaScript is a high-level programming language, it means less close to the machine, and memory is handled for us. So, you have to be careful when you want to pass down JavaScript non-primitive value down a function because if you mutate it and if you are not aware of this behavior, unexpected issues can happen.

## Checking the return value of methods

It may sound obvious, but we must always check for the function's return value, especially from the standard library. I've seen many issues caused by this. For example, if you want to search in a collection, an object thanks to some of its properties:

```
const contacts = [
  { address: "20B Rue Lafayette", name: "work" },
  { address: "90B Rue laffite", name: "bar" },
];
const contact = contacts.find((contact) => contact.name === "home");
console.warn(contact); // undefined
```

But for some reason, if the object you search for is unavailable, you will end up with a variable set to unexpected things. It's the best way of making a mistake. That's why you should always check for the function's returned value.

You can apply that recommendation to all the programming languages in the world.

## Catching throwing async/await promise with catch helper

Using `async/await` is incredibly powerful to write clean and readable code to avoid turning a code into a callback hell style. However, the way we handle the error case is indeed different from the `then/catch` method that turns into a `try/catch`.

Here is a handy trick I find to help handle the throwing state of the promise:

```
const results = await getPosts().catch((err) => {
  return {
    type: "error",
    message: err.message,
  };
});
console.warn(results); // { type: "error", message: "cannot get posts from this endpoint" }
```

Doing that helps to stay in the same non-error state rather than going in the `catch` of the `try/catch`.

In some situations, it can significantly help where you want to control some specific error state that you potentially expect.

## Weakmap

Finally, the `WeakMap` is a data structure less known and less used in JavaScript, but I find it interesting enough to speak about it. It's a classic hashmap data structure with key/value storage. Still, the only difference is that the entry got deleted automatically from the map object when no reference points to it in memory. But also, the key could be whatever object type, which means that you can retrieve the value associated with the key by bypassing the whole object. It looks for the address in memory to retrieve the good one. The cleaning is called a garbage collector mechanism.

The `WeakMap` could be used in applications lacking memory, like embedded devices where resources are precious.

```
const videoSegments = new WeakMap();
let options = { id: "1234", timeStart: 1653831957378, size: 10000 };
const segment = { data: new Uint8Array(200) };
videoSegments.set(options, segment);
console.warn(videoSegments.get(options)); // { data: new Uint8Array(200) }
options = null;
console.warn(videoSegments.has(options)); // false, the `options` key object is deleted from the WeakMap
```

## Conclusion

I hope you found that article helpful, don't hesitate to give feedback for me to improve the paper.

Thank you for your attention.
