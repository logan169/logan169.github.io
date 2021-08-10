---
title: 'Maya MEL Programming 101'
tags: Maya MEL
---

MEL (Maya Embedded Language) is the interpreted Maya Autodesk 3D software scripting language. While MEL as been there since a long time, it is still being used by cinema studio's technical directors to extend existing Maya UI, menus and tools. Given Maya compatible python package that were created on the way (like Pymel, cmds, openMaya) since Maya's creation, the place and use cases of MEL has been greatly reduced while it might still be usefull for some specific tasks or fast prototyping.

## Ways to run MEL/Python code in Maya

- Command Line 
- Command Shell
- Script Editor
- Shelf Tool
- Script File
- Callback
- Expressions

## Default Mel scripts folder
```
// scripts put here will be automatically loaded by Maya at launch time
~/maya/x.x/scripts
```

## Adding custom scripts folders 
```
// Custom extra scripts folders could be added by updating bash env variable
set MAYA_SCRIPTS_PATH=$MAYA_SCRIPTS_PATH;<new_script_path>;

// An alternative way is to update Maya.env file with
MAYA_SCRIPTS_PATH = <new_script_path>;

// Retrieve the current Maya.env filepath location
string $envFile = `about -environnementFile`;
```

## Maya Init scripts
```
// userSetup.mel is being executed every time maya is started
// this can't be used to set the scene up
```

## Debugging with Trace
```
int $a = 65;
trace -where ("value : " + $a);
// returns 
// file: <filepath> line n : value : 65 

```

## CRUD operations
```
// 3 Mel command mode: create (default), query, edit
// <nodeType> <mode> <flags/values> -name <nodeName>
sphere -radius 4 -name mySphere;  // Create (default)
sphere -query -radius mysphere; // Query 
sphere -edit -radius 2 mySphere; // Edit
rename mySphere mySphere2; // Rename
delete mySphere; // Delete
```

## Reloading (updated) Mel file
```
source myMelFile.mel // reimport current myMelFile definition state
```

## Cancel last action
```
undo;
```

## Retrieve available flags for command: 
```
help(sphere)
```

## Mel Variables
```
int $myvar = 2;
sphere -edit -radius $myvar MySphere;
``` 

## Mel Data Types
```
// Int & Float
int $myInt = 2;
float $myFloat = 2.0;

// String
string $myString = "Hello World\n";
int numChars = size($myString);

// Vector
vector $myVector = << 10.0, 2.0, 3.0 >>; 
// update x axis value as follow
vector $myVector = << 15.0, $myVector.y, $myVector.z >>; 
move -absolute ($myVector.x) ($myVector.y) ($myVector.z);
print ($myVector.x);

// Array
int $myIntArray[] = {2, 5, 8}; // more efficient to provide array size at creation time
string $myStrArray[3] = {"hello", "world", '!'}; 
vector $myVecArray[2] = {<< 10.0, 2.0, 3.0 >>, << 10.0, 5.0, 3.0 >>};
float $myFloatArray[3] = {2.5, 5.0, 8.1};
print ( $myFloatArray[0], size($myFloatArray) );

// dynamic array resizing
int $vals = [];
$vals[0] = 2;
$vals[4] = 4;
print ( $vals ) // returns 2 0 0 4

// appending to array
$myStrArray[ size($myStrArray) ] = "\nYea";

// deleting array content
clear( $myStrArray );

// Matrices
matrix $myMatrix[2][4] = << 3,8,9,7; 5,4,3,2 >>;
print ( $myMatrix[0][0] );
float $topTransformMtx[] = `xform -query -matrix topTransform;` // retrieve topTransform node's matrix as a float values list

// Boolean
bool_true = on; // => true
bool_true = true; // => true
bool_true = yes; // => true

bool_false = off; // => false
bool_false = false; // => false
bool_false = no; // => false
```
## Assignment chaining & in-place operation

```
int $a, $b;

$b = $a = 50;

$a/=1;
$a+=1;
$a-=1;
$a*=1;

$a++;
$a--;
```

## Getter & Setter

```
float $myNodeScale = `getAttr myNode.scale`; // store scale info in $myNodeScale var

vector  $tmp = << 1.0, 0.0, 0.0>>;
setAttr myNode.scale ($tmp.x) ($tmp.y) ($tmp.z);
```

