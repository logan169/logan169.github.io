---
title: 'Houdini vex 101'
tags: Houdini VEX
---

VEX is a Houdini (sideFx) language. Its evaluation is typically very efficient giving performance close to compiled C/C++ code. VEX is not an alternative to scripting, but rather a smaller, more efficient general purpose language for writing shaders and custom nodes.<!--more--> VEX is loosely based on the C language, but takes ideas from C++ as well as the RenderMan shading language.

VEX is used in several places in Houdini:

	- Rendering – mantra uses VEX for all shading computation. This includes light, surface, displacement and fog shaders.

	- Compositing – The VEX Generator and VEX Filter COPs allows you to write complex custom COPs in VEX. The expressions evaluate very close to C/C++ speeds and run 1000's of times faster than the Pixel Expression COP.

	- Particles – The POP VOP allows you to do almost anything that POPs can do themselves. It is possible to write one single VEX function which performs the task of many POPs. Because the VEX code is localized to one single operator, the VEX code will typically run faster than a network of POPs.

	- Modeling – The VEX SOP allows you to write a custom surface node that manipulates point attributes. This can move points around, adjust velocities, change colors. As well, you can group points or do many other useful tasks. VEX SOPs typically run 10 or more times faster than a point SOP.

	- CHOPs – The VEX CHOP lets you to custom CHOPs. The CHOP functions can manipulate arbitrary numbers of input channels and process channel data in arbitrary ways. In some cases, the VEX code can run faster than compiled C++ code.

	- Fur – Procedural fur behavior is implemented with VEX.



