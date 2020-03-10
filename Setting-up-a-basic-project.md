# Setting up a basic project

The below steps show how you can set up a bare minimum project in Godot using the OpenVR plugin.

Note that you can get a head start by simply adding the `res://addons/godot-openvr/scenes/ovr_first_person.tscn` scene as a subscene or using the `res://addons/godot-openvr/scenes/ovr_main.gd` as a script on your ARVRServer node (some of this is only available in the upcoming 1.1 release of this plugin).

In this writeup however we are building the bare minimum step by step.

It is assumed in this writeup that you are using Godot 3.2 or newer. The plugin will work with Godot 3.1 but you may need to tweak things a little.

Some basic prior knowledge of how Godot works is required, we're not going over the basic interface or how to program GDScript in this writeup.

## Creating a new project

We start by creating a new project in Godot:

[[images/openvr_create_project.png]]

You have a choice between the GLES2 and GLES3 renderer. 
The GLES3 renderer provides better lighting and a higher quality output but at the expense of performance.
GLES2 is a more lightweight renderer better suited to VR if you are targetting lower to medium end hardware.

We are choosing the GLES3 renderer here.

## Copy the plugin in.

Now we can either download the plugin from the asset library or just copy the plugin from a prior download. It is important that you place the entirety of the `addons/godot-openvr` folder into your project. Don't copy over the files outside of this folder as you will overwrite important parts of your project.

## Setting up our main scene

We are presented with an empty project, on our left hand side we're asked what type of root node we need. Select `3D Scene` here. I tend to rename this to `Main` but you can rename it to anything you like.
Now we add an [ARVROrigin](https://docs.godotengine.org/en/3.2/classes/class_arvrorigin.html) node to this, and to that we add an [ARVRCamera](https://docs.godotengine.org/en/3.2/classes/class_arvrcamera.html) and two [ARVRController](https://docs.godotengine.org/en/3.2/classes/class_arvrcontroller.html) nodes. We rename the first [ARVRController](https://docs.godotengine.org/en/3.2/classes/class_arvrcontroller.html) `LeftHand` and the second `RightHand`.

[[images/openvr_scene_tree.png]]

Select the `RightHand` node and in the properties change the controller id to 2:

[[images/openvr_right_hand.png]]

Don't forget to save your scene. 

## Creating our initialisation script

To make OpenVR work we need to initialise it. For this we are going to add a script on the ARVROrigin node. Copy the following code into this script:

```
extends ARVROrigin

func _ready():
	# Find the interface and initialise
	var arvr_interface = ARVRServer.find_interface("OpenVR")
	if arvr_interface and arvr_interface.initialize():		
		# switch to ARVR mode
		get_viewport().arvr = true
		
		# keep linear color space, not needed and thus ignored with the GLES2 renderer
		get_viewport().keep_3d_linear = true
		
		# make sure vsync is disabled or we'll be limited to 60fps
		OS.vsync_enabled = false
		
		# up our physics to 90fps to get in sync with our rendering
		Engine.target_fps = 90
```

Our plugin registers itself automatically, the code `ARVRServer.find_interface("OpenVR")` will find the registered plugin.
If we successfully find it we call `initialize` on it, this will start up SteamVR and starts looking for a headset and controllers.

The function `get_viewport()` obtains our main viewport, we access the `arvr` property and set this to true so the viewport gets rendered in stereoscopic and the render result is sent to our HMD.

Next we set `keep_3d_linear` to true. Rendering of images happens in linear color space but is displayed in sRGB color space on screen. OpenVR however expects the linear color data to be sent to it. Setting this to true ensures the picture looks correct inside of the HMD but does make the preview on screen look too dark.

Follow the instructions in [[Rendering to a separate viewport]] to solve this.

This line of code is not needed when using GLES2.

Next we access `OS.vsync_enabled` and set this to false. VSync is a mechanism that syncs up outputting frame to your monitors refresh rate to prevent screen tearing. The problem is that most peoples monitors have a refresh rate of 60hz effectively limiting Godot to render at 60fps.

Most HMDs run at 90fps.

OpenVR will sync to the refresh rate of the HMD so turning of Godots vsync ensures we can render at the correct framerate.

Finally we set `Engine.target_fps` to 90. This set the update rate for the physics engine which needs to be in sync with the framerate of our game or objects controlled by our physics engine moving irradicly. 

# Adding placeholders for our hands

Just to get started we want to visualise our hands and we do this by adding a MeshInstance as a child to each hand node and simply setting up a cube.

[[image/openvr_hand_cube.png]]

Obviously we don't want a cube but for our getting started it will do. You can read all about actually getting a controller displayed here: [[Displaying 3D controller models]].


# Run your project

And now we can run our project. The first time Godot will ask you to select the scene you want to run, select the scene you've saved and away we go.

[[image/openvr_beginner.png]]
