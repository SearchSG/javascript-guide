# Javascript Guide

A list of do's and dont's when working with JS code

## Use Typescript

- First rule of a JS guide is to use TS instead of JS.

## Always prefer immutability

- This will be a recurring theme in this guide
- Immutable means that each variable we define will not be modified later on.
  > Why so obsessed with immutability?
  >
  > A function starts off short and simple. However, it is very common that complexity increase over time as we add features. Mutable variables gets hard to reason about and leads to unintended side-effects.

```ts
// Bad example: function that modifies object
function enhanceWithMetadata(obj) {
  obj.key2 = 123;
}

let foo: { key: string; key2: string } = { key: "value", key2: "value2" };
enhanceWithMetadata(foo);
foo; // { key: "value", key2: 123 } does not conform to the typing
```

---

## Use const, not let

- Always use `const` to define new variables
- If you really need to use `let`, extract that part into a small, helper function.
- Con: This can easily lead to variable sprawl, we will use lodash to mitigate that

---

## Iterators & lodash

- Never ever use a `for` loop. Use `lodash` functions instead
- `lodash` has a many useful pure functions
- The functions can be chained together, with each function massaging the passed in variable a bit until the final form
- You can google any problem you are trying to solve and add `loadsh` and see how people are chaining loadash functions
- eslint: [`no-iterator`](https://eslint.org/docs/rules/no-iterator) [`no-restricted-syntax`](https://eslint.org/docs/rules/no-restricted-syntax)

> Why?
>
> Most lodash functions do not modify the passed in variable. This is consistent with our rule on immutability. Do note there are some that mutate the variable, see lodash.com docs to confirm.

### Example:

```ts
// Following function decodes a session cookie string into a JS object
function decodeSessionCookie(sesssionCookieStr: string): ISearchSessionCookie {
  // By using lodash chaining, we avoided variable sprawl in this function. There is only one const
  const sesionCookieObj = _(sesssionCookieStr)
    // ["sid=123", "uid=456xyz"]
    .split(" ")
    // [["sid","123"], ["uid",["456xyz"]]
    .map((x) => x.split("="))
    // {sid:"123",uid:"456xyz"}
    .fromPairs()
    .value() as ISearchSessionCookieDTO;
  return {
    sessionId: sesionCookieObj.sid,
    userId: sesionCookieObj.uid,
  };
}
// {sessionId: '123', userId: '456xyz'}
decodeSessionCookie("sid=123 uid=456xyz");
```

---

## Documenting in code

- Make use of JSDocs in your code
- In interfaces, JS Docs for the interface itself, as well as properties in the interface
- In functions, JS Docs to roughly explain what the function does
- Sometimes, we think the code is self-explanatory and docs are for n00bs.but when there is a long trail of functions, docs help us to understand roughly what each one is doing. This makes it a lot easier to piece everything together quickly.
- Documentation are expensive to maintain, keeping some of it closer to the code is a tradeoff between maintaining docs and having no docs at all

### Example

```ts
/**
 * The inputs for override web domain
 */
interface IInput {
  /**
   * display name that follows the search domain
   * @example life.gov.sg
   */
  displayName: string;
  /**
   * s3FileWithUrls is for adhoc executions where a list of URLs are read from S3 file.
   * 's3FileWithUrls' and 'startingUrl' are mutually exclusive configs
   */
  s3FileWithUrls?: string;
  /**
   * The starting URL to start the job at. 's3FileWithUrls' and 'startingUrl' are mutually exclusive configs
   */
  startingUrl?: string;
}

/**
 * Processes the input
 * @param input the input to the function
 * @returns the string representation of input
 */
function processInput(input: IInput): string {
  const output = input.toString();
  return output;
}
```

---

## Use Bluebird to do async iteration

- [Bluebird](http://bluebirdjs.com/docs/api-reference.html) is a promise library with useful iteration functions, like `map`, `mapSeries`,`some`

> Why do we need this?
>
> - Sometimes, we want to perform interation with async functions
> - For example, we want to read all the files in a folder and extract the content of the file

### Example

```ts
const filesNamesToProcess = ["a.json", "b.json"];
const arrOfStrings = await bluebird.map(
  filesNamesToProcess,
  async (fileName) => {
    return await fs.promises.readFile(fileName, "utf-8");
  },
  { concurrency: 1000 }
);
```

---

## Mocks in code testing

- We can use Dependency Injection (DI) to make our functions easier to test
- General idea: Sometimes, we have functions that contains code that fetch data from somewhere, and then perform logic on that data
- In these scenario, we want to mock the returned data
- This can be done in jest with some setup, but we can achieve the same thing with DI

```ts
/**
 * fetchData is an async function
 **/
async function fetchData(key): string {
  return await s3.getObject(key);
}

/**
 * fetchData is a last optional param in the function
 **/
function main(input: { key: string; metadata: string }, fetchData = fetchData) {
  const data = await fetchData(input);
  if ((data.meta = metadata)) {
    //do something
    return;
  } else {
    return;
  }
}

/**
 * Testing main function
 **/
function testMain() {
  expect(
    // swap out fetchData with our function that returns specific output
    main({ key: "mykey", metadata: "mymeta" }, () => {
      return "mymeta";
    })
  ).toEqual("something");
}
```
