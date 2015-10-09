# A pruposal for the new structure of Riiskit

### Terms
---           | ---
------------- | -------------
The plugin    | The new proposed plugin to replace the parent theme
The theme     | The skeleton theme to be used with the plugin
The developer | The developer using the child theme to develop a theme
should        | Required as a code standard, but not enforced
may           | optional
must          | Required, and fatally fails if not upheld


## A new plugin & theme structure
In favor of the precent "parent theme and child theme", the suggestion is a plugin & theme structure.
Where the plugin (now Riiskit Parrent) would provide functionality that the theme can use.


### Reasons for doing this

	1. Parent themes should be used for providing templates that the child can extends, and seing as we do not do this, there is no real reason for the theme to be a plugin
	2. With new proposed functionality, the now parent theme would have to have access to the child theme and its files. This is not recomended, and "deep links" the two. Plugins on the other hand are known to access files and such from themes. This makes unit testing a lot easier, as the plugin->theme method is well known.
	3. Alot of the new proposed functionality is considered "plugin functionality", read; functionality that typically lives inside of plugins. This last point is basically "we should follow the masses, there is probably a reason for them doing it that way

# The theme

The theme should not contain any functionality at all. This makes updating possible. Once you start developing a child theme, updating is impossible. On the other hand, if all functionality is moved into the new plugin, updating is easy peasy lemon squezzy.

The new child theme would only contain **very** basic files, and a directory structure. The pruposed structure of the child theme is

<pre>
templates/                  # Contains TWIG templates
template-controller/        # Contains TWIG tepmlate controllers
core/                       # Conrtains autoloaded namespaced PHP
    model/                  # Contains PHP classes meant to access the DB
        postTypeWrappers/   # Contains classes extending the PostTypeWrapper class from *the plugin*
post-types.json             # Contains a JSON file defining post types.
res/                        # Contains resources that will be accessable from twig
    images/                 # Contains images
    image-res/              # Contains images in different resoulutions. These are auto-generated
    js/                     # Contains compiled javascript
    css/                    # Contains compiled css
src/                        # (Optional) contained uncompiled resources. (sass, TypeScript, ES6, what have you)
phpunit.xml                 # PHPUnit config file
tests/                      # Contains PHPUnit tests
    bootstrap.php           # PHPUnit bootstrap file
    wordpress-test-library  # Contains the Wordpress test library
    wordpress               # Contains a full copy of wordpress, used in UnitTesting
    tests/                  # Contains test classes
config.json                 # Contains config for this theme, described below
.travis                     # A preconfigured Travis file. Optional, but should be included to force a structure that would be compatible with .travis.
composer.json               # Would from the get go include "phpunit" and "codesniffer"
</pre>

## Desciprtion

### templates/

Twig is a templating engine. This is where the templates will live

### tempalte-controlles/

Each twig template will have a maching template controller here. This is how Twig works.
The template controllers job is to provide the data that the template needs(custom post data etc.)
If no special data is needed, the template controller still needs to be present.

### core/

This directory contains PHP code. All of the code **must** be namespace, and **must** match the directory structure.

example structur
<pre>
core/
    MyPhpClass.php //Contains a PHP class or interface with the name of myTheme\MyPhpClass
    someDir/
        MyOtherClass.php //Contains a phpclass or interface with the name myTheme\someDir\MyOtherClass
</pre>

