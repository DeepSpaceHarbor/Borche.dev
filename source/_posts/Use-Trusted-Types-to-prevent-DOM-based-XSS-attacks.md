title: Trusted types can prevent DOM-based XSS attacks
featured_image: files/images/featured/trusted-types.jpg
summary: State of the art XSS prevention with the help of trusted types.
tags: []
categories: []
date: '2023-04-01T01:23:27.000Z'
---
If you’re reading this, you’re probably familiar with the concept of XSS attacks. Most web apps accept user-supplied input, typically provided in input fields or coming from API requests. This input travels throughout the codebase until it reaches a function where it is misinterpreted and executed as code.
This function is called a sink. Functions like eval, innerHTML, and document.write are typically cited as examples of sink functions.
Trusted types [^1] are a new experimental mechanism that tells the browser to stop accepting potentially dangerous strings as inputs into the sink functions.
It forces the developers to use safe API when manipulating the DOM and create trusted type policies that will sanitize the input data from places beyond their control.
Take this code as an example. It has two inputs one for the name and another for the URL to an avatar.  When any of the inputs changes, the user sees a greeting with their name and avatar.

```html
<head>
  <script>
    function welcomeUser() {
      const name = document.querySelector('input[name="fname"]').value;
      const avatar = document.querySelector('input[name="avatar"]').value;
      const welcomeMsg = document.querySelector("#welcomeMsg");
      welcomeMsg.innerHTML = `<h2>Hello 👋 ${name}!</h2><img src="${avatar}" />`;
    }
  </script>
</head>
<body>
  <form>
    <label for="fname">First name:</label><br />
    <input type="text" name="fname" onchange="welcomeUser()" /><br />
    <label for="avatar">Avatar URL</label><br />
    <input type="text" name="avatar" onchange="welcomeUser()" />
  </form>
  <div id="welcomeMsg"></div>
</body>

```
![](/files/images/posts/trusted-types/base-code.png)
This code is obviously insecure. To test that type 
```text
My name is <img src="x" onerror="alert('XSS')">
```
as first name and observe the result.
![](/files/images/posts/trusted-types/Selection_125.png)
Next, enable trusted types through the content security policy. Typically, you would do this through the headers, but since this is a demo, we can take a shortcut and create CSP through the meta tags. 
```html
<meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'" />
```
Open developer tools and type something in the first name field. The browser will stop you from putting the potentially dangerous input into the dom.
![](/files/images/posts/trusted-types/Selection_126.png)

One way to deal with this error is to refactor the welcomeUser function into something like this:
```javascript
    function welcomeUser() {
      const name = document.querySelector('input[name="fname"]').value;
      const avatar = document.querySelector('input[name="avatar"]').value;
      const welcomeMsg = document.querySelector("#welcomeMsg");
      const helloEl = document.createElement("h2");
      const imgEl = document.createElement("img");
      imgEl.src = avatar;
      helloEl.textContent = `Hello 👋 ${name}!`;
      welcomeMsg.replaceChildren(helloEl, imgEl);
    }
```
The other way is to create a trusted types policy and sanitize the input before passing it to the sink function. A sample policy that takes an insecure string and passes it to the sink function looks like this:
```javascript
    const escapeHTMLPolicy = trustedTypes.createPolicy("default", {
      createHTML: (string) => string,
    });
```
This will get our app back to a working state, but the security problem will remain. To safeguard from XSS attacks, use [DomPurify](https://github.com/cure53/DOMPurify) to sanitize the inputs before passing them to the sink functions.
```html
<head>
  <script
    src="https://cdnjs.cloudflare.com/ajax/libs/dompurify/3.0.1/purify.min.js"
    integrity="sha512-TU4FJi5o+epsahLtM9OFRvH2gXmmlzGlysk9wtTFgbYbMvFzh3Cw1l3ubnYIvBiZCC/aurRHS408TeEbcuOoyQ=="
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  ></script>
  <meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'" />
  <script>
    trustedTypes.createPolicy("default", {
      createHTML: (string) => DOMPurify.sanitize(string),
    });
    function welcomeUser() {
      const name = document.querySelector('input[name="fname"]').value;
      const avatar = document.querySelector('input[name="avatar"]').value;
      const welcomeMsg = document.querySelector("#welcomeMsg");
      welcomeMsg.innerHTML = `<h2>Hello 👋 ${name}!</h2><img src="${avatar}" />`;
    }
  </script>
</head>
<body>
  <form>
    <label for="fname">First name:</label><br />
    <input type="text" name="fname" onchange="welcomeUser()" /><br />
    <label for="avatar">Avatar URL</label><br />
    <input type="text" name="avatar" onchange="welcomeUser()" />
  </form>
  <div id="welcomeMsg"></div>
</body>

```
For situations where more granular control is needed, custom policies can help. If we were to recreate the same example, we would start by creating a new policy.
```javascript
    document.myCustomPolicy = trustedTypes.createPolicy("myCustomPolicy", {
      createHTML: (string) => {
        return DOMPurify.sanitize(string);
      },
    });
```
Follow up by creating safe values using the new policy.
```javascript
 welcomeMsg.innerHTML = document.myCustomPolicy.createHTML(`<h2>Hello 👋 ${name}!</h2><img src="${avatar}" />`);
```
Finally, the CSP header is updated to inform the browser of the changes.
```html
  <meta
    http-equiv="Content-Security-Policy"
    content="require-trusted-types-for 'script'; trusted-types myCustomPolicy dompurify"
  />
```
That's it! 🥳 We just used trusted types to secure a part of our web app.

## Next Steps
Adopting trusted types in your organization is a great way to promote the usage of safe APIs and drastically reduce the XSS attack surface. 
When Google started its API hardening project [^2], the majority of the questions and help requests occurred in the first two weeks. 
In the following weeks, it fell to an average of one question per week. 
This experience suggests that large organizations should be able to adopt trusted types with relatively low effort. As expected, the number of XSS reports saw a significant decrease. 
The development team behind Angular had a similar experience [^3]. 
It took six weeks for a team of four security engineers and one intern to implement trusted types. During this period, they implemented trusted types and discovered previously unknown vulnerabilities in Angular and the surrounding ecosystem. 
If you're looking to play with this, you should know that trusted types are [available](https://caniuse.com/trusted-types) in Chromium based browsers, and a polyful is available in the [w3c repository](https://github.com/w3c/trusted-types#polyfill).

[^1]: Google LLC. "Trusted Types", W3C, Available: [https://w3.org/TR/trusted-types/](https://w3.org/TR/trusted-types/#introduction).
[^2]: Pei Wang, Julian Bangert, and Christoph Kern. "[If it’s not secure, it should not compile: Preventing DOM-based XSS in large-scale web development with API hardening.](https://research.google/pubs/pub49950/)" In 2021 IEEE/ACM 43rd International Conference on Software Engineering (ICSE), pp. 1360-1372. IEEE, 2021.
[^3]: P. Wang, B. Á. Guðmundsson and K. Kotowicz, "[Adopting Trusted Types in ProductionWeb Frameworks to Prevent DOM-Based Cross-Site Scripting: A Case Study](https://research.google/pubs/pub50513/)," 2021 IEEE European Symposium on Security and Privacy Workshops (EuroS&PW), Vienna, Austria, 2021, pp. 60-73, doi: 10.1109/EuroSPW54576.2021.00013.