## Custom Attributes
```
/* Create */
addAttr -longName "mySlider" -attibutType long -min 0 -max 10 -defaultValue 0 myNode; //create a slider

if (!attributeExists ("id", myNode)) {
    addAttr -longName "id" -attibutType long myNode;
}

/* Read */
listAttr myNode; // list all attributes
listAttr -userDefined myNode; // list all custom attributes
listAttr -keyable myNode; // list all keyable attributes

/* Update */
setAttr -keyable true myNode.mySlider;

/* Delete */
deleteAttr myNode.mySlider; //delete attr
```

## If statement

```
int $a = 10;

if ( $a < 10) {
	print "a is lower than 10";
}
else if ($a == 10) {
	print "a is equal to 10";
}
else {
	print "a is higher than 10";
}
```

## Switch statement

```
int $a = 10;

switch ($a) {
    case 1:
        print "a == 1";
        break;
    case 2:
        print "a == 2";
        break;
    default:
        print "a is not in equal to either 1 or 2";
        break;
}
```
## Conditional operator

```
int $a = 10;
$equal_ten = ($a == 10) ? true : false; 
```

## For loop

```
int $elems[3] = { 10, 5, 2 };
int $index;

for ($index=0; $index < size($elems); $index++ ) {
    print ( $elems[$index] );
}
```

## For-in loop

```
int $elems[3] = { 10, 5, 2 };
int $elem;

for ($elem in $elems ) {
    print ( $elem );
}
```

## Warnings & Errors
```
warning "Incorrect parm, it will be ignored!";
error "Error, illegit value";
```

## Check if a procedure exists
```
if (!`exists myProcedure`)
    error "Error, Couldn't find myProcedure. Please install it"
else 
    // returns the filepath where the procedure comes from
    `whatIs myProcedure`;  // whatIs could also be used to retrieve variable's type
```

## Type variables
```
int $a = 10;
whatIs "$a"; // returns int variable
```

## Functions aka:procedure
```
proc int test (int $input){
	if ($input == 1) {
		return 1;
	}
	return 0;
}
```

## Try / Except
```
int $err = catch( eval("source foobar") ) // return 1 if an error was raised
```

## Time and keyframes

```

float $currentFrame = currentime -query; //retrieve current frame
currentTime 2; // set current frame to be the #2 
currentTime 2sec; // compute and place the current frame to be on 2sec 

listAnimatable myNode; // retrieve all keyable attributes
keyframe -query -name myNode; // retrieve myNode keyframed attributes
keyframe -query -name myNode.translateY; // retrieve myNode.translateY keyframes
keyframe -query -time 2 -eval myNode.translateY; // retrieve myNode.translateY keyframes at frame 2 without evaluating DAG
getAttr -time 2 myNode.translateX; // retrieve x translate value at frame 2 (getAttr with -time could induce DAG eval)
play; // play the frame range

setKeyframe -time 2 -value 2 myNode.translateY; // set a keyframe at frame 2
setKeyframe -time 4 -value 4 myNode.translateY; // set a keyframe at frame 2
setKeyframe -insert -time 3 myNode.translateY; // insert a keyframe at frame 3 without changing the curve shape
keyframe -edit -time 4 -valueChange 10 myNode.translateY; // update keyframe value

```

## Fetch & select nodes

```
ls  // list all nodes
ls -type surfaceShape // list all surface shape nodes
ls -selection // list currently selected nodes
ls -clear // cancel selection

listRelatives parentNode; // list all child nodes, extra flags could be used to filter out result
listRelatives -parent childNode; // retrieve parent nodes
listRelatives -shapes node; // retrieve shapeNode

select -r mySphere; // select a node


```

## Parent & Group

```
group -name topTransform mySphere; // create a new transform node and parent the node to it. Transformation applied to topTransform will also affect mySphere
parent mySphere2 topTransform; // make mySphere2 a child of topTransform and applied transforms
inherit Transform -off MySphere2; // Disable parent tranforms inheritance for MySphere2

```

## Transform obj

```
/* Transform Objects: */

// -relative or -absolute flags could be used
move -relative 1 0 0 topTransform; // Translate topTransform and its childrens
scale -absolute 10 2.3 3 MySphere2;// Scale MyShere2
rotate -relative 90deg 0 0 topTransform; // Rotate topTransform and its childrens

xform -relative -translation 0 5 -6 -scale 0.5 0 0 -rotation 90deg 0 0 topTransform; // Do all operation at once

// transform every selected node per 1 on y axis
$nodes = `ls -selection`;
for ($node in $nodes) {
    setAttr ($node + ".translateY") (1.0 + `getAttr $node.translateY`)
}


/* Move an object along a curve: */
pathAnimation -curve testCurve myNode;
```


