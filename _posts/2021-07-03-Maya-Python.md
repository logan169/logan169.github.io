---
title: 'Maya Python Programming'
tags: Maya cmds
---

Python is an object oriented programmation language (OOP) that has been created by Guido von Rossum in 1981. In 2021, Python have a considerable community behind it and is one of the most used programming language around the world and in the CG/pipeline fields. In this blog, we will look at how we can use Python to interact with Maya.<!--more--> Since maya was created, different flavors of python interfaces has been released as Pymel, Mayapy and OpenMaya. While they are all supposed to be able to achieve globaly the same result they differs in their philosophy.

## Environnement Setup

There is several way to setup env variables in Maya.

Using sys python package directly from code editor or userSetup.py
```python
import sys
sys.append("/path_to_scripts_folder")
```

Adding Maya env variables either in system global var or in [Maya.env](https://knowledge.autodesk.com/support/maya/learn-explore/caas/CloudHelp/cloudhelp/2020/ENU/Maya-EnvVar/files/GUID-8EFB1AC1-ED7D-4099-9EEE-624097872C04-htm.html) (Maya.env location depends on MAYA_APP_DIR env var)
see the [documentation](https://knowledge.autodesk.com/support/maya/learn-explore/caas/CloudHelp/cloudhelp/2020/ENU/Maya-EnvVar/files/GUID-925EB3B5-1839-45ED-AA2E-3184E3A45AC7-htm.html) for more information.
```bash
SHARED_MAYA_DIR = HostName:/usr/localhome/public/maya/<version>
MAYA_SCRIPT_PATH = $SHARED_MAYA_DIR/scripts:$MAYA_APP_DIR/scripts/custom
MAYA_PLUG_IN_PATH = $SHARED_MAYA_DIR/plug-ins
TMPDIR = /disk2/tempspace #override tmp folder
```

Adding a path files (.pth) to maya python site-packages installation folder.
This will add to sys.path all folders mentionned in the .pth file when maya python is runned. 
Other alternative is to manually add the folder containing the .pth file using [site](https://docs.python.org/3/library/site.html) python package in userSetup.py
```python
import site
site.addsitedir("path_to_pth_file_folder") # Warning: should be the folder path 
```

Using [Maya modules](https://help.autodesk.com/view/MAYAUL/2017/ENU/?guid=__files_GUID_130A3F57_2A5D_4E56_B066_6B86F68EEA22_htm)
{%- include extensions/youtube.html id='2xVPTHMg9C0' -%}

## Batch mode init (Maya headless)
```bash
maya -batch -file someMayaFile.mb -command "file -save"
```

## Using Maya.cmds in batch mode
```python
from maya import cmds, standalone
standalone.initialize() # this is only needed in batch mode
```

## Help
```python
cmds.help('polyCube', doc=True, language='python')
```

## Reload updated python modules
```python
import mySuperScript
reload(mySuperScript)
```

## Run Mel Command
```python
from maya import mel

def run_mel_script():
	mel.eval("source myMelScript;")
	mel.eval("myMelScript;")
```

## Get vertex/edge/face numbers
```python
# create a sphere
transform_node, shape_node = cmds.polySphere()

vertex_number = cmds.polyEvaluate(shape_node, vertex=True)
edge_number = cmds.polyEvaluate(shape_node, edge=True)
face_number = cmds.polyEvaluate(shape_node, face=True)
```

## Get vertex/edge/face numbers
```python
# create a sphere
transform_node, shape_node = cmds.polySphere()

vertex_number = cmds.polyEvaluate(shape_node, vertex=True)
edge_number = cmds.polyEvaluate(shape_node, edge=True)
face_number = cmds.polyEvaluate(shape_node, face=True)
```

## Select vertex/edge/face
```python
# create a sphere
transform_node, shape_node = cmds.polySphere()

cmds.select(shape_node + ".vtx[0:5]", replace=True) # vertex
cmds.select(shape_node + ".e[0:5]", replace=True) #edge
cmds.select(shape_node + ".f[0:5]", replace=True) #face
```

## Move edges
```python
# create a sphere
transform_node, shape_node = cmds.polySphere()

cmds.select(shape_node + ".e[0]", replace=True) 
cmds.move(1.0, 2.0, 3.0, relative=True) 

# alternative use polyMoveVertex 
cmds.polyMoveVertex(shape_node + ".e[0]", t=[1.0, 2.0, 3.0]) # this is slower than cmds.move
```

## Move edges
```python
cmds.ls(selection=True) # select a sphere
cmds.xform(pivot=(0,0,0)) # move pivot at origin
cmds.makeIdentity(apply=True) # freeze transforms so object starts at 0, 0, 0
```

## List Relatives/Get child/parent nodes
```python
cmds.listRelatives(children=True, allDescendents=True) # get child nodes
cmds.listRelatives(parent=True) # get direct parent
cmds.listRelatives(parent=True, fullPath=True) # get fullpath

```

## Create curves
```python
points = []
points.append( (-0.5, 0 , 0) )
points.append( (   0, 0 , 0) )
points.append( ( 0.5, 0 , 0) )

cmds.curve(degree=1, p=points)
```

## Create faces
```python
points = []
points.append( (-0.5, 0 , 0) )
points.append( (   0, 0 , 0) )
points.append( ( 0.5, 0 , 0) )
points.append( (-0.5, 0 , 0) )

cmds.polyCreateFacet(p=points)
```

## Triangulate/Quadrangulate faces
```python
points = []
points.append( (-0.5, 0 , 0) )
points.append( (   0, 0 , 0) )
points.append( ( 0.5, 0 , 0) )
points.append( (-0.5, 0 , 0) )

cmds.polyCreateFacet(p=points)
# triangulate newly created face
cmds.polyTriangulate() 

# quadrangulate from a triangulated face, Warning this doesn't always works well
cmds.polyQuad() 
```

## Create a polygon
```python
f1 = [ (0, 0 , 0), ( 1, 0 , 0), ( 1, 1 , 0), (0, 1 , 0)]
f2 = [ (0, 0 , 0), ( 0, 0 ,1), ( 0, 1 , 1), (0, 1 , 0)]
f3 = [ (0, 0 , 1), (0, 1 , 1), ( 1, 1 ,0), (1, 0 , 0)]

faces = [
	cmds.polyCreateFacet( p=f1, texture=1),
	cmds.polyCreateFacet( p=f2, texture=1),
	cmds.polyCreateFacet( p=f3, texture=1)
	]


cmds.select( faces[1], faces[2], faces[3], replace=True )

# unite polygons
obj = cmds.polyUnite()

# merge redundant vertexes
cmds.select(obj[0] + ".vtx[:]")
cmds.polyMergeVertex(distance=0.0001)
```

## getAttr & setAttr
```python

# get all obj non default attributes 
custom_user_attrs = cmds.listAttr(obj, userDefined=True)

# add a new custom attr
my_new_attr = "custom_attr"
if my_new_attr not in custom_user_attrs:
	cmds.addAttr(
			obj,
			longName=my_new_attr,
			keyable=True
		)

# set attr
my_new_value = 3
cmds.setAttr( "{}.{}".format(obj, my_new_attr), my_new_value)

# get attr
my_new_Attr_value = cmds.getAttr("{}.{}".format(obj, my_new_attr))

```

## Construction history
```python
history = cmds.listHistory(obj) # Not super reliable as it can be deleted by user
cmds.delete(ch=True) # Delete construction history
```

## Querying UV data
```python
transform, shape = cmds.ls(selection=True)
uvs = cmds.polyEvaluate(shape, uvComponent=True)
uvPos = cmds.polyEditUV(shape + ".map[0]", query=True)

print "Num UVs => {} \nPos uv => {}".format(str(uvs), uvPos)
```

## Get edges' uv/vertexes
```python
elt1 = cmds.polyListComponentConversion(shape + ".e[0]", fromEdge=True, toUV=True) # retrieve edge uv 
elt2 = cmds.polyListComponentConversion(shape + ".e[0]", fromEdge=True, toVertex=True) # retrieve edge vertexes
```

## Laying out UV
```python
cmds.polyProjection(shape + ".f[*]", type="planar") # type value could be either planar / cylindrical / spherical
cmds.polyProjection(shape + ".f[0:50]", shape + ".f[75:80]", type="planar") # could receive several inputs
```

## Create shading node network
```python
# Creating a toon shader network
shader_node = cmds.shadingNode('blinn', asShader=True)
ramp_node = cmds.shadingNode('ramp', asTexture=True)
sample_node = cmds.shadingNode('samplerInfo', asUtility=True)

cmds.setAttr(ramp_node + ".interpolation", 0)
cmds.setAttr(ramp_node + ".colorEntryList[0].position", 0)
cmds.setAttr(ramp_node + ".colorEntryList[1].position", 0.3)

cmds.setAttr(ramp_node + ".colorEntryList[0].color", 0,0,0, type="float3") # float3 is because we provide 3 floats input
cmds.setAttr(ramp_node + ".colorEntryList[1].color", 1,0,0, type="float3") # float3 is because we provide 3 float input

cmds.connectAttr(sample_node + ".facingRatio", ramp_node + ".vCoord")
cmds.connectAttr(ramp_node + ".outColor", shader_node + ".color")
```

## Connect all shading nodes attributes at once
```python
# Creating a File network
file_tex = cmds.shadingNode('file', asTexture=True)
place_tex = cmds.shadingNode('place2dTexture', asUtility=True)

cmds.defaultNavigation(connectToExisting=True, source=place_tex, destination=file_tex)
```

## Disconnect shading Node
```python
# following previous File network code
cmds.disconnectAttr(place_tex + ".offset", file_tex + ".offset")
```

## Check if node is a geo
```python
def is_geo_generator(obj):

	geo_type = ["mesh", "nurbsSurface", "subdiv"] 
	shape = cmds.listRelatives(obj, shapes=True)[0]
	shape_type = cmds.nodeType(shape) # retrieve node type where geo nodeType being in geo_type
	if shape_type in geo_type:
		return True
	return False
```

## Set default shader to unshaded obj
```python
def is_shaded(obj):
	cmds.select(obj, replace=True)
	cmds.hyperShade(obj, shaderNetworksSelectionMaterialNodes=True)
	result = cmds.ls(selection=True)

	if result:
		return False
	return True

# Get unshaded obj
transforms = cmds.ls(type="transform")
unshaded_nodes = [ node for node in transforms if is_shaded(node) is False]

# create shader
blinn_shader = cmds.shadingNode("blinn", asShader=True)
cmds.setAttr(blinn_shader + ".color", 0, 1, 1, type="double3")

# Set shader obj
for unshaded_node in unshaded_nodes:
	cmds.select(unshaded, replace=True)
	cmds.hyperShade(assign=blinn_shader)
```

## Found nodes that are assigned a shader
```python
# following previous code
def get_obj_from_shader(shader):
	cmds.hyperShade(objects=shader)
	objects = cmds.ls(selection=True)
	return objects

blinn_shaded_nodes = get_obj_from_shader( blinn_shader )
```

## Use shading nodes with 3D geo data

![avgNode](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/maya_python/avgNodeMaya.gif)

```python
# use a plus minus average shading node to keep an object at the center
def center():
	objs = cmds.ls(selection=True)
	if len(objs) < 3:
		cmds.error("Please select at least 3 nodes and retry!")

	center_obj = objs.pop()
	avg_node = cmds.shadingNode('plusMinusAverage', asUtility=True)
	cmds.setAttr(avg_node + ".operation", 3) #set operation to average

	px, py, pz = [0,0,0]
	for i, obj in enumerate(objs):
		cmds.connectAttr(obj + ".translateX", avg_node + ".input3D[{}].input3Dx".format(i)) 
		cmds.connectAttr(obj + ".translateY", avg_node + ".input3D[{}].input3Dy".format(i)) 
		cmds.connectAttr(obj + ".translateZ", avg_node + ".input3D[{}].input3Dz".format(i)) 
    
	cmds.connectAttr(avg_node + ".output3D.output3Dx", center_obj + ".translateX")
	cmds.connectAttr(avg_node + ".output3D.output3Dy", center_obj + ".translateY")
	cmds.connectAttr(avg_node + ".output3D.output3Dz", center_obj + ".translateZ")

cmds.select("pSphere1", "pSphere2", "pCylinder1", replace=True)
center()
```

## Create joints
```python

root = cmds.joint(name="root", position =[2.5, -1, 0])
for i1 in range(5):
	for i2 in range(3):
		cmds.joint(name="root_finger{}_joint{}".format(i1, i2), position=(i1*2, i2*3, 0) )
	cmds.select(root, replace=True)

# cmds.pickWalk(direction="Up") # could be use to travel trhough Maya hierarchy
```

## Create set-driven key 

{%- include extensions/youtube.html id='WsFbhpiVIXk' -%}

```python
def set_driven_key(objs, attr, min_driver_val, max_driver_val, min_driven_val, max_driven_val):
	
	# set driver attr
	driver_joint = objs[0]
	driver_attr = driver_joint + attr
	original_driver_val = cmds.getAttr(driver_attr)

	childrens = cmds.listRelatives(children=True, allDescendents=True)
	for child in childrens:
		driven_attr = child + attr

		# set driven key for max and min
		cmds.setAttr(driver_attr, min_driver_val)
		cmds.setDrivenKeyframe(driven_attr, cd=driver_attr, value=min_driven_val, driver_value=min_driver_val)

		cmds.setAttr(driver_attr, max_driver_val)
		cmds.setDrivenKeyframe(driven_attr, cd=driver_attr, value=max_driven_val, driver_value=max_driver_val)

	cmds.setAttr(driver_attr, original_driver_val)

objs = cmds.ls(selection=True)
set_driven_key(
	objs=objs,
	attr=".rotateX",
	min_driver_val=0,
	max_driver_val=30,
	min_driven_val=0,
	max_driven_val=30
	)

```

## Creating custom attributes
```python
# regular attr
cmds.addAttr(
	obj,
	shortName="my_custom_regular_attr",
	longName="my_custom_regular_attr",
	defaultValue=0,
	minValue=-1,
	maxValue=1,
	keyable=True
	)

# enum attr
cmds.addAttr(
	obj,
	shortName="my_custom_enum_attr",
	longName="my_custom_enum_attr",
	attributeType="enum",
	enumName="Val1:Val2:Val3", # enum options separated by ':'
	keyable=True
	)

# bool attr
cmds.addAttr(
	obj,
	shortName="my_custom_bool_attr",
	longName="my_custom_bool_attr",
	attributeType="bool",
	keyable=True
	)

# color attr
cmds.attr(
	obj,
	shortName="my_custom_color_attr",
	longName="my_custom_color_attr",
	attributeType="float3",
	usedAsColor=True
	)

for color in ["colorR","colorG","colorB"]:
	cmds.addAttr(
		obj,
		shortName="{}_attr".format(color),
		longName="{}_attr".format(color),
		attributeType="float",
		parent="my_custom_color_attr"
		)
```

## Locking attributes
```python
attr = obj + ".rotateX"
cmds.setAttr(attr, edit=True, lock=True, keyable=True, channelBox=False)
```

## Create locators
```python
cmds.spaceLocator(name="loc1")
```

## Replace locator per a joint node
```python
loc1 = cmds.spaceLocator(name="loc1")
# ... some code
# transform loc1 position
cmds.xform(loc1, absolute=True, translation=(1,0,0))

# get loc1 position
pos = cmds.xform(loc1, query=True, translation=True, worldSpace=True)

# create a joint at this place
cmds.joint(name="joint@loc1", position=pos)
```

## Set inverse kinematic Handle (IK)
```python
cmds.ikHandle(startJoint=hip_joint, endEffector=ankleJoint)
```

## Limit joints rotations
```python
cmds.setAttr(joint_node + ".jointTypeX", 0) # disable rotation around x axis
cmds.setAttr(joint_node + ".jointTypeZ", 0) # disable rotation around z axis

# enable rotation on Y axis between -45 to 45 degree only
cmds.transformLimits(joint_node, rotationY=(-45, 45), enableRotationY=(1,1))

# enable rotation on Y axis between -45 degree without upper limit
cmds.transformLimits(joint_node, rotationY=(-45, 45), enableRotationY=(1,0))

# enable rotation on Y axis between any value up to 45 degree max
cmds.transformLimits(joint_node, rotationY=(0, 45), enableRotationY=(0,1))
```

## Get obj's keyframes informations
```python
obj = cmds.ls(selection=True)[0]
animated_attributes = cmds.listAnimatable(obj)
shot_frame_range=50

for animated_attribute in animated_attributes:

	keyframe_count = cmds.keyframe(animated_attribute, query=True, keyframeCount=True)

	if keyframe_count:
		#index flag is only used if we don't want full shot frame range
		times = cmds.keyframe(animated_attribute, query=True, index=(0,shot_frame_range), timeChange=True) 
		values = cmds.keyframe(animated_attribute, query=True, index=(0,shot_frame_range), valueChange=True)

		print "{}.{}@{} = {}".format(obj, animated_attribute, times, values)

```

## Animation layers

{%- include extensions/youtube.html id='QLUdaLrL3Eo' -%}

```python
def create_anim_layer(layer_name):
	animation_layers=cmds.animLayer(query=True, root=True)

	# check if layer already exists
	if animation_layers:
		sub_anim_layers = cmds.animLayer(animation_layers, query=True, children=True)

		if sub_anim_layers and layer_name in sub_anim_layers:
			print "Error: a layer with this name already exists"
			return

	cmds.animLayer(layer_name)

# create layer
mylayer = create_anim_layer('mylayer')

# create a sphere and assigned it the layer
transform_node, shape_node = cmds.polySphere()
frame=1010
cmds.select(transform_node, replace=True)
# add all animatable attr, use attribute flag to specify the attr
cmds.animLayer(transform_node, edit=True, addSelectedObjects=True) 
cmds.setKeyframe(transform_node + ".translateY", value=yVal, time=frame, animLayer="mylayer")

```

## Setting keyframe
```python
# set a keyframe at 1005 for tx=1
cmds.setKeyframe(
	transform_node + ".translateX",
	value=1,
	time=1005,
	inTangentType="linear", # optional
	outTangentType="linear" # optional
	) 
```

## Transfer animation between objects
```python
# make a lambda function to split long name and retrieve attr name
get_attr_name = lambda longname: longname.split('.')[-1]

def transfer_anim():
	objs = cmds.ls(selection=True)

	if len(objs < 2):
		print "Error: need at least 2 items to transfer animation from 1st selected item to others"
		return

	driver = objs[0]
	animated_attributes = cmds.listAnimatable(driver)
	for animated_attribute in animated_attributes:
		keyframes = cmds.keyframe(animated_attribute, query=True, keyframeCount=True)

		if keyframes:
			cmds.copyKey(animated_attribute)
			for driven_obj in objs[1:]:
				# we can specify a animLayer flag to pasteKey to not overwrite keys data
				cmds.pasteKey(
					driven_obj,
					attribute= get_attr_name(animated_attribute),
					option="replace"
					)
```

## Create expression
```python
def make_expression(node, attr, value):
	# expression strings needs to be written with mel syntax
	return "{}.{} = {}".format(node,attr,value)

expression_str = make_expression(transform_node, "translateX", "cos(time/2)*10")
cmds.expression(string=expression_str)
```

## Create a light rig
```python
# create a 3 points light rig
def create_light_rig(offset=10, rotation=30):

	# need this since cmds.spotLight returns the light instead of the transform
	keylight = cmds.listRelatives(
		cmds.spotLight(rgb=(1,1,1), name="keylight"),
		parent=True
		)[0]


	filllight = cmds.listRelatives(
		cmds.spotLight(rgb=(0.8,0.8,0.8), name="filllight"),
		parent=True
		)[0] 

	backlight = cmds.listRelatives(
		cmds.spotLight(rgb=(0.2,0.2,0.2), name="backlight"),
		parent=True
		)[0] 

	# keylight
	cmds.move(0,0,offset, keylight)
	cmds.move(0,0,0, keylight + ".rotatePivot")
	cmds.rotate(-rotation, rotation, 0, keylight)

	# filllight
	cmds.move(0,0,offset, filllight)
	cmds.move(0,0,0, filllight + ".rotatePivot")
	cmds.rotate(-rotation, -rotation, 0, filllight)

	# backlight
	cmds.move(0,0,offset, backlight)
	cmds.move(0,0,0, backlight + ".rotatePivot")
	cmds.rotate(180 + rotation, 0, 0, backlight)

	light_rig = cmds.group(empty=True, name="light_rig")
	for light in [keylight, filllight, backlight]:
		cmds.parent(light, light_rig)

	cmds.select(light_rig, replace=True)


create_light_rig()
```

## Create a light rig controllers UI

[light controller UI](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/maya_python/lightControllerUI.PNG)

```python
class LightControllersUI():

	def __init__(self):

		self._ui_name = "Lightcontrollers"
		self.lights = []
		self.light_controls = []

		self.init_ui()

	def delete_old_windows(self):
		if cmds.window(self._ui_name, exists=True):
			cmds.deleteUI(self._ui_name)

	def init_ui(self):

		# delete old windows if one exists
		self.delete_old_windows()

		self.win = cmds.window(self._ui_name, title=self._ui_name)
		cmds.columnLayout()

		scene_lights = cmds.ls(lights=True)
		for light_index, scene_light in enumerate(scene_lights):
			self.add_light_controls(light_index, scene_light)

		cmds.showWindow(self.win)

	def update_color_slider(self, light_index):

		new_color_slider = cmds.colorSliderGrp(
			self.light_controls[light_index],
			query = True,
			rgb = True
			)

		cmds.setAttr(
			self.lights[light_index] + ".color", 
			new_color_slider[0],
			new_color_slider[1],
			new_color_slider[2],
			type="float3"
			)

	def add_light_controls(self, light_index, light_shape):
		
		light_transform = cmds.listRelatives(light_shape, parent=True)[0]

		light_color = cmds.getAttr(light_shape + ".color")
		color_slider = cmds.colorSliderGrp(
			label = light_transform,
			rgb = light_color[0],
			changeCommand = lambda x: self.update_color_slider(light_index)
			)

		self.lights.append(light_shape)
		self.light_controls.append(color_slider)

LightControllersUI()
```

##  Create a camera rig
```python
def create_cam_rig(offset=10):

    aim_target = cmds.spaceLocator()
    
    for i in range(4):
    	cam = cmds.camera()
    	cmds.aimConstraint(aim_target[0], cam[0], aimVector=(0,0,-1))
    
    	pos_x = -offset if i%2 == 0 else offset 
    	pos_y = 6
    	pos_z = -offset if i>=2 else offset
    
    	cmds.move(pos_x, pos_y, pos_z, cam[0])

create_cam_rig()
```

## DG nodes manipulation

{%- include extensions/youtube.html id='cSaQM03RfFU' -%}


## Maya's command line render

{%- include extensions/youtube.html id='ys8x5rwObKs' -%}

## Arnold render Setup Python API and commands

maya documentation [here](https://knowledge.autodesk.com/support/maya/learn-explore/caas/CloudHelp/cloudhelp/2020/ENU/Maya-Rendering/files/GUID-FFC3298A-9803-4EAF-B472-D13247319EB0-htm.html)

## Internal var
```python
user_workspace_dir = cmds.internalVar(userWorkspaceDir=True)
user_script_dir = cmds.internalVar(userScriptDir=True)
```


## FileDialog2
```python
import maya.cmds as cmds


# fileMode flag decide what the dialog is to return.
# 0 Any file, whether it exists or not.
# 1 A single existing file.
# 2 The name of a directory. Both directories and files are displayed in the dialog.
# 3 The name of a directory. Only directories are displayed in the dialog.
# 4 Then names of one or more existing files.

## Read #######################################################################################################

basicFilter = "*.mb"
cmds.fileDialog2(fileMode=1, fileFilter=basicFilter, dialogStyle=2)

singleFilter = "All Files (*.*)"
cmds.fileDialog2(fileMode=1, fileFilter=singleFilter, dialogStyle=2)

multipleFilters = "Maya Files (*.ma *.mb);;Maya ASCII (*.ma);;Maya Binary (*.mb);;All Files (*.*)"
cmds.fileDialog2(fileMode=1, fileFilter=multipleFilters, dialogStyle=2)

multipleFilters = "Maya Files (*.ma *.mb);;Maya ASCII (*.ma);;Maya Binary (*.mb);;All Files (*.*)"
user_workspace_dir = cmds.internalVar(userWorkspaceDir=True)
cmds.fileDialog2(fileMode=1, fileFilter=multipleFilters, startingDirectory=user_workspace_dir, dialogStyle=2)
###############################################################################################################
## Write ######################################################################################################

basicFilter = "*.mb"
cmds.fileDialog2(fileMode=0, fileFilter=basicFilter, startingDirectory=user_workspace_dir, dialogStyle=2)

###############################################################################################################
```

## Maya viewport contexts
```python

# get current context 
current_context = cmds.currentCtx()

# set current context
cmds.setToolTo('selectSuperContext')
cmds.setToolTo ('polySelectEditContext')
cmds.setToolTo ('polySlideEdgeContext')
cmds.setToolTo( 'moveSuperContext' )

# reset context
cmds.ctxAbort()
```

## Maya custom contexts
```python
# be stored in a customCtx.py file in scripts folder
# to be runned from a shelftool
def startCtx():
    print ('Running init context')
    
def finalizeCtx():
    objs = cmds.ls(selection=True)
    
    x,y,z = [0,0,0]
    
    for obj in objs:
        pos = cmds.xform(obj, query=True, worldSpace=True, translation=True)
        x += pos[0]
        y += pos[1]
        z += pos[2]
        
    x /= len(objs)
    y /= len(objs)
    z /= len(objs)
    
    loc = cmds.spaceLocator()
    move(x,y,z,loc)
    
def createContext():
    toolStartStr = 'python ("customCtx.startCtx()")'
    toolFinishStr = 'python ("customCtx.finalizeCtx")'
    
    
    # i1 flag could be provided if we want an icons
    # just need 32px x 32px icon in the icon folder
    newCtx = cmds.scriptCtx(
        title="avg context",
        setNoSelectionPrompt="Select at least 2 objs",
        toolStart=toolStartStr,
        finalCommandScript=toolFinishStr,
        totalSelectionSets=1,
        setSelectionCount=2,
        setAllowExcessCount=True,
        setAutoComplete=False,
        toolCursorType="create",
        
    )
        
    cmds.setToolTo(newCtx)
createContext()
```

## Create script job

Maya script jobs are like houdini parm callback and can be super usefull for a couple of use cases: switching asset variation, ik/fk rig switch, ...

[scriptJobs](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/maya_python/scriptJob.gif)

```python
# small variation switcher proof of concept
def create_variation(variation_type):
    
    x,y,z = [0,0,0]
    try:
        # node should always be there
        x,y,z = cmds.xform('myAsset', query=True, absolute=True, translation=True)
        cmds.delete('myAsset')
    except:
        pass
    
    if variation_type == "cube":
        obj = cmds.polyCube(name="myAsset")
        variation_index = 0
    else:
        obj = cmds.polySphere(name="myAsset")
        variation_index = 1
    
    cmds.move(x,y,z, worldSpace=True)
    
    shape_node = cmds.listRelatives(shapes=True)[0]
    cmds.addAttr(
        shape_node,
        shortName="Variation",
        longName="variation",
        attributeType="enum",
        enumName="cube:sphere",
        defaultValue=variation_index,
        keyable=False
    )
    
    cmds.scriptJob(attributeChange=["myAsset.variation", update_variation], killWithScene=True)

def update_variation():

    variation = cmds.getAttr("myAssetShape.variation" )

    variation_type = [ 'cube', 'sphere' ][variation]
    print '::', variation_type
    create_variation(variation_type)
    
# init scene
create_variation("sphere")

```

Maya use script jobs under the hood to make the interface interactive so messing with script jobs might produce unresponsiveness/crashes. It is super important to ensure we aren't duplicating script jobs each time the code is being runned. Ensure your script job is not either protected or permanent

```python
# retrieve all script jobs
scriptjobs = cmds.scriptJobs(listJobs=True)

# remove all script jobs => will mess with maya
cmds.scriptJob(killAll=True, force=True)

# remove all un-protected / un-permanent script jobs
cmds.scriptJob(killAll=True, force=False)

# remove specific job id
jid = cmds.scriptJob(attributeChange=["myAsset.variation", update_variation], killWithScene=True)
cmds.scriptJob(kill=jid) # would need to add force=True for protected scripts

# delete script jobs when UI is deleted with parent flag
def test_UI_scriptjobs():
	win = cmds.window(title="UI test")
	cmds.scriptJob(parent=win, event=["SelectionChanged", python_callback] )
	cmds.showWindow(win)
```

## Embed code in Maya scene file

In order to embed some code into a Maya scene file script node could be use.
Embedding code into a scene could be usefull to have some critical code component 
without inducing pipeline dependancies. 

```python
def embed_script():

	script_folder = cmds.internalVar(userScriptDir=True)
	path = cmds.fileDialog2(
			fileMode=1, # only allow a single file selection
			fileFilter="Mel Files (*.mel);;Python files (*.py)",
			startingDirectory=script_folder
		)

	if not path:
		return

	sourceType = "python" if path[0].endswith(".py") else "mel"
def embed_script():

	path = cmds.fileDialog2(
			fileMode=1, # only allow a single file selection
			fileFilter="Python files (*.py);;Mel Files (*.mel)",
			startingDirectory=cmds.internalVar(userScriptDir=True)
		)

	if not path:
		return
    
	sourceType = "python" if path[0].endswith(".py") else "mel"
	with open(path[0], 'r') as f:
	    	cmds.scriptNode(
	    	    sourceType=sourceType,
	    	    scriptType=2,
	    	    beforeScript=f.read()
	    	    )

embed_script()

# scriptType flag possible value
# Specifies when the script is executed. The following values may be used:
	# 0	Execute on demand. 
	# 1	Execute on file load or on node deletion.
	# 2	Execute on file load or on node deletion when not in batch mode.
	# 3	Internal
	# 4	Execute on software render
	# 5	Execute on software frame render
	# 6	Execute on scene configuration
	# 7	Execute on time changed
```

## Sources
```
    - Maya programming with python cookbook, A.Herbez (2016)
    - Maya python for games and film, A.Mechtley & R.Trowbridge (2012)
    - Practical Maya programming with python, R.Galanakis (2012) 
    - Maya documentation
```