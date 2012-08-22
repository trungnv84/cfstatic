---
layout: default
title: Full Useage Guide
---

# {{ page.title }}

1. [Preparing your static files](#preparation)
2. [Configuration](#configuration)
3. [API Useage](#useage)
4. [LESS](#less)
5. [CoffeeScript](#coffeescript)

<a id="preparation"></a>
## Preparing your static files

In order for CfStatic to know how to include and package your files in the correct order and with all the necessary dependencies, it needs to know how your javascript and css files relate to one another. The framework provides two methods of doing this:

1. JavaDoc style comments within each file
2. A single 'dependencies' text file

###JavaDoc style comments

JavaDoc comments look like this and must be present at the top of your files for CfStatic to process them:

{% highlight js %}
/**
 * Note the slash and double star to start the comments. The first paragraph of text
 * within the comment block is a free-text description (i.e. this text). CfStatic
 * will ignore this description but it can/should be used to document your code.
 * Key value pair properties are then defined using the @ symbol:
 *
 * @property property value
 * @property another property value
 * @anotherproperty yet another property value
 */
{% endhighlight %}

CfStatic makes use of the following properties:

* **@depends** used to indicate a file dependency (see below), e.g. `@depends /core/jquery.js`
* **@minified** used to indicate that a file is already minified (do `@minified true`), CfStatic will not then re-minify the file
* **@ie** used to indicate an Internet Explorer restriction for the file, e.g. `@ie LTE IE 8`
* **@media** for CSS files only, used to indicate the target media for the CSS file, e.g. `@media print`

### Documenting dependencies

Core to the correct running of CfStatic is the use of the @depends property to document dependencies between your files; CfStatic uses this information to ensure all necessary files are included in your page, and in the correct order. Dependencies can be either local or external, e.g. `@depends http://someurl.com/somejs.js` is an external dependency.

Local dependencies have a path that starts at the root folder of the type of file you are dealing with. i.e. If your javascript files all live at `/webroot/static/js/`, and the file `/webroot/static/js/plugins/myplugin.js` has a dependency on `/webroot/static/js/core/jquery.js`, that dependency would be written like so:

**myplugin.js**

{% highlight js %}
/**
 * myplugin does this and that...
 *
 * @depends /core/jquery.js
 * @author joe bloggs
 */
(function($){
     // my plugin code here...
})(jQuery);
{% endhighlight %}

#### Already minified files

Some of your local files may already have been minified or you may not want certain files to be minified by CfStatic. By setting the **@minified** property to true (i.e. `@minified true`), CfStatic will *not* put the file through the minifier. The file will still be renamed and moved to the output directory however.

#### Internet Explorer specific includes

These can be declared using the **@ie** property and static files with this property set will be wrapped in Conditional Comments when output as includes. Examples:

{% highlight js %}
/**
 * This is my IE 6 only print stylesheet
 *
 * @ie IE 6
 * @media print
 * @depends /core/layout.css
 */
{% endhighlight %}

{% highlight js %}
/**
 * This is my IE 8 and below stylesheet
 *
 * @ie LT IE 8
 * @depends /core/layout.css
 */
{% endhighlight %}

See here for conditional comment reference:

[http://msdn.microsoft.com/en-us/library/ms537512(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/ms537512\(v=vs.85\).aspx)

<a id="configuration"></a>
## Configuration

### Argument reference

The CfStatic init() method takes the following arguments. Do not be alarmed at the number of them, only two are mandatory, the rest have sensible defaults:

    staticDirectory:     Full path to the directory in which static files
                         reside (e.g. /webroot/static/)

    staticUrl:           Url that maps to the static directory (e.g.
                         http://mysite.com/static or /static)

    jsDirectory:         Relative path to the directoy in which javascript
                         files reside. Relative to static path. Default is
                         'js'

    cssDirectory:        Relative path to the directoy in which css files
                         reside. Relative to static path. Default is 'css'

    outputDirectory:     Relative path to the directory in which minified
                         files will be output. Relative to static path.
                         Default is 'min'

    minifyMode:          The minify mode. Options are: 'none', 'file',
                         'package' or 'all'. Default is 'package'.
                         downloadExternals:   If set to true, CfMinify will
                         download and minify locally any external
                         dependencies (e.g.
                         http://code.jquery.com/jquery-1.6.1.min,js). Default = false

    debugAllowed:        Whether or not debug is allowed. Defaulting to
                         true, even though this may seem like a dev setting.
                         No real extra load is made on the server by a user
                         making use of debug mode and it is useful by default.
                         Default = true.

    debugKey:            URL parameter name used to invoke debugging (if
                         enabled). Default = 'debug'

    debugPassword:       URL parameter value used to invoke debugging (if
                         enabled). Default = 'true'

    forceCompilation:    Whether or not to check for updated files before
                         compiling (true = do not check). Default = false.

    checkForUpdates:     Whether or not to attempt recompilation on every
                         request. Default = false

    includeAllByDefault: Whether or not to include all static files in a
                         request when the .include() method is never called
                         (default = true) *0.2.2*

    embedCssImages:      Either 'none', 'all' or a regular expression to
                         select css images that should be embedded in css
                         files as base64 encoded strings, e.g. '\.gif$' for
                         only gifs or '.*' for all images (default = 'none')
                         *0.3.0*

    includePattern:      Regex pattern indicating css and javascript files to
                         be included in CfStatic's processing. Defaults to .*
                         (all) *0.4.0*

    excludePattern:      Regex pattern indicating css and javascript files to
                         be excluded from CfStatic's processing. Defaults to
                         blank (exclude none) *0.4.0*

    outputCharset:       Character set to use when writing outputted minified
                         files *0.4.0*

    javaLoaderScope:     The scope in which instances of JavaLoader libraries
                         for the compilers should be persisted, either
                         'application' or 'server' (default is 'server' to
                         prevent JavaLoader memory leaks). You may need to use
                         'application' in a shared hosting environment *0.4.1*

    lessGlobals:         Comma separated list of .LESS files to import when
                         processing all .LESS files. Files will be included in
                         the order of the list *0.4.2*

### Static paths
The minimal setup, ready for production, involves declaring your root static directory and the url that maps to it. This assumes that you have 'js', 'css' and 'min' folders beneath your 'staticDirectory'. For example, consider the following directory structure:

    ./
    ../
    includes/
        css/
        js/
        min/
    Application.cfc

The minimal configuration might look like this:

**Application.cfc**

{% highlight cfm %}
<cfscript>
    application.cfstatic = CreateObject('component', 'org.cfstatic.CfStatic').init(
          staticDirectory = ExpandPath('/includes')
        , staticUrl       = "/includes"
    );
</cfscript>
{% endhighlight %}

Another example, this time with non-default folder names for the static files, and css and javascript having a folder to themselves in the root:

    ./
    ../
    styles/
    javascript/
    compiled/
    index.cfm
    Application.cfc

In this case, your configuration might look like:

**Application.cfc**

{% highlight cfm %}
<cfscript>
    application.cfstatic = CreateObject('component', 'org.cfstatic.CfStatic').init(
          staticDirectory = ExpandPath('./')
        , staticUrl       = "/"
        , jsDirectory     = 'javascript'
        , cssDirectory    = 'styles'
        , outputDirectory = 'compiled'
    );
</cfscript>
{% endhighlight %}

### Minify modes

Minify modes control how CfStatic minifies your files. The options are:

* **None**: no minification (i.e. only use CfStatic for the request inclusion and dependency handling framework )
* **File**: files are minified but not concatenated in any way
* **Package** (default): files are minified and concatenated in packages (all files in the same folder are considered a package)
* **All**: files are minified and concatenated into a single JavaScript file and a single CSS file

Use the *minifyMode* argument to set this behaviour. For a more in-depth look at minify modes, see my blog post here:

[http://fusion.dominicwatson.co.uk/2011/09/understanding-the-package-minify-mode-in-cfstatic.html](http://fusion.dominicwatson.co.uk/2011/09/understanding-the-package-minify-mode-in-cfstatic.html)

### External dependencies

External dependencies can be declared in your static files with `@depends http://www.somsite.com/somefile.js`. CfStatic can either download these dependencies for local minification and concatenation, or it can simply output an include to reference the file remotely. This might be a good idea for popular libraries hosted on CDNs such as jQuery. However, if you're creating an intranet application, it may be wise to ensure that all files are served from your own server.

Use the *downloadExternals* argument to set this behaviour.

### Debug mode

By default, you can make CfStatic output includes for your source files rather than the minified files, by providing the url parameter, `debug=false`. You can turn this behaviour off altogether with the *debugAllowed* argument, or you can set the url parameter name and value with the *debugKey* and *debugPassword* parameters.

### Monitoring for updates

CfStatic can be set up to monitor your static files for changes, recompiling when it finds them. This is really useful for local development though is turned *off* by default. Use the *checkForUpdates* argument to set this behaviour.

Also by default, CfStatic will *not* recompile on instantiation if it finds that there have been no changes to your files. You can change this behaviour, forcing recompiling on instantiation (e.g. on application start), by using the *forceCompilation* argument.

A combination of the two arguments, e.g. `checkForUpdates=true` and `forceCompilation=true`, will force recompiling on every request.

### Including all files by default (from 0.2.2)

Out of the box, CfStatic will include *all* your static files if you never use the .include() method to specifically pick files to include (i.e. you just do .renderIncludes()). You can change this behaviour by setting `includeAllByDefault` to false.


### Embedding CSS Images (from 0.3.0)

The `embedCssImages` option allows you to instruct CfStatic to embed CSS images as data URIs in the compiled CSS files (see <http://en.wikipedia.org/wiki/Data_URI_scheme>).

The default option of '`none`' performs no image embedding. Passing '`all`' will instruct CfStatic to embed *all* CSS images. Finally, you can pass a regular expression to specify which images will be chosen for embedding.

For more information, see the blog post here:

[http://fusion.dominicwatson.co.uk/2012/01/cfstatic---embedding-css-images.html](http://fusion.dominicwatson.co.uk/2012/01/cfstatic---embedding-css-images.html)

### Excluding resources from the engine (from 0.4.0)

For whatever reason, you may wish to have CfStatic overlook certain CSS, LESS or JavaScript files. From *0.4.0* there are two arguments that will allow you to do just that, `includePattern` and `excludePattern`. By combining these two arguments that both accept regex patterns, you can have full control over what CfStatic will include. For example, you may wish to exclude some Global LESS files:

      includePattern = '.*'
    , excludePattern = '.*/lessGlobals/.*'

Or only include any resources under a `raw` folder that do not contain an underscore:

      includePattern = '.*/raw/.*'
    , excludePattern = '.*_.*'


<a id="useage"></a>
## API Useage

Once you have configured CfStatic and marked up your static files with the appropriate dependency documentation, you arrive at the pleasing point of having very little left to do. The CfStatic API provides 4 methods:

1. **init( *options* )**: used to get an instance of Cfstatic
2. **include( *resource* )**: used to instruct CfStatic that a particular file or package (folder) is required for this request
3. **includeData( *data* )**: used to output data to a JavaScript variable when the javascript is rendered
4. **renderIncludes( *[type]* )**: used to render the CSS or JavaScript includes

### Creating an instance of the API for your application

The API is published in the component, `org.cfstatic.CfStatic`. An instance should be created using the component's init() method, passing in any [configuration](#configuration) arguments for your environment, and stored in a cacheable scope. An example, without using any framework, might look like this:

**Application.cfc**

{% highlight cfm %}
<cfscript>
    function onApplicationStart() {
        application.cfstatic = CreateObject('component', 'org.cfstatic.CfStatic').init(
            staticDirectory = ExpandPath('./static')
          , staticUrl       = "/static/"
        );
    }
</cfscript>
{% endhighlight %}

All CfStatic operations can now be performed using this instance, e.g.

**MyLayout.cfm**

{% highlight cfm %}
        ...
        #application.cfstatic.renderIncludes( 'css' )#
    </head>
{% endhighlight %}


### Include( *required string resource* )

You can use this method to include an entire package or a single file in the request page (a package is a folder of files, not including sub-folders). Paths start at the root static directory, so the following are all valid:

{% highlight cfm %}
<cfscript>
    // include the layout.css file
    cfStatic.include('/css/core/layout.css');

    // include the 'core' css package (note the trailing slash on the directory name)
    cfStatic.include('/css/core/');

    // include a bunch of js packages and files, chaining the method call
    cfStatic.include('/js/core/')
            .include('/js/core/ie-only/')
            .include('/js/plugins/timers.js')
            .include('/js/pagespecific/homepage/');
</cfscript>
{% endhighlight %}

#### Including non existent packages or files

If you try to include a package or file that does not exist, no error will be thrown and CfStatic will not render an include for that package or file. This is to allow you to create dynamic includes based on any rules you like. For instance, you might want to try to include page specific css when it is available, something like:

{% highlight cfm %}
<cfscript>
     // where request.pageName is some variable set by your application:
    cfStatic.include('/css/pageSpecific/#request.pageName#/');
</cfscript>
{% endhighlight %}

You can then include page specific css simply by creating a directory of css files with the appropriate name.

#### Don't worry about the order

You can include your static files in any order you like and you can include the same files multiple times; CfStatic will take care of rendering them in the correct order and once each only.


### IncludeData( *required struct data* )

You can use the IncludeData method to make CF data available to your JavaScript. The following example illustrates its usage:

**someColdFusionFile.cfm**

{% highlight cfm %}
<cfscript>
    data                    = StructNew();
    data['userColorChoice'] = session.user.prefs.colorChoice;
    data['fu']              = 'bar';
    data.watchMyCase        = 'sensitive isn't it?';

    cfStatic.includeData( data )
            .includeData( {bar="fu"} ); // you can chain me too
</cfscript>
{% endhighlight %}

**Outputted JavaScript before all includes**

{% highlight js %}
    var cfrequest = {
         "userColorChoice" : "#FF99FF"
       , "fu"              : "bar"
       , "WATCHMYCASE"     : "sensitive isn't it?"
       , "BAR"             : "fu"
    };

    // you now have access to cfrequest.userColorChoice, etc. from within your javacsript
{% endhighlight %}

#### Beware of case sensitivity

JavaScript is case sensitive and ColdFusion uppercases all struct keys that are not declared using array notation. Hopefully, the example above illustrates this.

### RenderIncludes( *[string type]* )

This method returns the necessary html to include your static files. The type argument is optional and should be set to either 'CSS' or 'JS' when passed. Here are a couple of examples:

**ExampleLayout1.cfm**

{% highlight cfm %}
    <html>
        <head>
            ...
            <!-- render all css and js includes (css first) -->
            #cfStatic.renderIncludes()#
        </head>
        <body>
            ...
        </body>
    </html>
{% endhighlight %}

**ExampleLayout2.cfm**
{% highlight cfm %}
    <html>
        <head>
            ...
            <!-- render css in the head of the page -->
            #cfStatic.renderIncludes('css')#
        </head>
        <body>
            ...
            <!-- render js at the bottom of the page -->
            #cfStatic.renderIncludes('js')#
        </body>
    </html>
{% endhighlight %}


That's it!


<a id="less"></a>
## LESS
### CfStatic now supports compiling of `.less` files!

If you've not heard of LESS CSS, head on over to [http://lesscss.org/](http://lesscss.org/) and fall to your knees in humble awe (and get all coder giddy).

In CfStatic, simply create .less files with LESS css in them in exactly the same way you create .css files for CfStatic. CfStatic will take your .less files and compile them as css, saving the output to `yourfile.less.css`. It will then minify that compiled css file in accordance with the rules you configure.

Neat eh?

<a id="coffeescript"></a>
## CoffeeScript

CfStatic will compile any `.coffee` files in your JavaScript directories, converting them to JavaScript files with the `.coffee.js` extension. A couple of things to note:

### Formatting of JavaDoc comments
JS comments are not valid CoffeeScript. To markup your CoffeeScript files ready for CfStatic, use the following format:

{% highlight js %}
    ###*
    * This is my coffeescript file, its really neat.
    *
    * @depends /some/file.coffee.js
    *
    ###
{% endhighlight %}

### Bare mode
By default, CoffeeScript will wrap the compiled `.js` in an anonymous function call to ensure no leaked variables:

{% highlight js %}
    (function(){
        // your compiled js here
    })();
{% endhighlight %}

If you do not want this behaviour, CoffeeScript offers a "bare mode" switch so that the anonymous function wrapper is not included (which they do not recommend). In CfStatic, simply name your CoffeeScript files with the `.bare.coffee` extension to have them compiled in bare mode. -->