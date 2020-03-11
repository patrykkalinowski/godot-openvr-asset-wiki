> **Important** Support for the OpenVR action system is currently under development and the functionality presented here is not yet available in the released plugin. This information is presented to make people aware of this breaking change, to help those wanting to test the functionality early, and to gain feedback on this functionality. It is planned to be included in the next major release of the plugin.

**This document is a work in progress!**

Valve introduced a new actions based system for OpenVR awhile back that replaces the current controller implementations with a system that gives far more control over how actions map to given inputs on the controller. 

Instead of hardcoding in your game that when the user presses button 1 a gun fires, you now define a `Fire Gun` action and bind that to the correct input. Especially where controllers differ in inputs they offer this makes it far easier to adopt those to your game. 
For instance Vive controllers have trackpads, Oculus Touch controllers have joysticks, Windows Mixed Reality controllers and Index controllers have both.

> The only limitation as far as I can tell is that the action system only works with two controllers. It does make sense seeing we can only hold two controllers at any given time. You can have more controllers/trackers active but there won't be any way to interact with buttons. As generally these additional controllers are tracking pucks that probably isn't an issue. A clear case of the benefits outweighing the drawbacks.

## The default actions.json file

When you install the new version of the plugin it comes with a default `actions.json` file that will automatically be used. This file is stored alongside mappings for all controllers OpenVR supports by default. You can find these files in the `res://addons/godot-openvr/actions` folder.

The default `actions.json` file is setup for compatibility with the other GodotVR plugins and basically results in the plugin working as before. Every button, trigger and joystick is mapped to the buttons and axis Godot is able to react to.
This is great when you want to build a game that you can deploy to all VR platforms Godot supports without having to worry about implementing different input system.
But doing so means you get non of the benefits of the OpenVR action system.

The default file is still good to have a look at for the syntax and formatting but it is suggested you start with a new file that you configure for the actions you actually need.

The plugin allows you to indicate an alternative file so you don't have to edit the default files. You can create your own.

Continuing from the project we created in [[Setting up a basic project]] we'll start by creating a subfolder called `ovr_actions`.

Inside this folder we create a new `actions.json` file and place the following text in it:
```
{
	"default_bindings": [
		{
			"controller_type" : "knuckles",
			"binding_url" : "bindings_index_controller.json"
		},
		{
			"controller_type" : "oculus_touch",
			"binding_url" : "bindings_oculus_touch.json"
		},
		{
			"controller_type" : "vive_controller",
			"binding_url" : "bindings_vive_controller.json"
		},
		{
			"controller_type" : "holographic_controller",
			"binding_url" : "bindings_holographic_controller.json"
		},
		{
			"controller_type" : "generic",
			"binding_url" : "bindings_generic.json"
		}
	],
	"actions": [
		{
			"name": "/actions/demo/in/our_first_action",
			"type": "boolean"
		}
	],
	"localization" : [
		{
			"language_tag": "en_US",
			"/actions/demo/in/our_first_action" : "Our first action"
		}
	]
}
```

In the `default_bindings` section we're defining our default binding file for each controller.
We'll need to create these files as well, create each file alongside our `actions.json` and just copy the following into these files for now:
```
{
   "bindings" : {
   },
   "controller_type" : "generic",
   "description" : "Bindings for generic",
   "name" : "Bindings for generic"
}

```
Be sure to change `controller_type` to match the controller type in the bindings list.

In the `actions` section we define our actions. Each action has a `name` and a `type`. Right now we're only defining one action and we don't have any of our default actions. As a result none of the default button and axis logic in Godot will work. We do still track the HMD and the controllers.

The supported types are:
- `boolean`, a button that can be pressed or released
- `vector1`, a value between -1.0 and 1.0 (or 0.0 and 1.0 depending on the input), so for a trigger
- `vector2`, a coordinate between `vec(-1.0, -1.0)` and `vec(1.0, 1.0)`
- `pose`, a tracked location on the controller
- `vibration`, the only output type that allows you to controll a haptic output.

In the `localization` section you can provide human readable descriptions for each action.

In order to tell our plugin that we want to use this alternative file we need to change our ARVROrigins `_ready` function like so:

