title: Reverse engineering Google Chrome extensions
featured_image: files/images/featured/hacking-chrome-extensiona.jpg
summary: >-
  We will analyze a Google Chrome extension in order to determine how it works
  and potentially find a way to access its paid features without paying for
  them.
tags: []
categories: []
date: '2022-12-27T19:15:47.000Z'
---
Browser extensions are web apps whose goal is to enhance our browser experience. Because the code runs on our computer, we can learn the inner workings of the extension and maybe modify it to access some of its paid features for free. For this post, I've chosen to explore the inner workings of [Nimbus Screenshot & Screen Video Recorder](https://chrome.google.com/webstore/detail/nimbus-screenshot-screen/bpconcjcammlapcogcnnelfmaeghhagj/related?hl=en). This extension has over a million users and appeared in multiple categories on the google chrome extensions homepage, making it the perfect target. 

## #1: Find the code
The first step is to install the extension. Next, open the folder where the extensions code is stored. If you don’t know the location, visit **chrome://version** and look at the value for Profile Path. 
The profile path folder contains a subfolder named extensions, where the extensions code is stored. Extensions are referenced by their id. To find the id, visit **chrome://extensions**, and click details on the target extension. 
The folder name that matches the extension id will contain the code for our target.
## #2: Break the code ![bongo cat typing on a keyboard](/files/images/posts/hacking-chrome-extensions/code.gif)
Start by copying the folder with the extension code to another location(ex: desktop, documents, etc). Go to **chrome://extensions**, click the load unpacked button and select the new directory. 
From now on, Chrome will load the extension from the new folder.
Open the directory in your code editor. 
The best way to start is to look around and inspect the file structure. In the case of Nimbus, we can see the files are structured by file type and then by feature.
![](/files/images/posts/hacking-chrome-extensions/Explore-initial-template.png)
![](/files/images/posts/hacking-chrome-extensions/explore-js.png)
The next step is to look into the [manifest.json](https://developer.chrome.com/docs/extensions/mv3/manifest/) file. The manifest contains metadata, permissions, keyboard shortcuts, tells you which files run in the background and so on. Our manifest tells us that the **background.html** file is running in the background.
![screenshot of manifest-json file](/files/images/posts/hacking-chrome-extensions/manifest-json.png)
Opening this file reveals the loading of scripts needed for initializing the extension. Looking at the filenames, you can probably guess the purpose of each script. 
![screenshot of background.html file](/files/images/posts/hacking-chrome-extensions/background-files.png)
Let's skip to the fun part and look at **nscNimbusConnect.js**.
Peeking at the outline, we can see functions related to login, logout and checking if the user has a premium plan.
![](/files/images/posts/hacking-chrome-extensions/outline.png)
While inspecting the file, one line of code piqued my interest:
![](/files/images/posts/hacking-chrome-extensions/premium-sus.png)
VSCode was able to find the function, and I did what any developer would do.
![](/files/images/posts/hacking-chrome-extensions/get-opt.png)
 Added a console log and created a screenshot with nimbus so that I could inspect the output.
 ![](/files/images/posts/hacking-chrome-extensions/console-log.png)
 This was the result:
 ![](/files/images/posts/hacking-chrome-extensions/console-logs.png)
 Through experimentation with the different keys, I realized that adding 
 ```javascript
      if (key === "modePremiumNoNimbus") {
        return true;
      }
 ```
 
 unlocks premium features and hides buttons related to the [nimbus platform](https://nimbusweb.me/).
 And just like that, we hacked our first chrome extension 🥳
## #3: Share the new code
To share the new version, we have to repack the extension.
Go to **chrome://extensions** and click pack extension. Select the folder with the new code and click on the pack extension button.
Chrome will generate two files: a .crx and a .pem file. The .crx is the extension others can download and install in their browser.
Make sure to test the repacked extension. Google doesn’t like when others modify the chrome extensions and will try to stop us. If there are errors when importing the repacked extension, remove all unnecessary fields from the manifests file. These are the fields like update url, key, etc. Repack and test again.


## Next steps
As an attacker, the next step is creating an account and testing if the API restricts the use of premium features for free accounts.
As a defender, it depends. In the case of Nimbus, it seems like the source of revenue is the nimbus platform, something that we didn't touch on in this blog post. Doing nothing is the first option. The second option is to obfuscate the code to make it harder for others to bypass the restrictions. 
Moving the functionality from the client to the server is also possible. It will protect you from hackers at the cost of increased server costs.