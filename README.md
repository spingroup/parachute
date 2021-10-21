# parachute
Webpack plugin which  componentizes JSON files for hubspot modules.

## Before you begin

Parachute assumes a few things about your webpack build:

1. Your src folder follows a similar structure as the base folder for a hubspot theme. 
   1. Check out the [hubspot boilerplate](https://github.com/HubSpot/cms-theme-boilerplate) for an example.
2. Your webpack setup is something similar in terms of plugins and overall functionality that is described in [Faster Better Blog by Andrey Kondratyuk](https://fasterbetter.dev/blog)
   1. If you don't have a webpack configuration for your hubspot build quite yet I would highly encourage you to check out his blog. He has some great insight and some great plugins to get you started along the way. 
3. Parachute will work with [Hubspot Auto Uploader Plugin](https://www.npmjs.com/package/@hubspot/webpack-cms-plugins/v/2.2.1)**
4. CopyWebpackPlugin is being used to port over all files not being bundled or run through webpack.

** Because we were working with the assumption of a few different plugins and the fact that we are not webpack experts -- you will notice that a correct install of webpack actually runs part of the bundling process twice... This is no mistake. It is a potentially, short term solution to the issue of making sure that the timing of fields.js being converted into fields.json jives correctly with all of the other plugin's timings. 


*** If you dive into the code and can figure out a more "webpack" solution that will work with both the AutoUpload plugin and CopyWebpackPlugin please feel free to submit a pull request!

## Installation 

`npm install --save @jazzyclimber/parachute`

### webpack.config.js

Configurations may vary depending on your specific implementation and use of themes within webpack. 

```
const parachute = require('@jazzyclimber/parachute');

...
module.exports = ({
  ...
  plugins: [
    new parachute(),
    ...
  ]
})
```

## Usage

After successful install of parachute you will now be able to write js files which will be converted into module fields.json files on bundle. 

### Expected file structure
```
--| .env
--| src
----| json // Folder to house all of your "json components"
------| button.json //reusable json file
----| modules
------| example-module.module
---------| module.html
---------| module.css
---------| module.js
---------| fields.json // This should be blank and will be overwritten
---------| fields.js // This is your prebundled entry point for fields.json
```

### .env

```
HAS_WRITTEN=False
```

### Json Component File 

This file will end up being a single source of truth for keeping field.json "chunks" consistant across the entire code base. Any json that is valid inside of fields.json is valid here. 

This will eventually be imported into whatever module or fields.js file you want to use it with. 

**Animations.json**
```
{
  "children": [{
    "default": false,
    "display": "checkbox",
    "help_text": "Will animate the content/copy section of the header banner",
    "id": "bdb2b290-1647-dc68-941a-fc2ac343938e",
    "label": "Animate Content?",
    "locked": false,
    "name": "animate_content",
    "required": false,
    "type": "boolean"
  }, {
    "choices": [
      ["fade-up", "Fade Up"],
      ["fade-down", "Fade Down"],
      ["fade-left", "Fade Left"],
      ["fade-right", "Fade Right"]
    ],
    "default": "fade-up",
    "display": "select",
    "id": "9a0de064-8dcd-d8f9-2104-c0e318d40afa",
    "label": "Content Animation Direction",
    "locked": false,
    "name": "type",
    "required": false,
    "type": "choice",
    "visibility": {
      "controlling_field": "bdb2b290-1647-dc68-941a-fc2ac343938e",
      "controlling_value_regex": "true",
      "operator": "EQUAL"
    }
  }],
  "default": {
    "animate_content": false,
    "type": "fade-up"
  },
  "expanded": false,
  "id": "3b2b8d46-09e4-dc25-05e3-081a52d52172",
  "label": "Animations",
  "locked": false,
  "name": "animations",
  "required": false,
  "tab": "CONTENT",
  "type": "group"
}
```

### Fields.js

```
// Any fields or groups that may be specific to this module instance. 

var json = [{
  "name": "example_field",
  "label": "Example Field",
  "required": false,
  "locked": false,
  "validation_regex": "",
  "allow_new_line": false,
  "show_emoji_picker": false,
  "type": "text",
  "placeholder": "",
  "inline_help_text": "",
  "help_text": "",
  "default": "test",
}]

// Importing our animations.json file
const animations = require('../../json/animations.json');

// Adding Animations.json component into our cusotm json variable. 
// Note that if you import multiple files above -- the order of pushing 
// them onto the json variable makes a difference. 
// You can also use object syntax to control exactly where certain imports
// should be added to the json variable array. 
json.push(animations)

// Exporting the whole json variable so that parachute can build fields.json file properly.
module.exports = {
  getJson: function () {
    return JSON.stringify(json);
  }
}```

```

### Expected fields.json output on bundle

```
[{
  "name": "example_field",
  "label": "Example Field",
  "required": false,
  "locked": false,
  "validation_regex": "",
  "allow_new_line": false,
  "show_emoji_picker": false,
  "type": "text",
  "placeholder": "",
  "inline_help_text": "",
  "help_text": "",
  "default": "test"
}, {
  "children": [{
    "default": false,
    "display": "checkbox",
    "help_text": "Will animate the content/copy section of the header banner",
    "id": "bdb2b290-1647-dc68-941a-fc2ac343938e",
    "label": "Animate Content?",
    "locked": false,
    "name": "animate_content",
    "required": false,
    "type": "boolean"
  }, {
    "choices": [
      ["fade-up", "Fade Up"],
      ["fade-down", "Fade Down"],
      ["fade-left", "Fade Left"],
      ["fade-right", "Fade Right"]
    ],
    "default": "fade-up",
    "display": "select",
    "id": "9a0de064-8dcd-d8f9-2104-c0e318d40afa",
    "label": "Content Animation Direction",
    "locked": false,
    "name": "type",
    "required": false,
    "type": "choice",
    "visibility": {
      "controlling_field": "bdb2b290-1647-dc68-941a-fc2ac343938e",
      "controlling_value_regex": "true",
      "operator": "EQUAL"
    }
  }],
  "default": {
    "animate_content": false,
    "type": "fade-up"
  },
  "expanded": false,
  "id": "3b2b8d46-09e4-dc25-05e3-081a52d52172",
  "label": "Animations",
  "locked": false,
  "name": "animations",
  "required": false,
  "tab": "CONTENT",
  "type": "group"
}]
```