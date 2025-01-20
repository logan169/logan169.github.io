---
title: 'Custom Maya UI'
tags: Maya cmds Python
---

Augmenting Maya factory UI with new menus, buttons and tool windows is a redundant need for a TD. Most of the time, you'll just need to add a new menu item to your studio menu and trigger a callback from there to launch your brand new tool/script. Using cmds/pymel becomes handy to quickly build up custom Maya specific tool windows.<!--more-->

This being said, I'd highly suggest you to use a Qt based interface (pyside/pyQt) to design UI you'll need to interact with across several DCC. Even then, different DCC may support different Qt based interfaces out of the box. In order to produce Qt code that should run everywhere, you can use the [Qt.py](https://github.com/mottosso/Qt.py) shim done by [mottosso](https://mottosso.com).


## Create a custom window UI obj
```python
import maya.cmds as cmds

cmds.window( title="Demonstration", menuBar=True, widthHeight=(300,200) )
```

## Menu Item Standard
```python
cmds.menu( label='Menu' )
cmds.menuItem( subMenu=True, label='MenuItem1' )
cmds.menuItem( label='SubMenu1', enable=False )
cmds.menuItem( label='SubMenu2', enable=False )
cmds.menuItem( label='SubMenu3' )
cmds.setParent( '..', menu=True ) # set parent to higher level so we stop adding menuItem to Demo

cmds.menuItem( divider=True )
cmds.menuItem( label='MenuItem2' )
cmds.setParent( '..', menu=True )
```

## Menu Item Radio Button
```python
cmds.menu( label='RadioMenu')
cmds.radioMenuItemCollection()
cmds.menuItem( label='Yes', radioButton=False )
cmds.menuItem( label='Maybe', radioButton=False )
cmds.menuItem( label='No', radioButton=True )
cmds.setParent( '..', menu=True )
```

## Menu Item Checkbox
```python
cmds.menu( label='RadioMenu')
cmds.menuItem( label='Top', checkBox=True )
cmds.menuItem( label='Middle', checkBox=False )
cmds.menuItem( label='Bottom', checkBox=True )
cmds.setParent( '..', menu=True )
```
## Menu Item OptionBox & dialog
```python
cmds.menu( label='OptionBox', allowOptionBoxes=True )
cmds.menuItem( label='Option1' )
cmds.menuItem( optionBox=True )

def open_options_window():
	promptInput = cmds.promptDialog(
		title="Option Window",
		message="Enter your lucky Number",
		button=["Ok", "Cancel"],
		defaultButton='Ok',
		cancelButton="Cancel",
		dismissString="Cancel"
		)
	if promptInput == "Ok":
		print  cmds.promptDialog(promptInput, query=True, text=True)
```

## MultiTabs/Regular layout
```python
tab_lay = cmds.tabLayout()
cmds.tabLayout(cmds.columnLayout(), edit=True, tabLabel=[first_tab, 'Column Layout'])
```

## Buttons
```python
cmds.rowLayout(numberOfColumns=3)
cmds.button(label="dummy_args_callback", command=lambda x: button_callback('button was clicked'))
cmds.button(label="print selection", command=lambda x: button_callback(cmds.ls(selection=True)))
b3 = cmds.button(label="edit this button label", command=lambda x: update_button_label())
cmds.setParent('..')

# link buttons/parms together
cmds.rowLayout(numberOfColumns=3)
cmds.text(label="Sphere radius")
sphere_radius = cmds.floatField(minValue=1)
cmds.button(label="Make Sphere", command=lambda x: cmds.polyCube(radius=cmds.floatField(sphere_radius, query=True, value=True)))
cmds.setParent('..')
cmds.setParent('..')
```

## Scrolling layout
```python
cmds.tabLayout(tab_lay, edit=True, tabLabel=[ cmds.scrollLayout(), 'Scroll Layout'])
cmds.columnLayout()

for i in range(20):
	cmds.button(label="Button {}".format(str(i)), command=get_active_tab)
```

## Callbacks function
```python
# Remember that by default, the callback function receive (False,) as argument.
# So either update function definition or use lambda as shown above to provide parm (or not)

## callback functions ##
def button_callback(str_input):
	print str_input
  
# edit button content
def update_button_label():
	cmds.button(b3, edit=True, label="I have a new name")
```

## Retrieve/Switch active UI tab
```python
def get_active_tab():
	print cmds.tabLayout(tab_lay, query=True, selectTabIndex=True)

def switch_active_tab(index):
	cmds.tabLayout(tab_lay, edit=True, selectTabIndex=index)
```

## Show window
```python
cmds.showWindow()
```

## Complete code
```python
import maya.cmds as cmds

# create windows
cmds.window( title="Demonstration", menuBar=True, widthHeight=(300,200) )

# create menu and some menuItems
cmds.menu( label='Menu' )
cmds.menuItem( subMenu=True, label='MenuItem1' )
cmds.menuItem( label='SubMenu1', enable=False )
cmds.menuItem( label='SubMenu2', enable=False )
cmds.menuItem( label='SubMenu3' )
cmds.setParent( '..', menu=True ) # set parent to higher level so we stop adding menuItem to Demo

cmds.menuItem( divider=True )
cmds.menuItem( label='MenuItem2' )
cmds.setParent( '..', menu=True )

# create radio menu
cmds.menu( label='RadioMenu')
cmds.radioMenuItemCollection()
cmds.menuItem( label='Yes', radioButton=False )
cmds.menuItem( label='Maybe', radioButton=False )
cmds.menuItem( label='No', radioButton=True )
cmds.setParent( '..', menu=True )

# create checkbox menu
cmds.menu( label='RadioMenu')
cmds.menuItem( label='Top', checkBox=True )
cmds.menuItem( label='Middle', checkBox=False )
cmds.menuItem( label='Bottom', checkBox=True )
cmds.setParent( '..', menu=True )

cmds.menu( label='OptionBox', allowOptionBoxes=True )
cmds.menuItem( label='Option1' )
cmds.menuItem( optionBox=True )

def open_options_window():
	promptInput = cmds.promptDialog(
		title="Option Window",
		message="Enter your lucky Number",
		button=["Ok", "Cancel"],
		defaultButton='Ok',
		cancelButton="Cancel",
		dismissString="Cancel"
		)
	if promptInput == "Ok":
		print  cmds.promptDialog(promptInput, query=True, text=True)

## UI ##
## make a multiTabs window
tab_lay = cmds.tabLayout()
cmds.tabLayout(cmds.columnLayout(), edit=True, tabLabel=[first_tab, 'Column Layout'])

### Button Demo ###
cmds.rowLayout(numberOfColumns=3)
cmds.button(label="dummy_args_callback", command=lambda x: button_callback('button was clicked'))
cmds.button(label="print selection", command=lambda x: button_callback(cmds.ls(selection=True)))
b3 = cmds.button(label="edit this button label", command=lambda x: update_button_label())
cmds.setParent('..')

# link parms together
cmds.rowLayout(numberOfColumns=3)
cmds.text(label="Sphere radius")
sphere_radius = cmds.floatField(minValue=1)
cmds.button(label="Make Sphere", command=lambda x: cmds.polyCube(radius=cmds.floatField(sphere_radius, query=True, value=True)))
cmds.setParent('..')
cmds.setParent('..')

# second tab is a scrolllayout window
cmds.tabLayout(tab_lay, edit=True, tabLabel=[ cmds.scrollLayout(), 'Scroll Layout'])
cmds.columnLayout()

for i in range(20):
	cmds.button(label="Button {}".format(str(i)), command=get_active_tab)


## callback functions ##
def button_callback(str_input):
	print str_input
  
# edit button content
def update_button_label():
	cmds.button(b3, edit=True, label="I have a new name")

def get_active_tab():
	print cmds.tabLayout(tab_lay, query=True, selectTabIndex=True)

def switch_active_tab(index):
	cmds.tabLayout(tab_lay, edit=True, selectTabIndex=index)
####

#show window
cmds.showWindow()
```

## Creating Menu in Maya

The following code would produce menu in a new separated window. In order to add those to the Maya menu, you'd need to retrieve the main Maya windows variable as follow and pass it as the parent arg. 
```python
from pymel.core import MelGlobals

main_window = MelGlobals()['gMainWindow'] # retrieve main maya window to be passed as tool window's parent arg
cmds.menu('Demo', parent=main_window)
```

## Executing Menu script at start time
Custom menus creation could be automated in the UserSetup.py to be done after Maya's boot sequence with 
executeDeffered, like:
```python
import maya.utils as utils
utils.executeDeferred('MenuSetupScript()')
```