Here is a VEX cheat sheet 100% taken from this [website](https://mrkunz.com) blog. I'm not the one who produce this content, All credit goes to John Kunz.
My goal here is to curate and gather interesting web ressources at a specific location that is easier for me to access.

## Global Variables
A list of variables available in wrangles. The type indicator isn't needed, but is included as a reminder.
```javascript
// Available in all SOP wrangles
f@Frame     //The current floating frame number, equivalent to the $FF Hscript variable
f@Time      //The current time in seconds, equivalent to the $T Hscript variable
i@SimFrame  //The integer simulation timestep number ($SF), only present in DOP contexts.
f@SimTime   //The simulation time in seconds ($ST), only present in DOP contexts.
f@TimeInc   //The timestep currently being used for simulation or playback.

// Available in Attribute Wrangle
v@P         //The position of the current element.
i@ptnum     //The point number attached to the currently processed element.
i@vtxnum    //The linear number of the currently processed vertex.
i@primnum   //The primitive number attached to the currently processed element.
i@elemnum   //The index number of the currently processed element.
i@numpt     //The total number of points in the geometry.
i@numvtx    //The number of vertices in the primitive of the currently processed element.
i@numprim   //The total number of primitives in the geometry.
i@numelem   //The total number of elements being processed.

// Available in Volume Wrangle
v@P                     //The position of the current voxel.
f@density               //The value of the density field at the current voxel location.
v@center                //The center of the current volume.
v@dPdx, v@dPdy, v@dPdz  //These vectors store the change in P that occurs in the x, y, and z voxel indices.
i@ix, i@iy, i@iz        //Voxel indices. For dense volumes (non-VDB) these range from 0 to resolution-1.
i@resx, i@resy, i@resz  //The resolution of the current volume.

// Common Geometry Attributes
// Frequently used attributes. Houdini knows to cast these to the appropriate VEX datatype.
// Int
@id         // A unique number that remains the same throughout a simulation.

// Float
@pscale     // Particle radius size.  Uniform scale.  Set display particles as 'Discs' to visualize.
@width      // Thickness of curves.  Enable 'Shade Open Curves In Viewport' on the object node to visualize.
@Alpha      // Alpha transparency override.  The viewport uses this to set the alpha of OpenGL geometry.
@Pw         // Spline weight.

// Vector3
@P          // Point position.  Used this to lay out points in 3D space.
@Cd         // Diffuse color override.  The viewport uses this to color OpenGL geometry.
@N          // Surface or curve normal.  Houdini will compute the normal if this attribute does not exist.
@scale      // Vector scale.  Allows directional scaling or stretching (in one direction).
@rest       // Used by procedural patterns and textures to stick on deforming and animated surfaces.
@up         // Up vector.  The up direction for local space, typically (0, 1, 0).
@uv         // UV texture coordinates for this point/vertex.
@v          // Point velocity.  The direction and speed of movement in units per second.

// Vector4
@orient     // The local orientation of the point (represented as a quaternion).
@rot        // Additional rotation to be applied after orient, N, and up attributes.

// String
@name       // A unique name identifying which primitives belong to which piece.  Also used to label volumes.
@instance   // Path of an object node to be instanced at render time.
```

## Specifying VEX Data Types
The following characters are used to cast to the corresponding data type.
```javascript
float       f@name    // Floating point scalar values.
vector2     u@name    // Two floating point values. Could be used to store 2D positions.
vector3     v@name    // Three floating point values. Usually positions, directions, normals, UVW or colors.
vector4     p@name    // Four floating point values. Usually rotation quaternions, or color and alpha (RGBA).
int         i@name    // Integer values (VEX uses 32 bit integers).
matrix2     2@name    // Four floating point values representing a 2D rotation matrix.
matrix3     3@name    // Nine floating point values representing a 3D rotation matrix or 2D transform matrix.
matrix      4@name    // Sixteen floating point values representing a 3D transform matrix.
string      s@name    // A string of characters.
```

## Channel Shortcut Syntax
This is the syntax used to hint at the data type of auto generated wrangle parameters.
```javascript
ch('flt1');             // Float
chf('flt2');            // Float
chi('int');             // Integer
chv('vecparm');         // Vector 3
chp('quat');            // Vector 4 / Quaternion
ch3('m3');              // 3x3 Matrix
ch4('m4');              // 4x4 Matrix
chs('str');             // String
chramp('r', x);         // Spline Ramp
vector(chramp('c', x)); // RGB Ramp 
```

## Accessing Attributes on Other Inputs and Nodes
This syntax is used to refer to nodes wired into the inputs of the wrangle, or other nodes in the network.
```javascript
// 'opinput:X' is the most legible and always works (the first input is input 0).
point('opinput:0', 'P', i@ptnum)
point('opinput:1', 'P', i@ptnum)
point('opinput:2', 'P', i@ptnum)
point('opinput:3', 'P', i@ptnum)

// The integer input number (the first input is 0).  Some functions don't support this but it's easy to type.
point(0, 'P', i@ptnum)
point(1, 'P', i@ptnum)
point(2, 'P', i@ptnum)
point(3, 'P', i@ptnum)

// @OpInputX works as well, but be careful as it isn't 0 based, instead it starts at 1 which is confusing
point(@OpInput1, 'P', i@ptnum)
point(@OpInput2, 'P', i@ptnum)
point(@OpInput3, 'P', i@ptnum)
point(@OpInput4, 'P', i@ptnum)

// v@opinputX_* reads an attribute from the same element on the numbered input (first input is input 0).
v@opinput0_P
v@opinput1_P
v@opinput2_P
v@opinput3_P

// Absolute and relative paths to other nodes look like this.
point('op:/obj/geo1/OUT', 'P', i@ptnum)
point('op:../../OUT', 'P', i@ptnum)
```

## Copying and Instancing Attributes
When copying or instancing, Houdini looks for these point attributes to customize each copy/instance.
```javascript
p@orient                    // Orientation of the copy.
f@pscale                    // Uniform scale.
v@scale                     // Non-uniform scale.
v@N                         // Normal (+Z axis of the copy, if no p@orient).
v@up                        // Up vector of the copy (+Y axis of the copy, if no p@orient).
v@v                         // Velocity of the copy (motion blur), used as +Z axis if no p@orient or v@N.
p@rot                       // Additional rotation (applied after the orientation attributes above).
v@P                         // Translation of the copy.
v@trans                     // Translation of the copy, in addition to v@P.
v@pivot                     // Local pivot point for the copy.
3@transform or 4@transform  // Transform matrix overriding everything except v@P, v@pivot, and v@trans.
s@shop_materialpath         // The instanced object uses this material.
s@material_override         // A serialized Python dictionary mapping material parameter names to values.
s@instance
s@instancefile              // File path indicating what geometry to instance.
s@instancepath              // Geometry to instance. This is either a path to a file on disk or an op: path.
```

## DOP Particle Attributes
A particle system is first and foremost driven by attributes. Here are some of the attributes used.
```javascript
f@age       // Time in seconds since the particle was born.
f@life      // Time in seconds the particle is allowed to live. When f@age>f@life, i@dead will be set to 1.
f@nage      // Normalized age, f@age divided by f@life.  Implicit attribute, you cannot write to this.
i@dead      // Whether a particle is living (0) or dead (1).  A dead particle is deleted in the Reaping stage.
i@id        // A unique id for the particle that remains the same throughout a single simulation.

i@stopped   // Whether a particle is moving (0) or stopped (1).
i@stuck     // Whether a particle is free (0) or stuck (1).
i@sliding   // Whether a particle is free (0) or sliding along a surface (1).
f@cling     // Force applied to sliding paritcles inwards (according to the collision's surface normal).
s@pospath   // The path to the object that the particle is colliding with.
i@posprim   // Which collision primitive in the path geometry whose position we wish to refer to.
v@posuv     // Parametric uv on the collision primitive.

i@hittotal  // The cumulative total of all hits for the particle (only incremented once per timestep).
i@has_pprevious // This is set to 1 if v@pprevious contains valid values.
v@pprevious // Stores the position of the particle on the previous frame.  Used for collision detection.
i@hitnum    // The number of times the particle collided in the last POP Collision Detect.
s@hitpath   // The path to the object that was hit. A path to a file on disk or an op: path.
i@hitprim   // The primitive hit. Could be -1 if it the collision detector couldn’t figure out which prim.
v@hituv     // The parametric UV space on the primitive.
v@hitpos    // Where the hit actually occurred.  Useful if the colliding object was moving.
v@hitnml    // The normal of the surface at the time of the collision.
v@hitv      // The velocity of the surface at the time of the collision.
f@hittime   // When the collision occurred, that could be within a frame.
f@hitimpulse// Records how much of an impulse was needed for the collision resolution.  varies with timestep.
f@bounce    // When particles bounce off another object, this controls how much energy they keep.
f@bounceforward // Controls how much energy they keep in the tangential direction.
f@friction  // When particles bounce, they are slowed down proportional to how hard they hit.
s@collisionignore   // Objects that match this pattern will not be collided.

f@force     // Forces on the particle for this frame.
f@mass      // Inertia of the particle.
v@spinshape // This is multiplied by f@pscale to determine the shape of the particle for rotational inertia.
f@drag      // How much the particle is effected by any wind effects.
f@dragexp   // Ranges from 1 to 2, default is set on the solver.  Used for both angular and linear drag.
v@dragshape // How much the particle is dragged in each of its local axes.
v@dragcenter// If specified, drag forces will also generate torques on the particle.
v@targetv   // The local wind speed. Thought of as the goal, or target, velocity for the particle.
f@airresist // How important it is to match the wind speed.  Thickness of the air.
f@speedmin  // Minumum speed, in units per second, that a particle can move.
f@speedmax  // Maximum speed, in units per second, that a particle can move.

p@orient    // Orientation of the particle.  Used for figuring out 'local' forces.
v@w         // Angular speed of the particle.  A vector giving the rotation axis.
v@torque    // The equivalent of force for spins. No inertial tensor (the equivalent of mass) is supported.
v@targetw   // The goal spin direction and speed for this particle.
f@spinresist// How important it is to match the targetw.
f@spinmin   // Minumum speed in radians per second that a particle can spin.
f@spinmax   // Maximum speed in radians per second that a particle can spin.


//DOP Grains Attributes
//Particles under control of POP Grains have the ispbd attribute set to 1. This causes them to not perform normal movement update in the POP Solver, as the actual motion update is done in this node.
i@ispbd     // A value of 1 causes the particle to behave as grains.
f@pscale    // Used to determine the radius of each particle.
f@repulsionweight   // How much the particle collision forces are weighted.
f@repulsionstiffness// How strongly particles are kept apart.  Higher values result in less bouncy repulsion.
f@attractionweight  // How much the particles will naturally stick together when close.
f@attractionstiffness   // How strongly nearby particles stick to each other.
v@targetP   // Particles are constrained to this location.
f@targetweight      // The weight of the v@targetP constraints.
f@targetstiffness   // The stiffness with which particles are fixed to their v@targetP attribute.

f@restlength// Particles connected by polylines will be forced to maintain this distance (prim attribute).
f@constraintweight  // Scale, on a per-particle basis of the constraint force.
f@constraintstiffness   // This controls the stiffness on a per-particle basis.
f@strain    // This primitive attribute records how much the constraint is stretched.
f@strength  // If f@strain exceeds this primitive attribute, the constraint will be removed.
```

## DOP packed RBD attributes
The Bullet Solver uses several point attributes to store the properties of each piece of a packed object.
```javascript
i@active    // Specifies whether the object is able to react to other objects.
i@animated  // Specifies whether the transform should be updated from its SOP geometry at each timestep.
i@deforming // Specifies whether the collision shape should be rebuilt from its SOP geometry each timestep.

f@bounce    // The elasticity of the object.
i@bullet_add_impact // Impacts that occur during the sim will be recorded in the Impacts or Feedback data.
i@bullet_ignore     // Specifies whether the object should be completely ignored by the Bullet solver.
f@bullet_angular_sleep_threshold// The sleeping threshold for the object’s angular velocity.
f@bullet_linear_sleep_threshold // The sleeping threshold for the object’s linear velocity.
i@bullet_want_deactivate// Disables simulation of a non-moving object until the object moves again. 
i@computecom// Specifies whether the center of mass should be computed from the collision shape.
i@computemass   // Specifies whether the mass should be computedfrom the collision shape and density.
f@creationtime  // Stores the simulation time at which the object was created.
i@dead  // Specifies whether the object should be deleted during the next solve.
f@density   // The mass of an object is its volume times its density.
f@friction  // The coefficient of friction of the object.
f@inertialtensorstiffness   // Rotational stiffness.  A scale factor applied to the inertial tensor. 
i@inheritvelocity   // v and w point attributes from the SOP geometry will override the initial velocity.
f@mass  // The mass of the object.
s@name  // A unique name for the object. Used by Constraint Networks.
p@orient// The orientation of the object.
v@P     // The current position of the object’s center of mass.
v@pivot // The pivot that the orientation applies to. If i@computecom is non-zero, this is auto-computed.
v@v     // Linear velocity of the object.
v@w     // Angular velocity of the object, in radians per second.

i@bullet_adjust_geometry    // Shrinks the collision geometry.
i@bullet_autofit    // Use the bounds of the object for Box, Capsule, Cylinder, Sphere, or Plane.
f@bullet_collision_margin   // Padding distance between collision shapes.
s@bullet_georep     // Can be convexhull, concave, box, capsule, cylinder, compound, sphere, or plane.
i@bullet_groupconnected     // Create convex hull per set of connected primitives.
f@bullet_length     // The length of the Capsule or Cylinder collision shape in the Y direction.
v@bullet_primR  // Orientation of the Box, Capsule, Cylinder, or Plane collision shape.
v@bullet_primS  // Size of the Box collision shape.
v@bullet_primT  // Position of the Box, Sphere, Capsule, Cylinder, or Plane collision shape.
f@bullet_radius // Radius of the Sphere, Capsule, or Cylinder collision shape.
f@bullet_shrink_amount  // Specifies the amount of resizing done by Shrink Collision Geometry.

s@activationignore  // Won't be activated by collisions with any objects that match this pattern.
s@collisiongroup    // Specifies the name of a collision group that this object belongs to.
s@collisionignore   // The object will not collide against any objects that match this pattern.
f@min_activation_impulse// Minimum impulse that will cause the object to switch from inactive to active.

f@speedmin  // Minumum speed, in units per second, that a particle can move.
f@speedmax  // Maximum speed, in units per second, that a particle can move.
f@spinmin   // Minumum speed in radians per second that a particle can spin.
f@spinmax   // Maximum speed in radians per second that a particle can spin.
f@accelmax  // Limits the change in the object’s speed that is caused by enforcing constraints.
f@angaccelmax   // Limits the change in the object’s angular speed that is caused by enforcing constraints.

f@airresist // Specifies how important it is to match the target velocity (v@targetv).
f@drag      // How much the the v@targetv and f@airresist attributes effect the object.
f@dragexp   // Ranges from 1 to 2, default is set on the solver.  Used for both angular and linear drag.
v@force     // Specifies a force that will be applied to the center of mass of the object.
f@spinresist// Specifies how important it is to match the target angular velocity (v@targetw).
v@targetv   // Target velocity for the object. Used in combination with the f@airresist attribute.
v@targetw   // Target angular velocity for the object. Used in combination with the f@spinresist attribute.
v@torque    // Specifies a torque that will be applied to the object.

i@bullet_autofit_valid  // Stores whether the solver has already computed collision shape attributes.
i@bullet_sleeping   // Tracks whether the object has been put to sleep by the solver.
f@deactivation_time // Amount of time the speed has been below the Linear Threshold or Angular Threshold.
i@found_overlap // Used by the solver to determine whether it has performed the overlap test.
i@id    // A unique identifier for the object.
i@nextid// Stores the i@id the solver will assign to the next new object.
```

## DOP Constraint Network Attributes
You can create attributes on the geometry to customize each constraint’s behavior and type. If a primitive attribute with the same name as a constraint property (such as damping) is present, the attribute value will be multiplied with the value from the constraint sub-data.
```javascript
i@anchor_id  // The anchor’s position will be bound to this point (or vertex).
i@anchor_type   // Specifies if the anchor is attached to a point, vertex or agent transform.
v@condir// The normal of a plane (i@condof==1) or axis (i@condof==2) the object can move on or rotate about.
i@condof// Identifies the number of constrained degrees of freedom for an anchor (0 to 3).
s@name  // The object the constraint is attached to.  An empty name attaches to world space position.
v@P     // Identifies the initial world space position of the anchor.
p@orient// Initial world space orientation of the anchor.  Takes precedence over v@r point attribute.
v@r     // Identifies the initial world space orientation of the anchor as Euler angles.
v@v     // Identifies the velocity of the world space position to which the constraint is attached.
v@w     // Identifies the angular velocity of the world space position to which the constraint is attached.

s@constraint_name   // Prim attribute specifying the Data Name of the constraint to create.
s@constraint_type   // Prim attribute specifying degrees of freedom ('position', 'rotation' or 'all').
s@next_constraint_name  // Prim attribute specifying the next constraint_name to use after broken.
s@next_constraint_type  // Prim attribute specifying the next constraint_type to use after broken.
i@propagate_iteration   // Detail attribute specifying number of impact propagations for glue constraints.

f@force     // Prim attribute updated by the solver to contain the force applied to satisfy the constraint.
f@distance  // Prim attribute updated by the solver to contain the distance between it's anchors.
f@torque    // Prim attribute updated by the solver to contain the torque applied to satisfy the constraint.
f@angle     // Prim attribute updated by the solver to contain the angle (in radians) between the anchors.
f@impact    // Prim attribute updated by the solver to contain the accumulated impulses for the glue bond.

i@group_broken  // Any constraints that are in the broken primitive group will be ignored by solvers.
```

## DOP Wire Attributes
You can create attributes on the wire object’s RestGeometry to influence its behavior. Most of these attributes allow fine-tuning of the wire by scaling values set in this node. Point, primitive, or detail attributes of the same name will be used if the vertex attributes are not present.
```javascript
f@width  // Width of each edge.
f@density   //  Density of each point.
p@orient// Initial orientation of each point. This value is stored as a quaternion.
v@v     // Initial velocity of each point.
v@w     // Initial angular velocity of each point measured in radians per second.
f@friction  // Friction of each point.
f@klinear   // Defines how strongly the wire resists stretching.
f@damplinear// Defines how strongly the wire resists oscillation due to stretching forces.
f@kangular  // Defines how strongly the wire resists bending.
f@dampangular   // Defines how strongly the wire resists oscillation due to bending forces.
f@targetstiffness   // Defines how strongly the wire resists deforming from the animated position.
f@targetdamping // Defines how strongly the wire resists oscillation due to stretch forces.
f@normaldrag// The component of drag in the directions normal to the wire.
f@tangentdrag   // The component of drag in the direction tangent to the wire.
i@nocollide // Collision detection for the edge is disabled (Only used if Collision Handling is SDF).
v@restP // Rest position of each point.
p@restorient// Rest orientation of each point.
i@gluetoanimation   // Causes a point’s position and orientation to be constrained to the input geometry.
i@pintoanimation    // Causes a point’s position to be constrained to the input geometry.
v@animationP// Target position of each point.
p@animationorient   // Target orientation of each point.
v@animationv// Target velocity of each point.
v@animationw// Target angular velocity of each point.
i@independentcollisionallowed   // Toggle external collisions (Only non-SDF Geometric Collision).
i@independentcollisionresolved  // Unresolved external collisions (Only non-SDF Geometric Collision).
i@codependentcollisionallowed   // Toggle soft body collisions (Only non-SDF Geometric Collision).
i@codependentcollisionresolved  // Unresolved toggle soft body collisions (Only non-SDF Geometric Collision).
i@selfcollisionallowed  // Toggle self collisions (Only non-SDF Geometric Collision).
i@selfcollisionresolved // Unresolved toggle self collisions (Only non-SDF Geometric Collision).
```
## DOP FLIP Attributes
The FLIP Solver contains an embedded POP Solver, enabling the use of many POP attributes.
```javascript
f@pscale// Particle scale
v@v     // Particle velocity
f@viscosity // The "thickness" of a fluid.
f@density   // The mass per unit volume.
f@temperature   // The temperature of the fluid.
f@vorticity // Measures the amount of circulation in the fluid.
f@divergence// Positive values cause particles to spread out, negative cause them to clump together.
v@rest  // Used to track the position of the fluid over time.
v@rest2 // Used for blending dual rest attributes, avoids stretching. 
f@droplet   // Identifies particles that separate from the main body of fluid.
f@underresolved // Particles that haven't fully resolved on the grid.
i@ballistic // Specifies particles which will be ignored by the fluid solve.
v@Lx    // Angular momentum X axis
v@Ly    // Angular momentum Y axis
v@Lz    // Angular momentum Z axis
```

##  USD attributes
```javascript
Build-in variables
@primpath // Path of the prim, on the stage .
@elemnum // Current element number, whether prim or array element.
@numelem // Total number of prims or array elements.
@primtype // The prim type name.
@primkind // A prim’s kind if set.
@primname // The name of a primitive.
@primpurpose // The display purpose of the current prim.
@primdrawmode // The draw mode of the prim.
@primactive // Returns whether the current prim is active or deactivated.
@primvisible // Returns the visibility of the prim.
```

## Setting primitives visibility for abc export
A frequently overlooked feature of the Alembic format is it's visiblity property. This property allows for entire scemes, objects or specific faces of an object to be flagged as either visible or hidden.

This is particularly useful for effects because it allows objects or faces to appear or disaper over time, by animating the visiblity property.
Quite commonly in Houdini, geometry will be created and deleted to show and hide it.
This can be problematic when it comes to transfering the geomtetry to another software package or rendering engine. The majority of applications and file formats aren't designed to support changing topology. They expect the same amount of objects, faces, verticies, etc. to exist on the first frame, the last frame and every frame in between. If the topology is changing, small issues can arise such as lack of motion blur, mismatched shader assignments and missing areas of geometry. In the worst of cases, it can result in freezing and crashes.

| --- |  --- |
|![broken_glass1](https://mrkunz.com/blog/img/20161208/without_visibility_animation_playblast.gif)|![broken_glass2](https://mrkunz.com/blog/img/20161208/with_visibility_animation_playblast.gif)|

In the image on the left, the visiblity isn't animated at all. This makes it look like the glass is broken before it even begins to shatter.
On the right, the visibility of the edge faces of the glass have animated visiblity so that they reveal the cracks as the pane breaking. This creates a much cleaner and more convincing shattering effect.


Using the Visibility SOP, this is the most straightforward method. The Group SOP can be used to specify visibility as well, by setting the "Group Name" field to: "_3d_hidden_primitives".

Houdini's Alembic Exporter looks for any primitives in this group and sets the Alembic propery accordingly upon export.

```javascript
// The following code will randomly toggle each primitive's visibility every frame
i@group__3d_hidden_primitives = int(rint(random(i@primnum + f@Frame)));

// The following code will randomly toggle each primitive's visibility every frame
setprimgroup(geoself(), '_3d_hidden_primitives', i@primnum, int(rint(random(i@primnum + f@Frame))), 'set');
```

## Creating anti-aliased noise
![anti-aliased noise](https://mrkunz.com/blog/img/03-04-2017/simple_vop_aanoise_240p.v3.png)
```javascript
#include <voplib.h>
vector4 pos = set(v@P.x, v@P.y, v@P.z, 0);
v@Cd = vop_fbmNoiseFP(pos, 0.5, 8, 'noise') + 0.5;
````

The #include directive in the first line of code let's you functions that SideFX has defined in $HFS/houdini/vex/include/voplib.h. Without it, the compiler wouldn't be able to find the vop_fbmNoiseFP() function and throw an error.

Listed below are all the function declarations for vop_fbmNoiseXX().
```c#
float vop_fbmNoiseFF(float pos; float rough; int maxoctaves; string noisetype);
float vop_fbmNoiseFV(vector pos; float rough; int maxoctaves; string noisetype);
float vop_fbmNoiseFP(vector4 pos; float rough; int maxoctaves; string noisetype);
vector vop_fbmNoiseVF(float pos; float rough; int maxoctaves; string noisetype);
vector vop_fbmNoiseVV(vector pos; float rough; int maxoctaves; string noisetype);
vector vop_fbmNoiseVP(vector4 pos; float rough; int maxoctaves; string noisetype);
```
The first of the last two upper case letters of the function name are used to specify if the function returns a float (F) or vector (V). The second of the last two upper case letters is used to specifiy the dimension of the position argument expected by the function, depending on your needs you could use float (F), vector (V), or vector4 (P).

To see all the functions defined in $HFS/houdini/vex/include/voplib.h, you can open it with a text editor.

## Creating vein noise

![vein noise1](https://mrkunz.com/blog/img/03-04-2017/vein_noise_240p.v2.png)

Using a for loop in VEX, it's possible to implement a simple fBm approach and combine several octaves of any arbritary pattern or noise you can think of. This is something that is much more cumbersome to do using VOP nodes, and a case where a code implementation is much more straightforward then a node implementation.

Below is the code used to generate the 'veiny' noise pictured below (inspired by Andy Moorer's blog entry).

```c#
vector npos = v@P/1. + set(0., 666., 0.);   // Noise input 3D position
float namp = 1.;                            // namp (Noise amplitude)
float nval = 0., nweight = 0.;              // Init nval (Noise output value), and nweight (Used to normalize octaves)
int oct = 9;                                // Number of Octaves
for( int i = 0; i < oct; i++ )    {
    float __nval = fit(abs(-0.5+noise(set(npos.x,npos.y,npos.z,f@Time))), 0.0, 0.1, 1., 0.);
    nval += __nval * namp;                  // Amplitude
    nweight += namp;                        // Accumulate weight
    npos *= 2.132433;                       // Lacunarity
    namp *= 0.666;                          // Roughness
}
v@Cd = 1 - pow(nval / nweight, 0.8765);     // Visualize Noise Output
```

Offseting the 4th dimension of the noise input coordinates over time produces this evolving motion.
![vein noise2](https://mrkunz.com/blog/img/03-04-2017/20170304_ezgif.com-optimize.gif)

## Source 
```
  - https://mrkunz.com
  - https://www.sidefx.com/docs/houdini/solaris/vex.html
```
