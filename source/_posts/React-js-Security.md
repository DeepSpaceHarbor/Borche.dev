title: Preventing unauthorized code execution in React.js
featured_image: files/images/featured/hack-macbook-money.jpg
summary: >-
  Your web app got hacked. The attacker modified the source code, and your
  customers downloaded the malicious version. What mechanisms do you have in
  place to prevent and minimize the damage?
tags:
  - React.js
  - React
  - Web Security
  - XSS attacks
categories: []
date: '2022-04-17T22:00:00.000Z'
---
Cybercrime is a serious business. Look at the [pew pew maps](https://threatbutt.com/map/) that show it all. Everyone and everything is under attack 24/7. One could argue that security knowledge is a must-have for every developer.

At the highest level, there are two main ways to hack a web app:
- Change the source code and serve malicious code to the user.
- Turn input sources such as database entries or URL parameters into malicious content.

With proper configuration, modern browsers can detect and stop these attacks.

## Stopping the malicious client
One way to introduce malicious code is to gain access to the server hosting the app and change the source code. A variation of this attack is gaining access to the CI/CD pipeline. After the code changes, the server will serve the malicious version to the user.
**Subresource integrity(SRI)** is the first line of defense against this type of attack. To activate these checks add integrity attribute to resources([`<link/>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#attributes) and [`<script/>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attributes) tags).
The integrity attribute should contain the cryptographic signature of the resource.
For example:
```html
<script src="https://example.com/example-library.js" integrity="sha384-qVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC" crossorigin="anonymous"></script>
```

The browser then compares the cryptographic signature of the downloaded resource with the cryptographic signature from the integrity attribute. 
If the signatures don't match, the browser will refuse to execute the downloaded resource and show a network error.

For libraries without SRI, tools such as [srihash.org](https://www.srihash.org/) can compute the cryptographic hash. In-depth details about subresource integrity are available on the [MDN website](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) and the official [specification](https://www.w3.org/TR/SRI/).

**The Content Security Policy(CSP) header** is another security feature at our disposal. 
It consists of different directives that limit how different types of resources behave. There are many versions(levels) of the standard. 
Each version introduces new directives that allow for more fine-tuning.

A policy that allows everything but only when it comes from the same origin looks like this:
```http
Content-Security-Policy: default-src 'self';
```
Policy for embedding scripts from external sources might look like this:
```http
Content-Security-Policy: script-src 'self' www.google-analytics.com service.example.com;
```
The same patterns are applied when specifying the behavior of other resources.

Writing CSP policy from scratch can be tedious, but some tools can help. 
The [CSP generator tool](https://report-uri.com/home/generate) from Report URI is a great starting point. 
It has a list of common directives and an explanation for each directive. After setting up the rules for each resource, it will generate a content security policy for you. If you're working on a project where defining the policy is hard, the [CSP generator](https://csper.io/docs/generating-content-security-policy) from Csper can help. It's a plugin that collects info about the resources loaded by the web app and creates starter policy. 
After you've created your policy, it's recommended that you test using report only mode. This will instruct the browser to use the policy and report back the errors, without breaking the web app. You can do this by setting the header name to `Content-Security-Policy-Report-Only`. 
To test your policy without deploying use browser plugins such as [Laboratory](https://addons.mozilla.org/en-US/firefox/addon/laboratory-by-mozilla/).


## Dealing with malicious input
An example of a malicious input would be someone using javascript code instead of their name. If the web app is vulnerable to XSS attacks and has "all users" page, the code will run and cause problems. 
Another example is changing the query string parameters to include unwanted code. The new link can be sent to the victims through a phishing campaign.
React.js applies auto-escaping by default. 
```jsx
export default function Demo() {
  const title = maliciousInput;
  //This is safe because the input is sanitized.
  return <h1>{title}</h1>;
}
```
But this protection doesn't apply to props and escape hatches.
Spreading of user controlled props is a common problem.
```jsx
const maliciousInput = "<img onerror='alert(\"hacked\")' src='123'/>";

export default function Demo() {
  const maliciousProps = {
    dangerouslySetInnerHTML: {
      __html: maliciousInput,
    },
  };
  //This is dangerous. User controlled props can introduce malicious code.
  return <div {...maliciousProps} />;
}
```
A single user controlled prop can also be unsafe, even more so when combined with an [escape hatch](https://reactjs.org/docs/design-principles.html#escape-hatches).
```jsx
const maliciousInput = "javascript:alert('hacked')";

export default function Demo() {
  const url = maliciousInput;
  //This is dangerous.
  return <a href={url}>My Website</a>;
}
```
```jsx
import { useRef } from "react";

const maliciousInput = "<img onerror='alert(\"hacked\")' src='123'/>";

export default function Demo() {
  const maliciousRef = useRef();
  function sayHello() {
    //This is dangerous. The content of the input is not sanitized when escape hatch is used.
    maliciousRef.current.innerHTML = maliciousInput;
  }
  return (
    <>
      <button onClick={sayHello}>Say hello</button>
      <div ref={maliciousRef}>Hi 👋</div>
    </>
  );
}
```

If you are working on an app that requires rendering of html code you should sanitize the input. [DOMPurify](https://github.com/cure53/DOMPurify) is one of the recommended libraries to do this.
It takes an html code and removes everything that might trigger an XSS attack.
```jsx
import DOMPurify from "dompurify";

const maliciousInput = "<img onerror='alert(\"hacked\")' src='123'/>";

export default function Demo() {
  return (
    <div
      //This is safe. DOMPurify will remove the dangerous code.
      dangerouslySetInnerHTML={{
        __html: DOMPurify.sanitize(maliciousInput),
      }}
    />
  );
}
```

In cases when you're working with a subset of html, you can also use libraries such as [sanitize-html](https://www.npmjs.com/package/sanitize-html). With sanitize-html you can define the set of allowed tags, attributes, etc.
```jsx
import sanitizeHtml from "sanitize-html";

const maliciousInput = "<img onerror='alert(\"hacked\")' src='123'/> <b>hello world!</b>";

export default function Demo() {
  // Allow only a super restricted set of tags and attributes
  const clean = sanitizeHtml(maliciousInput, {
    allowedTags: ["b", "p"],
    allowedAttributes: {},
  });
  return (
    <div //The output is <div><b>hello world!</b></div>
      dangerouslySetInnerHTML={{
        __html: clean,
      }}
    />
  );
}
```
The best way to catch these issues is during code reviews. 
Tools such as [SemGrep](https://semgrep.dev/), [SonarLint](https://www.sonarlint.org/), and [Snyk Security](https://snyk.io/product/snyk-code/) can spot potential problems during development. Use your judgment when deciding if reported items are real vulnerabilities. There are plenty of tools that can help us find suspicious code, but none of them is 100% effective.