`myTheme` is used as the "root namespace" in this examplte. The "root namespace" is chosen in the [config.json file](#configjson)

### core/model/

Contains all PHP classes meant for getting data from the DB or from anywhere else for that matter. This pattern is not enforced, but highly recomended.


### core/model/postTypeWrappers/

**must** only contain classes that extend the PostTypeWrapper class from *the plugin* These clases are used for fetching data from custom post types

### post-types.json

A JSON file that defines custom post type. The structure in the JSON file matches the parameter list in the wordpress function insert_post_type exactly, EXCEPT: the post name(not lable) is used as the key in the json file

The names are automatically prefixed with the prefix chosen in the [config.json file](#configjson)

```js
{
    "post-type-name" : {
        "supports" : ["title", "content"],
        ///etc.
    }
}
```

### res/

Contains resources that can be fetched by {{res}} in templates. {{res}} custom *twig tag* that creates a url form the resource passed in.

**Example:**
```twig
<img src="{{ res "js/myfile.js" }}">
``` 

**Would become:**
```html
<img src="http://example.com/wp-content/themes/mytheme/res/js/myfile.js">
```

### res/images

Contains images which can be retrived via the {{img}} custom *twig tag*.

**Example:**
```twig
<img src="{{ img "logo.png" size:200x200 }}">
``` 

**Would become:**
```html
<img src="http://example.com/wp-content/themes/mytheme/res/image-res/logi.png@200x200">
```
**or if the device is retina**
```html
<img src="http://example.com/wp-content/themes/mytheme/res/image-res/logi.png@200x200@x2">
```

### res/css

Contains compiled css. Nothing special about this directory, css is fetched via the `{{res}}` custom twig tag

### res/requirejs-config.js

Contains (optionally) a starting point for the require config. This file is required if the `modules` field is set to `requirejs` in the [config.json file](#configjson)

### src/

Should contain uncompiled src code. Nothing special about this directory

### phpunit.xml

Contains the PHPUnit configuration. The devleop should never have to touch this.

### tests/

Contains PHPUnit tests things(<--formal language ftw)

### tests/bootstrap.php

Contains the PHPUnit bootstrap code. The devleop should never have to touch this.

### tests/wordpress-test-library/

Contains the wordpress test library.

### tests/wordpress/

Contains a full copy of wordpress. For testing purposes

### tests/tests/

Contains the PHPUnit test cases

### config.json

Contains configs that the developer can choose.
These are

```js
{
    "prefix" : "prefix_", // The prefix used in this theme
    "php" : {
        "namespace" : "rootNameSpace", // The root namespace for the core/ directory
    },
    "js" : {
        "modules"   : "none", //requirejs or none. If an js module loader should be included
        "framework" : "meteor", //If an app framework should be included
    },
}
```

### .travis

A preconfigured .travis file. Travis may not be used on all projects but
    a) Travis is a pain to setup, it should be preconfigured even if it's used only a few times
    b) The structure should be "travis compatible", as making it "travis compatible" means that you have remove deep links, and make it easy to run tests for *the developer*
    
### composer.json

Of the bat, includes "phpunit" and "codesniffer" as dev dependencies.

# The plugin

The plugin is responsible for *all* functionality derectly related to Riiskit.
this includes.
1. Reading the config file
2. Including the module loader or js framework if one is chosen
3. Autoloading the core/ directory in the theme
4. Providing the PHP library (Which contains the PhpPostWrapper class for instance)
5. Providing the Twig templateing engine
6. Providing the Twig extension (That implements {{res}} and {{img}}


### PHP Library

The classes and functions provided by the build in PHP librARY

<pre>
riiskit\\model\PostTypeWrapper #The PostTypeWrapper super class
</pre>

### JS Library

Requesting help...

### Twig Extension

The tags, filters and functions for twig provided by the built in Twig extension

**res - tag**
Builds a resource URL

**img - tag**
Builds an image url. Can tage size:Width x Height as parameter.
If this parameter is provided, it will look in the image-res/ directory
to see if the image requested exists in that size, if not, it will generate one, and a retina version(with double the size)


# Discussion

This was just a proposal. Feel free to add issues, send pull requests or dismiss this all together.

If this looks okay so far, there a few things unanswered.

### How should the directory structre of the plugin be?

It is important to have good design, to make it easlily maintainable. This is a longterm prosject, small choices can have a huge impact.

### Is there more functionality that should be provided in the plugin?
### Is there functionality that is unnecesary and should be removed?
### How can we implement this to make a usable version as quickly as possible?
It's important to stress that we should be as quick as possible doing this. So we need a development plann which should include which features are important and should be included in the first version, and which features can wait.