```
extends ARVROrigin

var OpenVRConfig = null

func _ready():
	# Load our config before we initialise
	OpenVRConfig = preload("res://addons/godot-openvr/OpenVRConfig.gdns");
	if OpenVRConfig:
		OpenVRConfig = OpenVRConfig.new()
		
		OpenVRConfig.action_json_path = "res://ovr_actions/actions.json"
		OpenVRConfig.default_action_set = "/actions/demo"

	# Find the interface and initialise
	var arvr_interface = ARVRServer.find_interface("OpenVR")
	if arvr_interface and arvr_interface.initialize():
		...
```

Here we are loading and then instantiating our OpenVRConfig class. This is an object in the OpenVR plugin that allows you to further configure our plugin.

We use it to set the `action_json_path` property to point to our `actions.json` file.
We then set `default_actions_set` to `/actions/demo`, we'll talk about action sets in a minute.

It is **very** important these configuration changes are made before you initialize the plugin. Changing them afterwards at best will go unnoticed, at worst have unpredictable effects.

## Binding our actions

We can now run our game to configure our bindings. 
OpenVR will likely inform you of missing bindings and provide you with the bindings interface inside of the HMD but I find it easier to set this up on the desktop. 

[[images/openvr_actions_open_cs.png]]

From there select `SHOW OLD BINDING UI` and select Godot or if that is not available go through `MANAGE CONTROLLER BINDINGS`. You must do this while your game is running.

You should now see this window: 

[[images/openvr_actions_default_bindings.png]]

Just edit our current bindings file. I'm not going to give a full tutorial on setting up bindings here, play around with the interface. For our demo purposes I've set up my controller like so:

[[images/openvr_actions_knuckles.png]]

As you can see I've mapped the right hand trigger to our action. 

Now press the 'Replace Default Binding' to update our bindings in our project.

When you hit the back button you go back to the previous screen and you can select the other controller types to set up default bindings for them as well.

## OpenVRAction

OpenVRAction is a new GDNative object in our plugin that allows us to interact with input actions such as button presses and joystick input. 

Simply create a spatial node in your scene and drag the OpenVRAction.gdns script file into its script property. We assign our `our_first_action` action to the `Pressed Action` property.
You spatial nodes inspector should now look like this:

[[images/openvr_action.png]]

The `Pressed Action` property allows us to identify the button press action we want this node to react to. When the user invokes the action the node will emit a `pressed` signal. When the user releases the button a `released` signal is emitted.
You can also call `is_pressed` to query whether the action is currently invoked. 

The `Analog Action` property allows us to identify the analog action we want to have access to. When set you can call `get_analog` to obtain the current state of that analog input.

The `On Hand` property allows you to set whether you want to react regardless of which controller the action is bound to or if you only want to react for an action bound to a specific hadn.

You can combine a pressed and analog action in a single node. 

**Note** this is only a spatial node to give easy access and visibility to actions that have been configured. It doesn't matter where you create the node, this is one of the cool things around the action system. You may want to add a "throttle" action as a child of the a vehicle body for instance.

## OpenVRPose

OpenVRPose is a new GDNative object in our plugin that allows us to track any pose defined within our actions.

We can add such a pose to our `actions.json` file like so:
```
{
	...
	"actions": [
		...
		{
			"name": "/actions/demo/in/our_first_pose",
			"type": "pose"
		}
	],
	"localization" : [
		{
			"language_tag": "en_US",
			...
			"/actions/demo/in/our_first_pose" : "Our first pose"
		}
	]		
}
```

Now we can run our project and edit our bindings again.
**need to add screenshot**

Simply create a spatial node as a child of our `ARVROrigin` node and drag the OpenVRAction.gdns script file into its script property. 
Now set our `Action` property to `/actions/demo/in/our_first_pose` and our `On Hand` property to `right`. 
You spatial nodes inspector should now look like this:

[[images/openvr_pose.png]]

This node will now automatically be positioned by the OpenVR plugin.

To be continued....

## OpenVRHaptic

OpenVRHaptic is a new GDNative object in our plugin that allows us to provide haptic feedback to the user through our controllers. 

[[images/openvr_haptics.png]]


To be continued....

## OpenVRController

> This has not yet been implemented!

## Action sets

I will write about this soon.
