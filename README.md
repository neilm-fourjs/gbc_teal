# GBC NJM Customisation

NOTE: The GBC customization was done using the 1.00.53 release.


## Folders
* Screenshots : The screenshots for this README
* gbc-teal : The files for the custom GBC
* distbin : place for zip of GBC


## Building
This was written and tested on Linux. The Makefile and Script are to setup additional folders and to build the GBC.
* 1. The GBC prerequisites building ( grunt nodejs git ) plus: make & zip installed
* 2. Set GBCPROJECTDIR=<folder containing fjs-gbc-1.00.53-build201905131540-project.zip>
 
```
$ git clone <repo url>
$ cd <folder cloned>
$ make
```

## Screenshots
![ss1](https://github.com/neilm-fourjs/gbc_teal/raw/master/Screenshots/SS-1.png "SS1")
![ss2](https://github.com/neilm-fourjs/gbc_teal/raw/master/Screenshots/SS-2.png "SS2")


## GBC Customizations - CSS

### Customized the colours ( theme.scss.json )

See GBC manual for the theme colour details

### Removed the sidebar ( theme.scss.json )

See GBC manual for the 'gbc-sidebar-always-visible-min-width' values

### Removed the applicationHostMenu ( ApplicationHostWidget.scss )

If you want to turn off the default menu bar completely you can use this:
```css
.gbc_ApplicationHostWidget {
  header.mt-toolbar {
      display: none;
  }
}
```

### Removed the next/previous image space from the folder headings & toolbar ( main.scss )

Folders and toolbar can have next/previous images, but even when not required there is space for those image which can look bad.
```css
.vanished {
  width: 1px;
}
```

### Re-styled the window title bar for modal windows ( MyDialogWindowHeading.scss )

The window title was subtle,  I felt it needed to stand out as a title, so I restyled it.
```css
.gbc_ModalWidget .mt-dialog-header .mt-dialog-title {
  padding-top: 1px;
  fill: $gbc-secondary-text-color;
}

.gbc_ModalWidget .mt-dialog-header .mt-dialog-actions {
  color: $gbc-secondary-text-color;
  fill: $gbc-secondary-text-color;
}

.gbc_ModalWidget .mt-dialog-header {
  background-color: $gbc-primary-medium-color;
  color: $gbc-secondary-text-color;
  fill: $gbc-secondary-text-color;
  padding: 5px 2px 2px 5px;
  margin-bottom: 5px;
}

```

### Table headers to use gbc-primary-light-color for color ( MyTableWidget.scss )

The default table headers were a little bland, so I re-styled them.
```css
.gbc_TableWidget .gbc_TableColumnsHeaders, .gbc_TableWidget .gbc_TableLeftColumnsHeaders, .gbc_TableWidget .gbc_TableRightColumnsHeaders {
        background-color: $theme-primary-background-color;
        color: $palette-text-primary;
}
```

## GBC Customizations - Javascript

### Header text / logo ( MyHeaderBarWidget )
The header title and the logo were done using the method outlined in the GAS manual.
The header MyHeaderBarWidget.tpl.html file was expended to have a table to align the logo/title/app counter
The image is in resources/img folder 

### Footer to be at bottom of page rather than bottom of window ( MyFormWidget )

Here the goal was a footer that was at the bottom of the web page rather then anchored to the bottom of the browser window.
So I only wanted to see the footer when I scroll all the way down.


To make this work the footer has to be part of the 'form'. I created a MyFormWidget.tpl.html of this:
```html
<div>
  <div class="scroller">
    <div class="containerElement">
      <footer>
This is my customized GBC Demo - by neilm@4js.com
      </footer>
    </div>
  </div>
</div>
```
So the Genero 'form' will replace the 'containerElement' and below that will be the 'footer'.
Next we need to create the MyFormWidget.js file to use this tpl.html file.
```javascript
modulum('MyFormWidget', ['FormWidget', 'WidgetFactory'],
  function(context, cls) {
    cls.MyFormWidget = context.oo.Class(cls.FormWidget, function($super) {
      return {
        __name: "MyFormWidget"
      };
    });
    // register the class so only forms with a style of 'gbc_footer' use this widget.
    cls.WidgetFactory.register('Form', 'gbc_footer', cls.MyFormWidget);
});
```
You can see the MyFormWidget.js doesn't actually 'do' anything - so it's inheriting all the methods
from the default 'FormWidget' and overriding nothing. The class is registered with a style though
so only a Form with a style of 'gbc_footer' will get the footer - otherwise all form would get
the footer.

### Change the 'End Session' page ( MySessionEndWidget.js )

See GBC Manaul

### Created a custom toolbar to show data from two custom labels, ie welcome: user and basket values ( MyLabelWidget_stat, MyToolBarWidget )

The idea was that we have a bar at the top of the page with the current user, order totals and default buttons.
This bar should stay at the top of the screen when scrolled.
To do this we created a custom toolBar widget and a custom Label widget. The custom Label widget is used for the information text,
so when I display a value to the custom label it appears in a specific area in the toolBar.

The MyToolBarWidget.tpl.html is a copy of the default but with two additional 'span' elements, *mt-weboeUser* and *mt-weboeStat*
which will contain the content of the custom labels in the Genero form.
```html
  <div class="mt-tab-titles-bar">
    <div class="mt-tab-previous" style="display:none"><i class="zmdi zmdi-chevron-left"></i></div>
    <div class="mt-tab-titles">
        <span class="mt-weboeUser"></span>
        <span class="mt-weboeStat"></span>
        <div class="mt-tab-titles-container containerElement"></div>
      </div>
    <div class="mt-tab-next" style="display:none"><i class="zmdi zmdi-chevron-right"></i></div>
  </div>
```

The MyToolBarWidget.js is inheriting all the methods of the default ToolBarWidget and adding 2 new custom methods, 
one to set the *mt-weboeUser* 'span' element and the other to set the *mt-weboeStat* element.

```javascript
  // this function sets the weboeUser to the passed text value - called from custom label widget.
  setWebOEUser: function(text) {
    this._weboeUser = this._element.getElementsByClassName("mt-weboeUser")[0];
    this._weboeUser.innerHTML = text;
  },
  // this function sets the weboeStat to the passed text value - called from custom label widget.
  setWebOEStat: function(text) {
    this._weboeStat = this._element.getElementsByClassName("mt-weboeStat")[0];
    this._weboeStat.innerHTML = text;
  }
```
The custom toolbar is only used if the Genero toolbar has a style of 'gbc_weboe'.


The MyLabelWidget_stat.js is handling the label on the form, it's specifically not using a custom .tpl.html, this done
by using the'__templateName' entry to point to a default template. In the scss file I use a style of 'display: none;' so the label is not actually displayed in page.

```javascript
    cls.MyLabelWidget_stat = context.oo.Class(cls.LabelWidget, function($super) {
      /** @lends classes.MyLabelWidget_stat.prototype */
      return {
        __name: "MyLabelWidget_stat",
        // using default LabelWidget template!
        __templateName: "LabelWidget",
```


The MyLabelWidget_stat.js is inheriting the methods from the default LabelWidget but overriding the 'setValue' method.
The setValue function is finding the toobarnode in the AUI and getting the helper anchor node for the specific Label,
then getting the 'tag' attribute from the AUI for that label and using that to decide which MyToolBarWidget method to call.

```javascript
setValue: function(value) {
  // Call the parent class setValue function.
  $super.setValue.call(this,value);

  // Get the modelHelper.
  var modelhelper=new cls.ModelHelper( this );
  // Use the modelHelper to get the ToolBar node by first getting the Window Anchor node, then the form, then the toolbar
  var toolbarnode = modelhelper.getAnchorNode().getAncestor("Window").getFirstChild("Form").getFirstChild("ToolBar");

  // Get the tag attribute of the AUI object for this Label;
  var tag = modelhelper.getAnchorNode().getFirstChild("Label").attribute("tag");
  if ( tag == "user") {
    // now we get the controller for the toolbar node, then it's widget, then we can call our custom function.
    toolbarnode.getController().getWidget().setWebOEUser( value );
  };
  if ( tag == "status") {
    // now we get the controller for the toolbar node, then it's widget, then we can call our custom function.
    toolbarnode.getController().getWidget().setWebOEStat( value );
  };
}
```

### Changed toolbar items to be img and text on same line ( MyToolBarItemWidget )

In addition to the toolBar changes above I decided it would look better for the text to follow the image rather than be below it.
This was done in the SCSS file but I only wanted to override the look for my weboe program so I created an empty class of
MyToolBarItem that inherits the methods from the default and uses the default .tpl.html file.

### CSS based layouter - this allows the product tiles in the weboe program to be tiled according to the size of the window. ( CssLayoutEngine, CustCssBoxWidget )

I didn't do this one - it was done by Jean-Philippe from the Strasbourg GBC dev team.

