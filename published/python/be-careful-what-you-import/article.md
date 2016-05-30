I learned a valuable lesson recently about importing into a [Python](http://python.org) application. Typically I just find the module I need, import it, and use it without any issues. However, I found out that an [unnecessary import](http://pyqt.sourceforge.net/Docs/PyQt4/modules.html) can sometimes lead to a big waste of memory.

### Searching imports dynamically

I encountered this problem while writing a [tool](https://gist.github.com/durden/4723305) that intended to search a [Python](http://python.org) [module](http://docs.python.org/2/tutorial/modules.html) and/or [package](http://docs.python.org/2/tutorial/modules.html#packages) for a given object. The script is interesting and worth another post all on it's own. The [gist](https://gist.github.com/durden/4723305): you provide the script with two arguments, the module/package to search and a term to search for. The script should then give you a listing of all the places it found an object containing your search term.

For example, the documentation for
[PyQt4](http://www.riverbankcomputing.com/software/pyqt/intro) can be pretty
tough to search. I tend to use the standard [Qt](http://qt.digia.com/)
documentation and then translate any examples I find into code for the
[Python PyQt4 wrapper](http://www.riverbankcomputing.com/software/pyqt/intro).

### Duplicates?

I was testing my script with various terms I tend to use in PyQt4 applications
such as `MinimumExpanding` and `NoEditTriggers`:

    python import_searcher.py -s MinimumExpanding

    PyQt4.Qt.QSizePolicy.MinimumExpanding = 3
    PyQt4.QtGui.QSizePolicy.MinimumExpanding = 3
    PyQt4.Qwt5.qplt.QSizePolicy.MinimumExpanding = 3

Notice anything odd about the above output? Looks like `MinimumExpanding`
shows up in two almost identical locations -- `PyQt4.Qt` and `PyQt.QtGui`.
Naturally, I thought there was a bug in my script, but after some debugging and
reading I found the following jewel on the
[PyQt4 wikipedia page](http://en.wikipedia.org/wiki/PyQt):

> The Qt module consolidates the classes contained in all of the modules
> described above into a single module. This has the advantage that you don't
> have to worry about which underlying module contains a particular class. It
> has the disadvantage that it loads the whole of the Qt framework, thereby
> increasing the memory footprint of an application. Whether you use this
> consolidated module, or the individual component modules is down to personal
> taste.

So, using the `PyQt4.Qt` module is actually redundant and only for convenience.
As a result, we actually import a lot of bulky and extraneous modules, such as
[QtDesigner](http://pyqt.sourceforge.net/Docs/PyQt4/qtdesigner.html),
[QtWebKit](http://pyqt.sourceforge.net/Docs/PyQt4/qtwebkit.html), and
[QtHelp](http://pyqt.sourceforge.net/Docs/PyQt4/qtnetwork.html).

For example, assume that your application is only using the `PyQt4.QtGui` module, and you mistakenly decide that you need something from `PyQt4.Qt`. This single
import could essentially **double your memory usage** [1]:

    Line # Mem usage Increment Line Contents
    ================================================
        1 @profile
        2 def import_qt_module_by_module():
        3 #from PyQt4 import QtCore
        4 #from PyQt4 import QtDBus
        5 6.953 MB 0.000 MB #from PyQt4 import QtDeclarative
        6 12.844 MB 5.891 MB from PyQt4 import QtGui
        7 #from PyQt4 import QtHelp
        8 #from PyQt4 import QtMultimedia
        9 #from PyQt4 import QtNetwork
        10 #from PyQt4 import QtOpenGL
        11 #from PyQt4 import QtScript
        12 #from PyQt4 import QtScriptTools
        13 #from PyQt4 import QtSql
        14 #from PyQt4 import QtSvg
        15 #from PyQt4 import QtTest
        16 #from PyQt4 import QtWebKit
        17 #from PyQt4 import QtXml
        18 #from PyQt4 import QtXmlPatterns
        19 #from PyQt4 import phonon
        20 #from PyQt4 import QtAssitant
        21 #from PyQt4 import QtDesigner
        22 #from PyQt4 import QtAxContainer
        23 19.387 MB 6.543 MB from PyQt4 import Qt

Note that just importing 'PyQt4.Qt' increased the application memory usage by
**6.543 MB**. This could be costly depending on how much memory your
application was already using and how much your system has available. While this situation deals with smaller figures, multiple unnecessary imports can easily compound, leading to more significant memory usage issues.  

### Moral takeaways

1. Be careful of what you import. It could be redundant, costly, or
   both.

2. Read documentation carefully. The
   [main PyQt4 documentation](http://pyqt.sourceforge.net/Docs/PyQt4/modules.html)
   alludes to this redundant `PyQt4.Qt` module by describing it with the
   following, "**Consolidates all other modules into a single module for ease
   of use at the expense of memory.**"

3. Remember to try stuff, play around, and have some fun coding. A one-off script can lead to a nice, useful    discovery.
____
[1] You can profile this for yourself with this
    [gist](https://gist.github.com/durden/4956774).

