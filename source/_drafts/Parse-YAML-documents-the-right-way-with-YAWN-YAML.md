title: How to edit YAML files without losing comments and styling
featured_image: files/images/featured/package.jpg
summary: Parse YAML documents the right way with YAWN YAML
tags: []
categories: []
date: 2023-10-23 00:48:04
---
Sometimes you end up with a YAML file that needs to be updated, but only for a few specific values, keeping the rest of the content as it is.
In my case, it was a configuration file that 

YAML files are often used as configuration. As time goes by, these files keep on adding new entries, but the need for backward compatibility and keeping the comments untouched remains the same.

[YAWN YAML](https://github.com/mohsen1/yawn-yaml) is a YAML parser that will keep the comments and styling of the original file.



## YAWN YAML

```JavaScript
const YAWN = require("yawn-yaml/cjs");

let template = `
# Editor config
darkMode: False # Are you a night owl?
# misc
lineWidth: 80
`;

let yawn = new YAWN(template);

yawn.json = { darkMode: true, lineWidth: 120 };

// value in `yawn.yaml` is now changed.
console.log(yawn.yaml);

```
## Enhanced YAML
https://github.com/paul-soporan/enhanced-yaml