Libraries metadata registry
===========================

**This library holds information about libraries and what files they contain.**

The data is used in the plugin [gulp-libraries](https://github.com/timroes/gulp-libraries)
to retrieve information about libraries that doesn't bring their own `metadata.json` file.

How do I use this?
------------------

You normally don't need to use this directly at all. See the 
[gulp-libraries](https://github.com/timroes/gulp-libraries) project
for documentation on how to use the plugin.

How can I add data?
-------------------

**You got a message that you should head here, because of missing metadata?**

You tried to use a library for which there isn't any metadata (neither in this registry
nor in the library itself). You have two choices now:

* Add it yourself and send a pull request
* Open a ticket here and wait for me to add it

_In the first case:_ Thank you a lot! Just read further to learn about the structure
of the project, add the metadata and send a pull request. If the pull request looks
good I try to merge it as soon as possible for the library to work properly with
`gulp-libraries`.

_In the second case:_ Create a new issue in this project and state the library (if
possible the bower link directly, but at least its id) and the version you where trying
to use. I will try to add appropriate metadata for this library as soon as possible
(though it most likely will need longer, then accepting just a pull request).


I want to add my own library
----------------------------

If you have your own library you should put a `metadata.json` (see below)
directly into the root of your bower project for it to work. That is the
preferred way, so you can always keep the data in that file in sync with
the current state of your project.

Nevertheless there is a good reason why you might want to add files for your
own library here: _backward compatible_.

If you start adding a `metadata.json` to your project today, you might still
want to add `metadata.json` files for older versions of your library here.

How is this project structured?
-------------------------------

To add metadata add the name of library (i.e. its bower id) as a folder to the `data` folder
of this project. Inside this package folder you can add folders for the versions you
want to deliver. The version folders must be a semantic versions and can replace parts
of this version by 'x' from the end, which work like wildcards. Inside the version folder
there must be the file `metadata.json` which holds the metadata (see below for its format).

Let's explain with an example: Your library is named _mylib_. If the metadata for version 1.2.3
needs to be looked up, the following files will be tried until one is found, that exists:

1. `data/mylib/1.2.3/metadata.json`
2. `data/mylib/1.2.x/metadata.json`
3. `data/mylib/1.x.x/metadata.json`
4. `data/mylib/x.x.x/metadata.json`

Always try to use the most general version possible. Since most likely the structure of
your project won't change during a major version, you most likely want to create the
`data/mylib/1.x.x/metadata.json` in this repository to place the information for all versions
under major version 1.

The metadata.json
-----------------

The `metadata.json` file cotains information about what files are contained in the project
and needs to be copied over to a project using this library. A very simple matadata file could
look like:

```
{
  "js": [ "dist/jquery.min.js" ]
}
```

If the user requests all JavaScript files of all libraries now, the file named
`jquery.min.js` inside the `dist` folder of the bower installed library will be added
to that file list. You can add as many files as you want inside the array. You can also
use some simple wildcards in the file names.

To add another type of files just add another key to the object, e.g. to add some css files:

```
{
  "js": [ "dist/mylib.min.js" ],
  "css": [ "dist/mylib.min.css" ]
}
```

You basically can add any type, that is not a reserved keyword (see below). There are some
that should be used for common file types, because the user only fill get a list of your files
when requesting that specific type. Of course you could name a type `foobar`, but the user would
need to request files of type `foobar` to get these files, which he isn't most likely doing.

Some of the most common types in this file are:

type | description      
-----|------------------
js   | JavaScript files 
css  | CSS files
less | LESS files
scss | SASS files

### Reserved keywords (i.e. more options)

There are currently two reserved keywords that cannot be used as types, because they have a
special meanings, explained here:

#### `modules`

Some libraries have some optional features (e.g. themes when having CSS), that your user won't
need everytime using your library. These can be defined as modules in the metadata:

```
{
  "js": [ "mylib.min.js" ],
  "css": [ "common.min.css" ],
  "modules": {
    "theme-default": {
      "css": [ "theme-default.min.css" ]
    },
    "theme-light": {
      "css": [ "theme-light.min.css" ]
    }
  }
}
```

The modules key must hold an object (not an array). Each key in this object specifies a
module. The value of each module must be an object again. This object can specify any kind
of files again (as you can in the root object). If the user just includes your library he will
only get the files defined in the root of the metadata information. He can enable any of the
modules in their configuration. In that case the files defined in that module will be returned
in addition to the ones defined in the root of the metadata.

#### `options`

The `options` key hold an object of further configuration about this module. Currently
this options object can contain the following keys:

```
{
  "options": {
    "after": [ "pkgid", "pkgid2" ]
  }
}
```

The `after` option is an array of strings. These strings are bower ids of other libraries.
If a user include your library and one one of the others library, this option defines, that
your library always needs to be added *after* the other package to any file stream. If the user
does *not* have the library defined in *after* in their project nothing will happen. This is
just an ordering hint and nothing more.

By default all bower dependencies in the library are handled like *afters* in the metadata,
i.e. if your bower library has a bower dependency on `jquery` you don't need to specify
in your `metadata.json`, that you want to be loaded after jquery. This will be done
automatically due to the bower dependency.

You only need to specify *after* for soft dependencies. For example if you don't require `jquery`,
but if it is available you will make use of it (e.g. AngularJS does this), specify it in your
*after* and your library will be loaded after jquery **if** it is available.
