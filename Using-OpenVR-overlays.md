# Using OpenVR overlays

OpenVR has an overlay feature, which allows you to draw graphics onto planes located in the OpenVR world. These are drawn on top of other OpenVR applications that might be running, like games. Unlike normal OpenVR applications that can only run one at a time, overlay applications run alongside normal OpenVR applications.

These overlays can be attached to the HMD or the controllers (so it will track/move with them), or statically positioned in the OpenVR space.

Possible use cases for overlays are chat overlays for streamers, displaying advanced OpenVR settings, simple widgets like displaying time, FPS or desktop Windows in the VR world, and many more. Yes, even the built-in SteamVR dashboard is an overlay!

## Getting a version of Godot OpenVR with overlay support

The released asset library version does not have support for overlays just yet (this will come with the release of Godot 3.2), so you will need to use the Godot OpenVR module ([godot_openvr](https://github.com/GodotVR/godot_openvr)) master branch for now. This asset's master branch will be updated soon to make this easier.

## Basic concepts

Overlays are implemented in Godot OpenVR as a GDNative script (it's basically like a GDScript you would attach to any node, but built using C++), that you just have to attach to any Viewport node. It then converts that Viewport node into an overlay automatically. This also means you can have more than one overlay!

## From empty to overlay - step by step

### Create a new Godot project

1. Open Godot, click **New Project** and give your project a nice name. In this example, I have used **Overlay Test**.
2. Click **Browse** to choose a path where the project folder should be created in then click **Create Folder**.
3. Then click **Create and Edit** to create your new project and open the editor.

[[images/overlay_create_project.png]]

### Installing the Godot OpenVR module

1. Create a folder named **addons** in your project's main folder.
2. Download the Godot OpenVR repository's master branch, unpack it, and copy the **demos/addons/godot-openvr** folder to your **addons** folder, which now should look like:

[[images/openvr_addon.png]]

### Initialize Godot OpenVR

We initialize Godot OpenVR almost as normal, only with a few changes:

1. Since we do not use the main Viewport, we don't change any Viewport settings here.
2. We set the OpenVR library to "overlay" mode before initializing OpenVR.

Create a new scene (which will become your main scene), attach a new script (GDScript), and write the following in it's script **_ready()** function:

```GDScript
# Get configuration object
var OpenVRConfig = preload("res://addons/godot-openvr/OpenVRConfig.gdns").new()
OpenVRConfig.set_application_type(2) # Set to OVERLAY MODE = 2, NORMAL MODE = 1
OpenVRConfig.set_tracking_universe(1) # Set to SEATED MODE = 0, STANDING MODE = 1, RAW MODE = 2
	
# Find the OpenVR interface and initialise it
var arvr_interface : ARVRInterface = ARVRServer.find_interface("OpenVR")

if arvr_interface and arvr_interface.initialize():
        pass
```

### Attach the OpenVROverlay to a Viewport

### Let's put some things to display on the overlay