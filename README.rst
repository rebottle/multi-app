Multi-App sites with Bottle.
============================

The Perfect Framework?
-----------------------

Bottlepy is a perfect framework for a simple website, but more debatable is the choice of bottlepy for more complex multi app sites.

The mainsteram current wisdom is that for a more complex website, django is the perfect choice.

I would argue that this conventional wisdom has a flaw.  The flaw is that as site complexity increases, so does the chance a complex predefined structure is not a perfect fit.  Further, comparing the performance of bottle with django, bottle delivers the performance more critical for the complex environment.

Certainly, the chances of an 'off the shelf' solution being available in django is far more likely. If living with the 'off the shelf' as is will work, this is powerful arugment for django.  But if a lot of change will be needed, the pendulum swings back to the cleaner approach of bottle.

OK, bottle is perfect for a simple site.  What holds bottle back for a more complex app?

Multi App Sites
---------------

Lets consider a website comprised of smaller 'subapps'.
First Question: what would Utopia look like?
Second Question: How close does bottle get to utopia?

Utopia.
+++++++

Key Points:
  * The structure for multiple apps has zero impact on single app websites.
  * An app can be built as single app or a 'subapp' component of a multi app site without code changes
  * within a multi app system, each app can be self contained using its own folder structure
  * apps could be nested to any level
  * the complete system is git (or other VCS) friendly

In Practice this would imply something like the following folder structure::

    App (folder)
      file1
      file2
      static (folder)
        static_file1
        static_file2
      views (folder)
        view_file1
        view_file2
      sub_app1 (folder)
        file1
        static (folder)
          static_file1
        views
          view_file1
    etc

The possible contentious point is having statics and views in eash sub_app rather than in their own folder trees.
The above is an 'app tree' approach, and I will call the alternative a views/static tree approach.
I suggest that the logical structure above is utopia for git and development,
but the multi app system should be flexible and support separate image/static trees,
even if not an automatic default.
Single file structure can support both trees through symlinks, but both structures should be workable
with a multi app system both with and without the use of symlinks.

So Far With Bottle.
++++++++++++++++++++

The 'app' structure.
********************
Applications in bottle are instances of the 'Bottle' class. 
Each instance of the bottle module has an AppStack, which is a list of Bottle instances, or a list list of 'apps'.

Both 'app' and 'default_app' reference this AppStack. Import either 'app' or 'default_app' to access the AppStack instance.
This can be somewhat confusing having 'app' as a list as much documentation and general ideas these days descibes an 'app' as an instance of Bottle.
Each one is an app in most language and I recommend importing 'app' as 'apps'.

The AppStack is actually a class based on list, but with two extra methods, 'call' and 'push'.  So app[0]  (or default_app[0]) would
retrieve the fist app in the list, and so on.  Using the call method 'app()' is identical to app[-1] and retries the last app in the list.
The 'push' method, appends a new Bottle instance to the list.
::

  from bottle import app apps, default_app # these are two references to the same 'AppStack' list of Bottle instances
  from bottle import Bottle  # the class to instance 'apps'
  
  app[0]  #retrieve the first Bottle instance in the Appstack list
  app[-1] #retrive the last Bottle instance in the list
  app()  # same as app[-1]
  myApp = Bottle()  #instance a new Bottle application
  app.push(myApp)  #add the new application to the AppStack list
  myApp2 = app.push( Bottle() )  #create a Bottle instance and add to the app list
  myApp3 = app.push()  # same as above.  push() with no parameter instance and pushes a new Bottle instance

Default Bottle instance.
************************
The bottle module not only instances an AppStack() with both the names 'app' and 'default_app',
also one instance of a Bottle object (or app) is pre-added to the AppStack.
Note using this 'pre-added' instance of Bottle() is dangerous,
because serveral modules can accidentally use the same instance.
For the AppStack, there *is* only on instance so no problem.
app() or default_app()  both retrieve the last app added to the AppStack, which will initially be the
Bottle instance created internally to the bottle module.
@route etc decorators will by default use
the last Bottle instance added to the AppStack list.  If using two apps in the same module, @route etc
will by default work with
the automatically created app until a new app is instanced e.g. ::

    from bottle import Bottle, route, app as apps
    
    @route('/page')  # works with default app
    def pageapp1():
        pass
        
    app1=apps()  #save default app -you need to be sure only you will use this!
    newapp = Bottle()  #use 'newapp = apps.push()' to do all steps at once 
    
    @route('/page1')  #route using last app in AppStack which is still app1
    def page1(): pass
    
    @newapp.route('/page2') #explict route for 'newapp'
    def page2():pass
    
    apps.push(newapp)  #add 'newapp' to AppStack, which will make newapp now the default
    
    @route('/page2b')  #another route for newapp
    @newapp.route('/page2c') #explict route for same app
    def page2b():
        pass
        
    app1.route('/anotherpage')  #explicit route for first app
    def pageNot2b():
        pass
        
        

Combining Apps and Routes.
**************************
So even in a single file, it is possible to work with multiple bottle instances or 'apps'. But only one app is actually 'run',
so it is necessary to combine these apps to run collectively.

Bottle provides two ways of combining apps::

    mainapp.mount('/subapp',subapp1)  #mount subapp with '/subapp' as a path prefix
    mainapp.merge(subapp2)  #mount subapp2 at site root
  
If 'subapp1' has a @route('main')  then with the 'mount' above it this 'main' route would become '/subapp/main'.
::

    mainapp.mount('/',subapp)
        and
    mainapp.merge(subapp)

Would seem to be the same, however using 'mount' in this case is forbidden and 'merge' is required.
I am unsure why as it would seem using 'mount' for both cases would be elegant.
::

    #sub app
    from bottle import route,app as apps
    
    myapp= apps.push() 
   
    @route('/main')
    def main():
        return 'the main app page'

Main file::

    #main app
    from bottle import route,mount,run,app as apps
    from subapp import myapp as subapp
    
    myapp=apps.push() #note if both files used apps(), they would share the same app
    
    @route('/')
    @route('/home')
    def home():
        return 'site home page'
        
    myapp.mount('/sub')
    myapp.run()
    
This is simple structure allows for a separate python program for each 'app'.

Note:  using 'apps.push()' in place of 'apps()'  every time means that
there is one unused Bottle instance on the AppStack. But that is better than
accidentally using that one automatic Bottle() twice.

What about folders?
+++++++++++++++++++
The previous section covers all that is needed for multiple applications
where all files share the same folders.  Which effectively means the 'apps' are developed together.
Python files in the same folder, all statics in the same statics folder and all views in the same views folder.
However the 'Utopia' was to allow the subapp to live in its
own folder with self contained static and views folders.
Simply adding an __init__.py to the sub app and adjusting the import allows the sub app to live in its own folder
and a .gitignore can even keep the projects separate if you use Git.

But what about view and statics?
********************************
to be added