## Joint skeleton

```
joint; // create joint1 under the currently selected obj (might wanna do a ls -clear before) 
joint -position 0 1 0;  // create joint2 under joint1 at given position
insert joint1; //create a new joint under joint1 instead of joint 2 as the joint command would have

joint -edit -relative -position 0 5 0 joint2; // edit joint2 position to be 0 5 0 from its parent joint3
joint -query -degreeOfFreedom joint3; // return axis that could be edited
joint -edit -degreeOfFreedom "y" joint1; // limit joint1 movement to y axis only

joint -query -limitY joint1; // retrieve current min/max possible values for edition (-360; 360);
joint -edit -limitY 0deg 180deg joint1; // limit rotation value on Y axis from 0-90 degrees
joint -edit -angleY 50deg joint3; // rotate joint3 relatively to its parent
rotate 0 90deg 0 joint2; // rotate joint2 with an absolute rotation

removeJoint joint2; //delete joint



/* rescale skeleton: */

proc rescaleSkeleton ( string $inputNode, float $outputScale) {
    
    string $jointNodes = `listRelatives -fullPath -children -type joint -allDescendents $inputNode`;

    for ($jointNode in $jointNodes) {
        // retrieve current joint position, edit it and update it
        float $currentJointPosition = `joint -query -relative -position $jointNode`;

        $currentJointPosition[0] *= $outputScale;
        $currentJointPosition[1] *= $outputScale;
        $currentJointPosition[2] *= $outputScale;

        joint -edit -position $currentJointPosition[0] $currentJointPosition[1] $currentJointPosition[2]  $jointNode;
    }
}
--------------------------------

/* Retrieve joints position: */

proc printJointsTransform(string $inputNode, string $spaceFlag) {
    /*
    $inputNode is the skeleton root node
    $spaceFlag is either '-worldSpace' or '-objectSpace'
    */

    $jointNodes = `listRelatives -fullPath -children -type joint -allDescendents $inputNode`;
    
    for ($jointNode in $jointNodes) {
        // retrieve current joint transform and print it
        float $trans [] = `xform $spaceFlag -query -translation $jointNode;`
        float $rot [] = `xform $spaceFlag -query -rotation $jointNode;`
        float $scale [] = `xform $spaceFlag -query -scale $jointNode;`

        print ("\n Joint: " + $jointNode);
        print ('\n Translation: ' + $trans[0] + ' ' + $trans[1] + ' ' +  $trans[2]);
        print ('\n Rotation: ' + $rot[0] + ' ' + $rot[1] + ' ' +  $rot[2]);
        print ('\n Scale: ' + $scale[0] + ' ' + $scale[1] + ' ' +  $scale[2]);
    
    }

}
```

## UI

```

// Simple window connecting attr of a node to some parm
// source: https://www.asc.ohio-state.edu/price.566/courses/694/cntrlConnections.html


{
    string $node [] = `sphere`;
    window -title "1st way node parm binding" testWindow;
        panelLayout -configuration "horizontal2";
        columnLayout;
            attrFieldGrp -attribute $node[0] + ".translate" -label $node[0] + "_translate";
            attrFieldGrp -attribute $node[0] + ".rotate" -label $node[0] + "_rotate";
            attrFieldGrp -attribute $node[0] + ".scale" -label $node[0] + "_scale";
            setParrent ..;
  
}
///////////////////////////////////////////////////

{
    string $nodes [] = `ls -selection`;
    string $node = $nodes[0];
    //deleteUI testWindow2;
    
    window -title "2st way individual node parm binding" testWindow2;
        columnLayout;
            floatFieldGrp -label "Translate" -numberOfFields 3 translation_field;
            
            // connect fields to attr, all 3 parms needs to be plugged to work
            connectControl -index 1 translation_field ($node + ".tx");
            connectControl -index 2 translation_field ($node + ".ty");
            connectControl -index 3 translation_field ($node + ".tz");
    
    showWindow testWindow2;
    
}
```


## Miscellaneous

```

/* Type object: */

$obj = `ls -selection`;
objectType $obj; // returns node type
--------------------------------

/* Check if object exists: */

objectExists nurbs1; // returns 1 if nurbs1 exists, 0 otherwise
--------------------------------

/* Find operating System and Versioning: */

float $mayaVer = float(`about -version`);
string $opSys = `about -operatingSystem`;
--------------------------------

```

## Sources

```
    - Complete Maya Programming, David A. D. Gould (2003)
    - https://www.asc.ohio-state.edu/price.566/courses/694/
```