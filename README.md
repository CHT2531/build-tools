# Using NPM Scripts as a build tool

Download the repository. From the *src* folder open the *index.html* page in a browser and check it works (it is a very simply HTML page with some attached CSS).

> A note about 'slash' characters. When using the Command prompt or terminal, most operating systems use a forward slash between filenames e.g. *node_modules/.bin/lessc*. Windows use a backslash i.e. *node_modules\.bin\lessc*. The backslash character is also the escape character so if we have a file path as part of a string we write "node_modules\\.bin\\lessc". The following instructions are written for windows. If you are using a Mac, use forward slashes instead. 

## Running less as an NPM script
In the root of this folder create a new Node.js project

```
npm init -y
```

Next we will install *less* so that we can use it in our project. Enter the following:

```
npm install less --save-dev

```

Check this has worked by doing the following

1. Open *src/less/style.less* in a text editor
2. Make a simple change e.g. change the hex value of the colour
3. Back in the Node.js command prompt, enter the following:

```
node_modules\.bin\lessc src/less/style.less src/css/style.css
```

* Refresh the page in a browser to test this has worked. 
* Now, instead of typing the *less* command directly we are going to use an NPM script to run the *less* command. Open the *package.json* file and change it so that looks like the following:

```
{
  "name": "buildtest",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

* The key change from the default package.json file is that we have added a script 

```
  "scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css"
  },
```

The script name is *less* and the the script itself is simply the command to run less. 

* Save *package.json*
* Back in the command prompt enter the following:

```
npm run less
```

* You should get confirmation that your less command has run.
* Make another change to your less file.
* Run the script again
```
npm run less
```
* Refresh the browser to check this has worked. 

## Watching for changes with the *onchange* package
Instead of switching between the command prompt and your text editor, we can add another script that will watch the *.less* file and run the *less* commmand whenever the *.less* file changes. 

* First we need to install another package. In the command prompt enter the following:

```
npm install onchange --save-dev
```

The *onchange* package will watch files for us and then run a script e.g.

```
node_modules\.bin\onchange src/less/style.less -- npm run less
```

Whenever *style.less* changes we run the less script. We also want this to be a script, so modify the script section of the package.json file to include this and save your changes

```
"scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css",
    "watch-css": "node_modules/.bin/onchange src/less/style.less -- npm run less"
  },
```

* Back in the command prompt, enter the following:

```
npm run watch-css
```
* Now make a change to *style.less* and save it (you should get some feedback in the command prompt)
* Refresh the web browser to see your changes. 
* To stop watching the *.less* file hit *CTRL+c*

## Automatically update the page in the browser 
This works fine for our *.less* file, but we can improve our workflow by watching for changes to any files and then automatically refreshing the page in a browser. To do this we will use a package call *browser-sync*.

To install browser-sync
```
npm install browser-sync --save-dev
```

Next, in the command prompt we will tell browser-sync to start a web server and watch our css file and html file. 

```
node_modules\.bin\browser-sync start --server "src" --files "src/css/style.css, src/index.html"
```

A browser should open and your *index.html* page should be displayed. 
* The *--server* option specifes which folder to act as the root for the web server.
* The *--files* option specifies which files should be watched for changes.

To test this works:
* Make a change to your HTML page.
* Save the file. The page should update in the browser automatically. 
* Make a change to the CSS file, again the page should update automatically. 
* In the command prompt window hit *CTRL+c* to stop browser-sync. 

Next let's make this into a script. Modify the package.json file to include a browser-sync script.

```
"scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css",
      "watch-css": "node_modules/.bin/onchange src/less/style.less -- npm run less",
    "browser-sync":"browser-sync start --server \"src\" --files \"src/css/style.css, src/index.html\""
  },
