# Building a JavaScript Web Crawler.

## Getting started with node
### What is npm?
**npm** stands for **Node Package Manager**, by installing node we gain access to three new commands:
1. **node** - The JavaScript Runtime, to run our JS programs.  
2. **npm** - The package manager that manages dependencies, metadata and allows us to specify which scripts to run.  
3. **npx** - The package Runner, allows us to run a command from a local or remote **npm** package.
## Our First npm Project
- We start by initializing a new project in a new directory (_JsWebCrawler_)
with the following command:

> npm init

* This command will ask us several questions about our project and then create a custom _package.json_ file.
- We create a new file name _main.js_ with a simple _console.log_.  

- Then we modify our _package.json_ file to add support for _ES Modules_ and add a _start_ script to execute our _main.js_ file:

```json
...  
"type": "modules",  
"scripts" : {  
    "start": "node main.js" 
 },  
...
```

- We can now run our code and should get the output from _main.js by simply typing:
>npm start
* A _.gitignorefile_ is used to tell which files and folders to ignore before we commit and push our changes.  
we can create one and have it ignore our _node___modules/_ directory.

## Test Driven Development
TDD is a popular way to write software, the idea is that your write test for your code first, then your write the code that gets the tests to pass.
One of the popular libraries for TDD is **Jest**.

### Installing Jest
> npm install jest --save-dev

### Configure a test command
- We add a _test_ script and an empty _"jest.transform"_ property to out _package.json_ file:
``` json
...
"scripts": {  
    "start": "node main.js",  
    "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js"  
    },  
    "jest": {},  
...
```

* This enables **jest** to use **ES Modules**.

### Normalizing URLs
Normalizing a URL, means standardizing the way it is written, to understand this better, take a look at the URLs bellow:
```
-  https://blog.boot.dev/path/
-  https://blog.boot.dev/path
-  http://blog.boot.dev/path/
-  http://blog.boot.dev/path
```
All of these URLs are the same according to **HTTP** standards, but may cause a problem in our program when parsing them.
That is why we need to write a function that takes a URL as an argument and **normalizes** it.

To start, we create a _crawl.js_ file with our code, we can write our own code to normalize the URL:
``` javascript
function urlNormalizer(anyUrl){
        let normalizedUrl = anyUrl.toLowerCase().trim();

        if ((anyUrl.startsWith("http")) || (anyUrl.endsWith("/")))
                {
                        if (anyUrl.startsWith("https://")){
                                normalizedUrl = anyUrl.slice(8);
                        }

                        else if (anyUrl.startsWith("http://")){
                                normalizedUrl = anyUrl.slice(7);
                        }

                        if (anyUrl.endsWith('/')){
                                normalizedUrl = normalizedUrl.slice(0,-1);
                        }
                }
       
        return normalizedUrl;
}
```
Or use Node's **URL Object** API to dissect our **URLs**, which is the preferable way:

```javascript
function urlNormalizer(anyUrl)
{
        const myUrl = new URL (anyUrl);
        if (myUrl.pathname.endsWith('/'))
        {
            myUrl.pathname = myUrl.pathname.slice(0, -1);
        }
        return (myUrl.hostname + myUrl.pathname);
}
```

- Then we need to export our file to make it accessible to other modules:
 
``` javascript
export {urlNormalizer};
```

- Now let's create our test file __crawler.test.js_ to be used to be used by **jest**, (jest automatically detects files ending with _.test.js_ as test files) and import jest and our **urlNormalizer();** method:
```javascript
import { test, expect } from "@jest/globals";
import { urlNormalizer } from "./crawl.js";
```

- Then we add our testing logic:
```javascript
const testUrl = "https://www.example.com/users/user1"  
test("testing urls", () => {
    expect(urlNormalizer(testUrl)).toBe('www.example.com/users/user1');
});
```
- We can include any edge case we can think of.


## Dependencies and URL Extraction
Our link tracker needs to know how to read a page of HTML text and extract links
To do so we need to add a third party dependency **jsdom** which will give us node access to the Document Object Model API.

> npm install jsdom

This time we will **not** use the **--save-dev** argument since we want it to be project dependency rather than a development dependency.

We will implement that dependency inside of our _crawler.js_ file:

```javascript
import {JSDOM} from 'jsdom';
...
function getUrlFromHtml(htmlBody, baseUrl){
    const dom = new JSDOM(htmlBody);
    let foundUrls = dom.window.document.querySelectorAll('a');
    // returns a array of all the anchers found.
    for (i of foundUrls)
    {
        return baseUrl + i;
    }
}
...
export {getUrlFromHtml}
```
#### Testing the code
Before committing the code, we can test that:
1. Relative URLs are converted to absolute (full) URLs.
2. The code finds all the "anchor" elements in an HTML body.

## The Main Program
Now our application needs to  be able to take a URL as an argument from the command line.
The prerequisites are:
* The main program only takes 1 URL as an argument and returns a n error if more or no arguments are passed.
* If exactly one URL is passed, print a message.

We will be using the process.argv builtin method that will return an arrays of arguments passed tot the CLI, our actual URL starts at index 2 of the array, so we take care of that with a .slice() method.

```js
let givenUrl = process.argv.slice(2);

function main(givenUrl){
        if (givenUrl.length > 1)
        {
                console.log("Error, Too many arguments");
        }
        else if (givenUrl.length < 1)
        {
                console.log("Error, no URL Provided");
        }
        else
        {
                console.log(`Processing : ${givenUrl} \n`);
        }
}

```

## Crawling a page

Now its time to crawl a real web page.
We start by creating a *crawlPage function* inside crawl.js file, that takes a URL (The root of the website we want to crawl).
To do this we will use the **fetch API**:
The prerequisites are to:
* use fetch to fetch the page of the current URL.
* Return an error if the HTTP code is not 200,(300 (redirects) are OK, but not yer implemented.)
* if the _content-type_ header is not _text/html_ print an error and return.
* if everything is OK, print the HTML code.

```js
async function crawlPage(currentUrl) {
        try {
                const response = await fetch(currentUrl,{
                        headers: {
                        "Content-Type": "text/html"
                        }
                });

                const dataReceived = await response.text();
                const contentType = await response.headers.get("content-type");

                if (!response.ok)
                {
                        throw new Error ("Error: Response Status: " + response.status);
                }

                else if (!contentType || !contentType.includes("text/html"))
                {
                        throw new TypeError ("Error: Not Html");
                }

                //console.log(dataReceived);
                return dataReceived

        }

        catch (err)
        {
                console.log(err.message);
        }
}

...
export {crawlPage}
```
Then we update our main function in main.js to reflect the changes.

```js
import {crawlPage} from './crawl.js'
...
async function main(givenUrl){
        if (givenUrl.length > 1)
        {
                console.log("Error, Too many arguments");
        }
        else if (givenUrl.length < 1)
        {
                console.log("Error, no URL Provided");
        }
        else
        {
                console.log(`Processing : ${givenUrl} \n`);
                let myVar = await crawlPage(givenUrl);
                console.log(myVar)

        }
}
```

