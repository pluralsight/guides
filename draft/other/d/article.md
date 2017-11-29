# Start writing your guide here.



import: 
+ sys


 dd
* bs4
* requests
* json
* urllib.request
* from bs4 import BeautifulSoup
* from PyQt5.QtWebEngineWidgets import QWebEnginePage
* from PyQt5.QtWidgets import QApplication
* from PyQt5.QtCore import QUrl
* import pygal
* from pygal.style import Style
* 

<details open>
<summary>`Resource  Links`</summary>
-
-
-
-
-
-
-

</details>


<br>
```
import sys
import bs4, requests, json, urllib.request
from bs4 import BeautifulSoup
from PyQt5.QtWebEngineWidgets import QWebEnginePage
from PyQt5.QtWidgets import QApplication
from PyQt5.QtCore import QUrl
import pygal
from pygal.style import Style
```
```
class Page(QWebEnginePage):
    def __init__(self, url):
        self.app = QApplication(sys.argv)
        QWebEnginePage.__init__(self)
        self.html = ''
        self.loadFinished.connect(self._on_load_finished)
        self.load(QUrl(url))
        self.app.exec_()

    def _on_load_finished(self):
        self.html = self.toHtml(self.Callable) 
        print('Load finished')

    def Callable(self, html_str):
        self.html = html_str
        self.app.quit()
```