```
* To test this works
```
npm run browser-sync
```

## Running tasks in parallel with parallelshell
We now have two tasks
* One that watches less files and generates css. 
* A second task that watches for changes to the html or css files and updates the page in a browser. 

It would be nice to run both these tasks at the same time. To do this install another node package 
```
npm install -g parallelshell
```
This package allows us to run two scripts at the same time. 
* Add a new script in package.json, watch...

```
  "scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css",
      "watch-css": "node_modules/.bin/onchange src/less/style.less -- npm run less",
      "browser-sync":"browser-sync start --server \"src\" --files \"src/css/style.css, src/index.html\"",
      "watch":"parallelshell \"npm run watch-css\" \"npm run browser-sync\""
  },
```
* It should be fairly obvious what *watch* does, it simply runs *watch-css* and *browser-sync* in parallel. 

* In the node.js command prompt enter the following:

```
npm run watch
```

Now whenever you make a change to any of the files, the changes should be automatically reflected in the browser. 

## Creating a build version
A build version is simply a version of your site that is ready to be used by users. A build version of the site will differ from your developer version in a number of ways. For example a build version will contain minified files and compressed images to make the pages load as fast as possible. We will do a simple example that minifies our CSS. 

We are going to put our build version in the folder named *dist*. 

Add the following *move* script 
```
"scripts": {
    "less": "node_modules/.bin/lessc src/less/style.less src/css/style.css",
      "watch-css": "node_modules/.bin/onchange src/less/style.less -- npm run less",
      "browser-sync":"browser-sync start --server \"src\" --files \"src/css/style.css, src/index.html\"",
      "watch":"parallelshell \"npm run watch-css\" \"npm run browser-sync\"",
      "move": "copy src\\index.html dist\\"
  },
```
Open the command prompt and test it works:
```
npm run move
```

You should find that the *index.html* file has been moved into your *dist* folder. Next, we are going to put a minified copy of our CSS into the *dist* folder. 

* Install the *clean-css* package to minify our CSS
```
npm install -g clean-css-cli
```
* Next, to see what it does, run it from the command prompt
```
cleancss -o "src\css\style.min.css" "src\css\style.css"
```
* Open the file *style.min.css*, see how the CSS has been minified.
* Next, we'll add this as a script that will take the css file from the *src* folder, minify it and put it into the *dist* folder.

```
"scripts": {
    "less": "lessc src\\less\\style.less src\\css\\style.css",
    "watch-css":"onchange \"src\\less\\style.less\" -- npm run less",
    "browser-sync":"browser-sync start --server \"src\" --files \"src/css/style.css, src/index.html\"",
    "watch":"parallelshell \"npm run watch-css\" \"npm run browser-sync\"",
    "move": "copy src\\index.html dist\\",
    "minify": "cleancss -o \"dist/css/style.css\" \"src/css/style.css\""
  },
```

* Open up the command prompt and check this works.

* Finally we will add a *build* command that will run *move* and then run *modify*. 

```
  "scripts": {
    "less": "lessc src\\less\\style.less src\\css\\style.css",
    "watch-css":"onchange \"src\\less\\style.less\" -- npm run less",
    "browser-sync":"browser-sync start --server \"src\" --files \"src\\css\\style.css, src\\index.html\"",
    "watch":"parallelshell \"npm run watch-css\" \"npm run browser-sync\"",
    "move": "copy src\\index.html dist\\",
    "minify": "cleancss -o \"dist\\css\\style.css\" \"src\\css\\style.css\"",
    "build":"npm run move && npm run minify"
  },

```
* Again, test this works:
```
npm run build
```
* This single command should generate our user ready version of the site.

Typically there would be lots more steps we would include in a build process e.g.
* Compressing image files e.g. image-min (https://github.com/imagemin/imagemin-cli)
* Linting, bundling and uglifying our JavaScript.

These are things you can explore in your assignment.

## Learning more
* https://css-tricks.com/why-npm-scripts/ 
* https://medium.freecodecamp.org/why-i-left-gulp-and-grunt-for-npm-scripts-3d6853dd22b8
* https://scotch.io/tutorials/using-npm-as-a-build-tool

