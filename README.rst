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

Routes.
*******
 
Routes using @app.route() can take a prefix.  This means a sub app can with no recoding adopt a url prefix.
