In our [[Setting up a basic project]] page we set up a project that renders to a single viewport. This means we send our stereoscopic images to the HMD and blit our right eye as a preview to screen. In many cases this is exactly what you want because it is the most efficient way to run things especially if you're not very interested in what is going on on the monitor.

When people are spectating or if you are making a two player game with one player in VR you may want to do something else on screen.

Using a separate viewport also allows you to work around the issue of having to turn `keep_3d_linear` on when using the GLES3 driver.

So we're going to use the project we build in our [[Setting up a basic project]] tutorial and change it. 

First we'll purely change it to show the workaround for the GLES3 driver, then we will extend it to do something more complex.

## Moving everything into a viewport.

We create a new scene and turn this into a 2D scene. I tend to name the root node `Construct` but you can call it what you like.

We add a [TextureRect](https://docs.godotengine.org/en/3.2/classes/class_texturerect.html) node, we'll be using this to render our spectator view. 

Next we add a [Viewport](https://docs.godotengine.org/en/3.2/classes/class_viewport.html) node.

**note** it is very important the `TextureRect` comes before our `Viewport`!

Now we add our `main.tscn` scene as a subscene underneath our viewport.

Our scene tree should look like this:

[[images/openvr_viewport_texturerect.png]]

Save this scene and edit your project settings so this new scene becomes our loading scene:

[[images/openvr_viewport_projectsettings.png]]

## Setting up our viewport node.

With our viewport node selected we can set our viewports properties like so:

[[images/openvr_viewport_settings.png]]

We can leave most settings alone. Of note we don't tick `arvr` here, we will turn this on after we successfully initialise our OpenVR plugin.

We do however give the viewport a `Size`. Now our plugin will automatically resize the viewport to match the resolution requested by OpenVR. The problem we have is that Godot will ignore any viewport that doesn't have a size and therefor our resize never happens. 

Setting the viewport to a small size here will trigger the initialisation of the viewport and result in our plugin successfully overriding this size.

We also turn `Msaa` to 2x, this ensures antialiassing is enabled.

Next we turn `Keep 3d linear` on here, we will do it later on in code but can't hurt already doing it here.

The next one is really important, we set `Update Mode` to `Always`, this is set to visible by default resulting in our viewport **not** being rendered as our viewport is not copied to screen through a `ViewportContainer`. 

We also turn on `Enabled 3d` in `Audio listener`. If you do not turn this on you don't get spatial sound if you add sound to your game later on.

Finally we set our `Shadow Atlas` `Size` property to 4096. Without this shadows will not work!

## Changing our initialisation script

The script we added to our ARVROrigin node is still designed to use our main viewport so lets alter this:

```
extends ARVROrigin

export (NodePath) var viewport = null

func _ready():
	# Find the interface and initialise
	var arvr_interface = ARVRServer.find_interface("OpenVR")
	if arvr_interface and arvr_interface.initialize():
		var vp : Viewport = null
		if viewport:
			vp = get_node(viewport)
			if vp:
				vp.size = arvr_interface.get_render_targetsize()
		
		# No viewport? get our main viewport
		if !vp:
			vp = get_viewport()
		
		# switch to ARVR mode
		vp.arvr = true
		
		# keep linear color space, not needed and thus ignored with the GLES2 renderer
		vp.keep_3d_linear = true
		
		# make sure vsync is disabled or we'll be limited to 60fps
		OS.vsync_enabled = false
		
		# up our physics to 90fps to get in sync with our rendering
		Engine.target_fps = 90
```

At the top we've added an export variable of type [NodePath](https://docs.godotengine.org/en/3.2/classes/class_nodepath.html) which allows us to identify the viewport we are rendering to.

Then after we initialise our plugin we use `get_node(viewport)` to obtain our viewport.
While our plugin will automatically resize our viewport this information is not fed back into our viewport node. Because we do need this we'll perform this copy right here. 

Just to be save we grab our main viewport if an invalid node path was provided.

Finally we now use our variable when setting our `arvr` and `keep_3d_linear` properties.

We now go back to our construct scene and right click on our Main subscene and turn `editable children` on. 

Select our `ARVROrigin` node and use the inspector to assign our viewport to the viewport property:

[[images/openvr_assign_viewport_property.png]]

## Output our texture rect

With the above done we should have HMD working fine but we have no output to screen.

We select our `TextureRect` and set it up as follows:

[[images/openvr_setup_texturerect.png]]

We're leaving `Texture` unset, while we can setup our viewport texture here we run into trouble with the construct order of our nodes. We'll set this in code in a minute.

We tick `Expand` and set `Stretch Mode` to `Keep Aspect Covered`. This will center our image and scale it to fill the screen.

We also tick `Flip V` or our image would be upside down.

Now we scroll down to `CanvasItem` and unfold the `Material` section. Here we create a new material, select [ShaderMaterial](https://docs.godotengine.org/en/3.2/classes/class_shadermaterial.html) from the drop down list.

Click on the material that was created and in the `Shader` property create a new [Shader](https://docs.godotengine.org/en/3.2/classes/class_shader.html).

Add the following code to the shader:

```
shader_type canvas_item;

void fragment() {
	vec4 color = texture(TEXTURE, UV);
	
	// regular Linear -> SRGB conversion
	vec3 a = vec3(0.055);
	color.rgb = mix((vec3(1.0) + a) * pow(color.rgb, vec3(1.0 / 2.4)) - a, 12.92 * color.rgb, lessThan(color.rgb, vec3(0.0031308)));
	
	COLOR = color;
}
```

All this code does is a normal texture lookup but convert the color it fetches from linear to sRGB before outputting it.

Finally we need to create a script on our construct root node and add the following code:

```
extends Node2D

func on_window_size_changed():
	$TextureRect.rect_size = OS.window_size

func _ready():
	$TextureRect.texture = $Viewport.get_texture()
	
	get_tree().get_root().connect("size_changed", self, "on_window_size_changed")
	on_window_size_changed();
```

The `on_window_size_changed` function will resize our `TextureRect` to the new window size.

Then in our `_ready` function we get the texture from our `Viewport` node and assign it to our `TextureRect`.

Finally we connect our `size_changed` signal on Godots root node to our function so it gets called whenever the user resizes the window.
We also call our function to set the initial size.

Now we can run our project and test it out. Note that if you want to see a working example of the above code check out [the demo in the main source repository](https://github.com/GodotVR/godot_openvr/tree/master/demo).

## Moving away from our spectator view

The above nicely fixes our sRGB issue but what if we want to display something completely different on screen?

Let's remove our TextureRect and replace it with a [ViewportContainer](https://docs.godotengine.org/en/3.2/classes/class_viewportcontainer.html). Make sure this is placed above our HMD viewport

We now add a new `ViewPort` as a child to our `ViewportContainer`. This is the viewport we'll use as our output to screen.

We will be resizing our viewport size in code in a minute, the rest of the viewport settings really depends on what you want to do with it. We're going to add a 2D interface in a minute so I'm turning `Hdr` off, `Disable 3d` on and set `Usage` to `2D`.

Next we change the script on our construct node to this:
```
extends Node2D

func on_window_size_changed():
	$ViewportContainer/Viewport.size = OS.window_size

func _ready():
	get_tree().get_root().connect("size_changed", self, "on_window_size_changed")
	on_window_size_changed();
```

So all we're doing here is resizing our viewport to match our window size.

Finally we create a new 2D scene. Having this as a seperate scene makes it easier to test the 2D UI out by itself.
For our example we're just adding a button into this scene that doesn't do anything.

[[images/openvr_2d_ui.png]]

Now we add this as a subscene in our construct scene:

[[images/openvr_viewports_with_ui.png]]

## 3D person spectator camera

One thing that is often done in VR is to output a spectator camera to desktop. 

Especially when this virtual camera is attached to a tracker which is attached to a real camera so you can do green screen composition with the virtual world.

For this we need to make sure our desktop viewport is able to handle 3D rendering by leaving `Disable 3D` off and setting `Usage` to `3D`.

Then we add a camera to our desktop viewport.

All the ARVR nodes have to remain in our VR viewport and we'll need to add the tracked puck as a 3rd ARVRController to the scene. 
We can use a [RemoteTransform](https://docs.godotengine.org/en/3.2/classes/class_remotetransform.html) node to copy this trackers position to our camera. 

3D objects you want visible both in VR and on the desktop should be added after the viewports. Again I suggest creating a new scene for these.

Our construct scene should look something like this:

[[images/OpenVR_viewport_spectatorcam.png]]

This however is a bigger topic for which we'll make a demo available in due time.
