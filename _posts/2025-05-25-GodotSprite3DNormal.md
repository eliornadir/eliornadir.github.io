---
title:  "Godot Engine: Creating an Animated Sprite with Normal Textures in 3D"
categories:
  - GameDev
tags:
  - Godot
  - C#
  - 2D
  - 3D
  - NormalMapping
  - Sprites
last_modified_at: 2025-05-27 00:00:00 -0400
excerpt: "A guide to creating a 2D sprite in a 3D environment in Godot that supports normal textures."
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/ExZ2C4FcKY8?si=syzh2w6mBdVjHws5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This guide will walk you through creating a 2D sprite in a 3D environment in Godot that supports [normal textures](https://en.wikipedia.org/wiki/Normal_mapping).

The full code for this project can be found here: [eliornadir/sprite-in-3d-normals](https://github.com/eliornadir/sprite-in-3d-normals)

>📝 Note: This guide assumes you have a basic understanding of Godot and its 3D scene system as well as general programming knowledge. The scripting will be done in C#.

There will be multiple reminders to save your work throughout the guide. Please remember to save your work often to avoid losing any progress.

When code blocks are provided, they will either be the entire code of the target file, or the code that needs to be added to the file. If you see the following decorations, they mean that you should only add the code to the file that is inbetween the decorations. This is to help you find the right place to add the code.
```c#
// ** Begin Additions **
// ** End Additions **
```

Example:
```c#
    // Code to reference for finding your place in the file
    Console.WriteLine("Hello World");

    // ** Begin Additions **
    // Add the code here
    // ** End Additions **

    // Code to reference for finding your place in the file
    Console.WriteLine("Goodbye World");
``` 

Ideally, you should type the code out as it is a good learning exercise. If you are not comfortable with that, you can copy and paste the code, but be sure to understand what it does.

## Setting the Table: Create and Stage the Main Scene

### Create the Main Scene
1. Create a new Godot project
2. Create a new scene and add a Node3D as the root node
3. Rename the node to Main

### Create the Ground
1. Child a StaticBody3D node to the Main node and name it Ground
2. Set the Y Position of the Ground node to -0.5
3. Child a MeshInstance3D to the Ground node
4. Add a BoxMesh to the instance
5. Set the size to 60,1,60
6. Add a StandardMaterial3D to the Mesh
7. Set the Albedo color to a more natural color like a light brown (#9b7653)
8. Child a CollisionShape3D to the Ground node
9. Add a BoxShape to the CollisionShape3D
10. Set the size to 60,1,60

>⚠️ Remember to save!

### Add Objects
1. Add some objects to test collision and shadow casting
2. Child a StaticBody3D node to the Main node and name it Pillar
3. Set the Y position of the Pillar to 2.5 and the X to -5
4. Child a MeshInstance3D to the Pillar node
5. Add a CylinderMesh to the instance
6. Set the height to 5.0m
6. Child a CollisionShape3D to the Pillar node
7. Add a CylinderShape to the CollisionShape3D
8. Set the height to 5.0m
9. Duplicate the Pillar node by pressing Ctrl+D or right-clicking and selecting Duplicate
10. Move the duplicate to the right by setting the X position to 5
11. Select both Pillar and Pillar2 nodes and duplicate them again
12. Set the X position of Pillar3 and Pillar4 to 0
13. Set the Z position of Pillar3 to -5 and Pillar4 to 5

>⚠️ Remember to save!

### Add Lighting
1. Add a DirectionalLight3D node to the Main node
2. Set the Y position to 10 just to get it out of the way
3. Under the Light3D > Shadow section, enable the "Enabled" checkbox
4. Set the X rotation to -45 degrees and the Y rotation to -38.5 degrees
5. Add an OmniLight3D node to the Main node and name it Light
6. Under the Light3D > Shadow section, enable the "Enabled" checkbox
7. Set the X position to -4 and the Y position to 2.5
8. Duplicate the Light node and set the X position to 4
9. Select both Light and Light2 nodes and duplicate them again
10. Set the Z position of Light3 to -4 and Light4 to 4. Set the X position for both to 0
11. Create a new Node3D and name it Lights. Child all the lights to this node so you can hide them all at once if needed.

>⚠️ Remember to save!

## Getting the Ingredients: Create the SpriteCollection Scene

This scene will hold the textures and settings for the 2D sprites in 3D. It represents one "animation" of a sprite (i.e. a running animation, an idle animation, etc.). You can create multiple instances of this scene to represent different animations or characters.

1. Create a new scene and add a Node as the root node
2. Rename the node to SpriteCollection
3. Click the "Attach Script" button and create a new script named SpriteCollection.cs

```c#
using Godot;

public partial class SpriteCollection : Node {
    [Export]
    public CompressedTexture2D[] AlbedoTextures { get; set; } = [];

    [Export]
    public CompressedTexture2D[] NormalTextures { get; set; } = [];

    [Export]
    public CompressedTexture2D[] OcclusionTextures { get; set; } = [];

    [Export]
    public BaseMaterial3D.BillboardModeEnum Billboard { get; set; } = BaseMaterial3D.BillboardModeEnum.Enabled;

    [Export]
    public BaseMaterial3D.TextureFilterEnum TextureFilter { get; set; } = BaseMaterial3D.TextureFilterEnum.NearestWithMipmapsAnisotropic;

    [Export]
    public float AnimationSpeedScale { get; set; } = 5.0f;
}
```
- The **AlbedoTextures** array will hold the textures for the sprite's appearance.
- The **NormalTextures** array will hold the normal maps.
- The **OcclusionTextures** array will hold the occlusion maps.
- The **Billboard** property allows you to set how the sprite behaves in relation to the camera
- The **TextureFilter** property sets how the textures are filtered.
- The **AnimationSpeedScale** property allows you to control the speed of the sprite's animation.

>⚠️ Remember to save!

### Downloading Textures
You can use any 2D sprite textures you like if you have them. For this example, we will be using some free sprites from GandalfHardcore's Male and Female 2D Sprite Pack available on itch.io. These sprites come in a sheet, but this procedure requires individual textures for each frame of the animation. The extracted frames can be directly downloaded from this [link](https://github.com/eliornadir/sprite-in-3d-normals/raw/refs/heads/main/sample_textures.zip). Extract the zip file and place the "art" folder in the root of your project. The folder structure should look like this:

```
res://
├── art
├──── sprites
├────── generic_warrior
├──────── idle
├──────── walk
```

One gotcha that was discovered while testing is that the textures need be uncompressed. The process requires access to the texture's alpha channel, and if the texture is compressed, this will not work. To fix this, you can set the compression to "Lossless" in the import settings of the texture. You can do this by selecting all of the textures in the FileSystem panel, clicking the "Import" tab, and setting the "Compress > Mode" option to "Lossless". You should then click the "Reimport" button to apply the changes. This will ensure that the textures are not compressed and can be used correctly.

>📝 Note: The texture download contains both albedo and normal maps. The normal maps were generated using [Laigter](https://azagaya.itch.io/laigter), a free tool for generating normal maps from 2D images. You can use any tool you prefer to create normal maps, but creating them is outside the scope of this guide.

## The Meat and Potatoes: Player Scene

The CharacterBody3D scene will be where the majority of the logic happens. This scene will handle the loading of the sprite textures, setting up the materials, and animating the sprite.

### Scaffolding the Player Scene
1. Create a new scene and add a CharacterBody3D as the root node
2. Rename the node to Player
3. Add a MeshInstance3D node as a child of the Player node
4. Rename the node to SpriteMesh
5. Add a CollisionShape3D node as a child of the Player node
6. Rename the node to SpriteCollisionShape
7. Add a BoxShape to the CollisionShape3D node (this is just to remove the warning message, we will not be using it)
8. Add an AnimationPlayer node as a child of the Player node
9. Rename the node to SpriteAnimation

>⚠️ Remember to save!

### Building the Exports
1. Select the Player node and click the "Attach Script" button
2. Create a new script named Player.cs
3. Add the following code to the Player.cs script:

```c#
using Godot;
using System;

public partial class Player : CharacterBody3D {
    // ** Begin Additions **
	[Export]
	public int WalkSpeed { get; set; } = 5;

	[Export]
	public float CollisionDepth { get; set; } = 0.8f;

	[ExportGroup("Sprite")]
	[Export]
	public MeshInstance3D SpriteMeshInstance { get; set; } = null!;

	[Export]
	public AnimationPlayer SpriteAnimationPlayer { get; set; } = null!;

	[Export]
	public CollisionShape3D SpriteCollisionShape { get; set; } = null!;

	[Export]
	public Godot.Collections.Array<SpriteCollection> Sprites { get; set; } = [];
	
	private SpriteCollection _currentSpriteCollection = null!;
	private CompressedTexture2D[] _spriteAlbedoTextures => _currentSpriteCollection.AlbedoTextures;
	private CompressedTexture2D[] _spriteNormalTextures => _currentSpriteCollection.NormalTextures;
	private CompressedTexture2D[] _spriteOcclusionTextures => _currentSpriteCollection.OcclusionTextures;
	private BaseMaterial3D.BillboardModeEnum _spriteBillboard => _currentSpriteCollection.Billboard;
	private BaseMaterial3D.TextureFilterEnum _spriteTextureFilter => _currentSpriteCollection.TextureFilter;
	private float _spriteAnimationSpeedScale => _currentSpriteCollection.AnimationSpeedScale;
	private Vector3 _targetVelocity = Vector3.Zero;
    // ** End Additions **
    // ...
}
```
- The **WalkSpeed** property sets the speed of the player.
- The **CollisionDepth** property sets the depth of the collision shape that we will be generating later.
- The **SpriteMeshInstance** property is a reference to the MeshInstance3D node that will be used to display the sprite.
- The **SpriteAnimationPlayer** property is a reference to the AnimationPlayer node that will be used to animate the sprite.
- The **SpriteCollisionShape** property is a reference to the CollisionShape3D node that will be used to detect collisions.
- The **Sprites** property is an array of SpriteCollection nodes that will be used to load the textures and settings for the sprite.
- The **_currentSpriteCollection** property is a reference to the current SpriteCollection node that is being used.
- The **_spriteAlbedoTextures** property is an array of the albedo textures for the current sprite.
- The **_spriteNormalTextures** property is an array of the normal textures for the current sprite.
- The **_spriteOcclusionTextures** property is an array of the occlusion textures for the current sprite.
- The **_spriteBillboard** property is the billboard mode for the current sprite.
- The **_spriteTextureFilter** property is the texture filter for the current sprite.
- The **SpriteAnimationSpeedScale** property is the speed scale for the current sprite animation.
- The **_targetVelocity** property is a Vector3 that will be used to store the target velocity of the player.

4. Save the script and return to the Player scene in the Godot editor.
5. Build the C# project by clicking on the "Build" button in the top right corner of the editor or by pressing `Alt+B`.
6. After the build is complete, click on the Player node in the scene tree, expand the "Sprite" section, and set the following properties in the Inspector panel by dragging the nodes from the scene tree into the corresponding properties:
   - Set the **Sprite Mesh Instance** property to the **SpriteMesh** node.
   - Set the **Sprite Animation Player** property to the **SpriteAnimation** node.
   - Set the **Sprite Collision Shape** property to the **SpriteCollisionShape** node.

>⚠️ Remember to save!

### Add Movement Logic
1. Under Project > Project Settings, go to the Input Map tab
2. Add the following actions with the corresponding Keyboard bindings:
    - `PlayerMoveUp`: `W` and/or `Up`
    - `PlayerMoveLeft`: `A` and/or `Left`
    - `PlayerMoveDown`: `S` and/or `Down`
    - `PlayerMoveRight`: `D` and/or `Right`
3. Back in the Player.cs script, add the following code:

```c#
    private float _spriteAnimationSpeedScale => _currentSpriteCollection.AnimationSpeedScale;
    private Vector3 _targetVelocity = Vector3.Zero;

    // ** Begin Additions **
    public static class InputActions {
		public const string MoveUp = "PlayerMoveUp";
		public const string MoveLeft = "PlayerMoveLeft";
		public const string MoveDown = "PlayerMoveDown";
		public const string MoveRight = "PlayerMoveRight";
	}
    // ** End Additions **
    // ...
```
- The **InputActions** class is a static class that holds the string constants for the input actions. This allows you to reference the input actions by name instead of by string, which is less error-prone.

```c#
    public static class InputActions {
		public const string MoveUp = "PlayerMoveUp";
		public const string MoveLeft = "PlayerMoveLeft";
		public const string MoveDown = "PlayerMoveDown";
		public const string MoveRight = "PlayerMoveRight";
	}
    // ** Begin Additions **
    public override void _Ready() {
        if (Sprites.Count == 0 || Sprites[0] == null) {
			GD.PrintErr("No sprite collections provided.");
			return;
		}
		_currentSpriteCollection = Sprites[0];
		if (SpriteMeshInstance == null) {
			GD.PrintErr("SpriteMeshInstance is not assigned.");
			return;
		}
		if (SpriteAnimationPlayer == null) {
			GD.PrintErr("SpriteAnimationPlayer is not assigned.");
			return;
		}
		if (SpriteCollisionShape == null) {
			GD.PrintErr("SpriteCollisionShape is not assigned.");
			return;
		}
    }
    // ** End Additions **
```
- The **_Ready** method is called when the node is added to the scene. You may have already had this method in your script depending on which template you chose, but if not, you can add it now. We're adding some error checking to ensure that the sprite collections and other properties are set correctly. If they are not, an error message will be printed to the console. We'll be back to this method later to add a bit more functionality.

```c#
    public override void _Ready() {
        //...
    }
    // ** Begin Additions **
    public override void _PhysicsProcess(double delta) {
		Vector3 direction = Vector3.Zero;
		if (Input.IsActionPressed(InputActions.MoveUp)) {
			direction += Vector3.Forward;
		}
		if (Input.IsActionPressed(InputActions.MoveLeft)) {
			direction += Vector3.Left;
		}
		if (Input.IsActionPressed(InputActions.MoveDown)) {
			direction += Vector3.Back;
		}
		if (Input.IsActionPressed(InputActions.MoveRight)) {
			direction += Vector3.Right;
		}
		if (direction != Vector3.Zero) {
			direction = direction.Normalized();
		}
		_targetVelocity.X = direction.X * WalkSpeed;
		_targetVelocity.Z = direction.Z * WalkSpeed;
		Velocity = _targetVelocity;
		MoveAndSlide();
	}
    // ** End Additions **
```
- The **_PhysicsProcess** method is called during physics processing and is where we will handle the player movement. We check for input actions and set the target direction based on the input. We then set the target velocity based on the direction and `WalkSpeed`. Finally, we call the **MoveAndSlide** method to move the player.

>📝 Note: We are only working in 2D movement (X and Z planes) for this example. You can add more movement logic if you want to allow for 3D movement, but that is outside the scope of this guide.

We now have basic movement set up for the player. However, if you run the game now, you will see that the sprite is not visible and there are no animations. We will fix that soon. But first we need to add a small class to store some general world information: `Globals.cs`.

### Create the Globals Class
1. Create a new script in the root of your project and name it Globals.cs
2. Add the following code to the Globals.cs script:

```c#
public static class Globals {
    public static class World {
        // The number of pixels per meter in the game world.
        public const float PixelsPerMeter = 24.74f;
    }
}
```
- The **Globals** class is a static class that holds global constants and settings for the game. This is a good place to store any constants or settings that you want to access from anywhere in the game. Because we are only storing a single constant in this example, we could have just used a static constant in the Player.cs script, but this is a good practice to get into. You can add more constants or settings to this class as needed.
- The **PixelsPerMeter** constant is used to convert between pixels and meters in the game world. This is important for the sprite scaling and movement calculations. You can adjust this value based on your needs and the size of your sprites. For this example, we are assuming that our generic warrior character is 5' 10" tall (1.778m) and that the sprite is 44 pixels tall. This gives us a value of ~24.74 pixels per meter. You can adjust this value based on your needs and the size of your sprites.

> 📝 Note: There is a small bug in Godot 4.4 that may cause this error after adding the Globals class:
> ```
> ERROR: Script class can only be set together with base class name
> ```
> It can be safely ignored and will not affect the functionality of the game. It is a known issue that will hopefully be fixed in a future release of Godot.

### Add Sprite Loading

1. In the Player.cs script, Add the following two addition blocks:

```c#
// ** Begin Additions **
using System.Linq;
// ** End Additions **
using Godot;

public partial class Player : CharacterBody3D {
    //...
    public override void _PhysicsProcess(double delta) {
        //...
    }
    // ** Begin Additions **
    private void LoadSprite() {
        if (_spriteAlbedoTextures.Length == 0) {
			GD.PrintErr("No albedo textures provided.");
			return;
		}

		// Check if all albedo textures have the same size
		Vector2 albedoTextureSize = _spriteAlbedoTextures[0].GetSize();
		if (!_spriteAlbedoTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
			GD.PrintErr("Albedo textures do not have a uniform size.");
			return;
		}

		// Create the material to be used for the quad
		StandardMaterial3D material = new() {
			Transparency = BaseMaterial3D.TransparencyEnum.AlphaDepthPrePass,
			CullMode = BaseMaterial3D.CullModeEnum.Disabled,
			AlbedoTexture = _spriteAlbedoTextures[0],
			BillboardMode = _spriteBillboard,
			TextureFilter = _spriteTextureFilter
		};

		// If there are normal textures, check if they have the same size as the albedo textures
		// and set the first normal texture in the material
		if (_spriteNormalTextures.Length > 0) {
			if (!_spriteNormalTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
				GD.PrintErr("Normal textures must have the same size as the albedo textures.");
				return;
			}
			material.NormalEnabled = true;
			material.NormalTexture = _spriteNormalTextures[0];
		}

		// If there are occlusion textures, check if they have the same size as the albedo textures
		// and set the first occlusion texture in the material
		if (_spriteOcclusionTextures.Length > 0) {
			if (!_spriteOcclusionTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
				GD.PrintErr("Occlusion textures must have the same size as the albedo textures.");
				return;
			}
			material.AOEnabled = true;
			material.AOTexture = _spriteOcclusionTextures[0];
		}

		// Create the quad mesh and assign the material
		QuadMesh quadMesh = new() {
			Size = albedoTextureSize / Globals.World.PixelsPerMeter,
			Material = material
		};

		// Assign the quad mesh to the SpriteMeshInstance node
		SpriteMeshInstance.Mesh = quadMesh;
    }
    // ** End Additions **
}
```

That's a lot of code, so let's break it down:

```c#
if (_spriteAlbedoTextures.Length == 0) {
	GD.PrintErr("No albedo textures provided.");
	return;
}

// Check if all albedo textures have the same size
Vector2 albedoTextureSize = _spriteAlbedoTextures[0].GetSize();
if (!_spriteAlbedoTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
	GD.PrintErr("Albedo textures do not have a uniform size.");
	return;
}
```
The first block checks if there are any albedo textures provided. If not, an error message is printed and the method returns.

The second block checks if all albedo textures have the same size. If not, an error message is printed and the method returns. (This is why we needed to add `System.Linq` at the top of the file: to use the `.All()` extension method) This is important because we need to ensure that all textures are the same size for the animations to work correctly. The `IsEqualApprox` method compares the sizes of the textures, allowing for a small margin of error (as recommended by Godot's documentation for comparing vectors).

```c#
// Create the material to be used for the quad
StandardMaterial3D material = new() {
	Transparency = BaseMaterial3D.TransparencyEnum.AlphaDepthPrePass,
	CullMode = BaseMaterial3D.CullModeEnum.Disabled,
	AlbedoTexture = _spriteAlbedoTextures[0],
	BillboardMode = _spriteBillboard,
	TextureFilter = _spriteTextureFilter
};
```
Here we create a new `StandardMaterial3D` object and set its properties. The `Transparency` property is set to `AlphaDepthPrePass` instead of `Alpha` to allow for the sprite to cast shadows. The `CullMode` property is set to `Disabled` to allow the sprite to be visible from both sides. The `AlbedoTexture`, `BillboardMode`, and `TextureFilter` properties are set based on the values from the `SpriteCollection` node.

```c#
// If there are normal textures, check if they have the same size as the albedo textures
// and set the first normal texture in the material
if (_spriteNormalTextures.Length > 0) {
	if (!_spriteNormalTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
		GD.PrintErr("Normal textures must have the same size as the albedo textures.");
		return;
	}
	material.NormalEnabled = true;
	material.NormalTexture = _spriteNormalTextures[0];
}

// If there are occlusion textures, check if they have the same size as the albedo textures
// and set the first occlusion texture in the material
if (_spriteOcclusionTextures.Length > 0) {
	if (!_spriteOcclusionTextures.All(t => t.GetSize().IsEqualApprox(albedoTextureSize))) {
		GD.PrintErr("Occlusion textures must have the same size as the albedo textures.");
		return;
	}
	material.AOEnabled = true;
	material.AOTexture = _spriteOcclusionTextures[0];
}
```
The next two blocks check if there are any normal or occlusion textures provided. If so, they check if the sizes match the albedo textures and set the corresponding properties in the material. If the sizes do not match, an error message is printed and the method returns.

```c#
// Create the quad mesh and assign the material
QuadMesh quadMesh = new() {
	Size = albedoTextureSize / Globals.World.PixelsPerMeter,
	Material = material
};

// Assign the quad mesh to the SpriteMeshInstance node
SpriteMeshInstance.Mesh = quadMesh;
```

Finally, we create a new `QuadMesh` object and set its size based on the albedo texture size divided by the `PixelsPerMeter` constant. We then assign the material to the quad mesh and set the mesh of the `SpriteMeshInstance` node to the quad mesh.

Before we can see the sprite, we need to call the `LoadSprite` method in the `_Ready` method.
```c#
    public override void _Ready() {
        //...
        if (SpriteCollisionShape == null) {
			GD.PrintErr("SpriteCollisionShape is not assigned.");
			return;
		}
        // ** Begin Additions **
        LoadSprite();
        // ** End Additions **
    }
```

## Main Scene: Finishing Touches

Head back to the Main scene and build the project again to make sure everything is up to date. 

1. Add a Node as a child of the Main node
2. Rename the node to SpriteCollections.
3. Instantiate the SpriteCollection scene as a child of the SpriteCollections node
4. Rename the instance to SpriteCollectionIdle.
5. Under the `art/sprites/generic_warrior/idle` folder, select all of the albedo textures (the ones that are what you want the sprite to look like) and drag them into the "Albedo Textures" array in the Inspector panel for the SpriteCollectionIdle node.
6. Select all of the normal textures (the ones that are named the same as the albedo textures but have an extra `_n` in the name) and drag them into the "Normal Textures" array in the Inspector panel for the SpriteCollectionIdle node.
7. Instantiate the SpriteCollection scene again as a child of the SpriteCollections node
8. Rename the instance to SpriteCollectionWalk.
9. Under the `art/sprites/generic_warrior/walk` folder, select all of the albedo textures and drag them into the "Albedo Textures" array in the Inspector panel for the SpriteCollectionWalk node.
10. Select all of the normal textures and drag them into the "Normal Textures" array in the Inspector panel for the SpriteCollectionWalk node.
11. Instantiate the Player scene as a child of the Main node.
12. Under the **Sprites** property of the Player instance, click the "Add Element" button and select the **SpriteCollectionIdle** node for index 0. Repeat this step to add the **SpriteCollectionWalk** node to index 1. You can also drag and drop the nodes from the scene tree into the array in the Inspector panel.

>📝 Note: We will be referencing these SpriteCollection nodes by their numerical index in code, and index 0 will be the "default" sprite and animation that will be loaded when the game starts. This could be improved to reference the nodes by their name, but for now be aware of which sprite collections are related to each index.

13. Add a Node3D as a child of the Player node and rename it to CameraPivot
14. Set the X rotation of the CameraPivot node to -35 degrees
15. Add a Camera3D node as a child of the CameraPivot node and rename it to PlayerCamera
16. Set the Y position of the PlayerCamera node to -1 and the Z position to 4

⚠️ Remember to save!

You should now be able to run the game and see the player sprite in the 3D scene. The sprite should be facing the camera and casting shadows on the ground and you should be able to move the player around using the WASD or arrow keys.

## Adding Animation
Back in the Player.cs script, we need to continue adding functionality to the `LoadSprite` method to handle the animations.

```c#
private void LoadSprite() {
    //...
    SpriteMeshInstance.Mesh = quadMesh;
    // ** Begin Additions **
    // If there are multiple albedo textures, create an animation to cycle through them
    if (_spriteAlbedoTextures.Length > 1) {
        SpriteAnimationPlayer.SpeedScale = _spriteAnimationSpeedScale;

		Animation animation = new() {
			Length = _spriteAlbedoTextures.Length,
			LoopMode = Animation.LoopModeEnum.Linear
		};

        int albedoAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
        int normalAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
        int occlusionAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
        string spriteMeshName = SpriteMeshInstance.Name;
        animation.TrackSetPath(albedoAnimationTrackIdx, $"{spriteMeshName}:mesh:material:albedo_texture");
        if (_spriteNormalTextures.Length > 0) {
            animation.TrackSetPath(normalAnimationTrackIdx, $"{spriteMeshName}:mesh:material:normal_texture");
        }
        if (_spriteOcclusionTextures.Length > 0) {
            animation.TrackSetPath(occlusionAnimationTrackIdx, $"{spriteMeshName}:mesh:material:ao_texture");
        }
        for (int i = 0; i < _spriteAlbedoTextures.Length; i++) {
            // Add a track to change the albedo texture
            animation.TrackInsertKey(albedoAnimationTrackIdx, i, _spriteAlbedoTextures[i]);
            if (_spriteNormalTextures.Length > 0) {
                int ni = i <= _spriteNormalTextures.Length - 1 ? i : _spriteNormalTextures.Length - 1;
                animation.TrackInsertKey(normalAnimationTrackIdx, i, _spriteNormalTextures[ni]);
            }
            if (_spriteOcclusionTextures.Length > 0) {
                int oi = i <= _spriteOcclusionTextures.Length - 1 ? i : _spriteOcclusionTextures.Length - 1;
                animation.TrackInsertKey(occlusionAnimationTrackIdx, i, _spriteOcclusionTextures[oi]);
            }
        }

        if (!SpriteAnimationPlayer.HasAnimationLibrary(string.Empty)) {
            SpriteAnimationPlayer.AddAnimationLibrary(string.Empty, new AnimationLibrary());
        }
        SpriteAnimationPlayer.GetAnimationLibrary(string.Empty).AddAnimation("sprite_sequence", animation);
        if (SpriteAnimationPlayer.HasAnimation("sprite_sequence")) {
            SpriteAnimationPlayer.Stop();
            SpriteAnimationPlayer.Play("sprite_sequence");
        }
	}
    // ** End Additions **
}
```
Let's go through the new code step by step:

```c#
if (_spriteAlbedoTextures.Length > 1) {
    SpriteAnimationPlayer.SpeedScale = _spriteAnimationSpeedScale;

	Animation animation = new() {
		Length = _spriteAlbedoTextures.Length,
		LoopMode = Animation.LoopModeEnum.Linear
	};
```
We first check if there are multiple albedo textures provided. If so, we create a new `Animation` object and set its length to the number of albedo textures. We also set the loop mode to `Linear` so that the animation will loop through the textures.

```c#
int albedoAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
int normalAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
int occlusionAnimationTrackIdx = animation.AddTrack(Animation.TrackType.Value);
string spriteMeshName = SpriteMeshInstance.Name;
animation.TrackSetPath(albedoAnimationTrackIdx, $"{spriteMeshName}:mesh:material:albedo_texture");
if (_spriteNormalTextures.Length > 0) {
    animation.TrackSetPath(normalAnimationTrackIdx, $"{spriteMeshName}:mesh:material:normal_texture");
}
if (_spriteOcclusionTextures.Length > 0) {
    animation.TrackSetPath(occlusionAnimationTrackIdx, $"{spriteMeshName}:mesh:material:ao_texture");
}
```
We then add three tracks to the animation: one for the albedo texture, one for the normal texture, and one for the occlusion texture. We set the path of each track to the corresponding property of the `SpriteMeshInstance` node's material. If there are no normal or occlusion textures provided, we skip setting the path for those tracks.

```c#
for (int i = 0; i < _spriteAlbedoTextures.Length; i++) {
    // Add a track to change the albedo texture
    animation.TrackInsertKey(albedoAnimationTrackIdx, i, _spriteAlbedoTextures[i]);
    if (_spriteNormalTextures.Length > 0) {
        int ni = i <= _spriteNormalTextures.Length - 1 ? i : _spriteNormalTextures.Length - 1;
        animation.TrackInsertKey(normalAnimationTrackIdx, i, _spriteNormalTextures[ni]);
    }
    if (_spriteOcclusionTextures.Length > 0) {
        int oi = i <= _spriteOcclusionTextures.Length - 1 ? i : _spriteOcclusionTextures.Length - 1;
        animation.TrackInsertKey(occlusionAnimationTrackIdx, i, _spriteOcclusionTextures[oi]);
    }
}
```
Next, we loop through the albedo textures and add a key to each track for each texture. We also check if there are normal or occlusion textures provided and add keys for those as well. If there are not enough normal or occlusion textures to match the number of albedo textures, we use the last texture in the array to avoid an out-of-bounds error and to ensure that the animation still plays smoothly.

```c#
if (!SpriteAnimationPlayer.HasAnimationLibrary(string.Empty)) {
    SpriteAnimationPlayer.AddAnimationLibrary(string.Empty, new AnimationLibrary());
}
SpriteAnimationPlayer.GetAnimationLibrary(string.Empty).AddAnimation("sprite_sequence", animation);
if (SpriteAnimationPlayer.HasAnimation("sprite_sequence")) {
    SpriteAnimationPlayer.Stop();
    SpriteAnimationPlayer.Play("sprite_sequence");
}
```
Finally, we check if the `SpriteAnimationPlayer` has a Global animation library. If not, we create a new one. We then add the animation to the library and play it. The animation will now cycle through the albedo textures and update the material properties accordingly.

You should now be able to run the game and see the sprite animating. Let's add a check for if the player is moving and switch between the idle and walk animations.

```c#
public override void _PhysicsProcess(double delta) {
    //...
    if (targetDirection != Vector3.Zero) {
			targetDirection = targetDirection.Normalized();
            // ** Begin Additions **
            ChangeSprite(1);
            // ** End Additions **
    }
    // ** Begin Additions **
    else {
        ChangeSprite(0);
    }
    // ** End Additions **
    // ...
}
// ** Begin Additions **
public void ChangeSprite(int index) {
	if (index < 0 || index >= Sprites.Count) {
		GD.PrintErr($"Invalid sprite collection index: {index}");
		return;
	}
	if (Sprites[index].Name == _currentSpriteCollection.Name) {
		return;
	}
	_currentSpriteCollection = Sprites[index];
	LoadSprite();
}
// ** End Additions **
```
We've added a new method called `ChangeSprite` that takes an index as a parameter. This method checks if the index is valid and if the sprite collection at that index is different from the current one. If so, it updates the `_currentSpriteCollection` to the new sprite collection and calls the `LoadSprite` method to update the sprite mesh.

In the `_PhysicsProcess` method, we check if the target direction is not zero (i.e. the player is moving). If so, we call the `ChangeSprite` method with index 1 to switch to the walk animation. If the player is not moving, we call the `ChangeSprite` method with index 0 to switch to the idle animation.

You should now be able to run the game and see the player sprite animating between the idle and walk animations based on the player's movement. The sprite will switch to the walk animation when the player is moving and back to the idle animation when the player stops moving.

## Adding Dynamic Collision Shapes
Now that we have the sprite animating, we can add a dynamic collision shape to the player based on the size of the sprite.

In the Player.cs script we'll add some logic at the end of the `LoadSprite` method to create a dynamic collision shape based on the size of the sprite:

```c#
private void LoadSprite() {
    //...
    if (_spriteAlbedoTextures.Length > 1) {
        //...
	}
    // ** Begin Additions **
    Rect2I collisionShapeRect = _spriteAlbedoTextures[0].GetImage().GetUsedRect();
	GD.Print($"Collision shape rect: {collisionShapeRect}");
	BoxShape3D boxShape = new() {
		Size = new(
            collisionShapeRect.Size.Abs().X / Globals.World.PixelsPerMeter,
            collisionShapeRect.Size.Abs().Y / Globals.World.PixelsPerMeter,
            CollisionDepth
        )
	};
	SpriteCollisionShape.Shape = boxShape;
    // ** End Additions **
}
```
The `GetUsedRect` method returns a rectangle that contains all non-transparent pixels in the texture. This requires the textures to be uncompressed, which is why we set the compression mode to "Lossless" earlier. We are only currently using the first albedo texture to determine the collision shape, but you could modify this logic to use a different texture or to calculate the collision shape based on multiple textures if needed.

We then create a new `BoxShape3D` object and set its size based on the rectangle's size divided by the `PixelsPerMeter` constant. The Z size is set to the `CollisionDepth` property that we defined earlier.

Finally, we assign the `BoxShape3D` to the `SpriteCollisionShape.Shape` property to update the collision shape of the player.

You should now be able to run the game and see the player sprite animating with a dynamic collision shape that matches the size of the sprite. The collision shape will update automatically based on the size of the sprite, allowing for accurate collision detection.

## Conclusion
You have successfully created a 3D sprite with normal maps in Godot 4 using C#. You now have a player character that can move around the scene, animate between idle and walking states, and has a dynamic collision shape based on the size of the sprite.

I will be working on making video tutorials for this and any other topics that I cover in the future, so be sure to check out my [YouTube channel](https://www.youtube.com/@elior-nadir) for that. I am fairly new to Godot, but I have a lot of experience with C#, so I hope to be able to share my knowledge.

If you have any questions or feedback, feel free to reach out to me on [X](https://x.com/EliorNadir) or [GitHub](https://github.com/eliornadir).

May Yahweh bless you. If you don't know Him yet, I encourage you to seek Him out. He is the Creator of the universe and desires a personal relationship with you. He came to Earth as a man, Yeshua (Jesus) and died for the sins of all of us, and each of us. You can learn more about Him through [His word](https://www.biblegateway.com/passage/?search=Isaiah%2053&version=LEB). I would start with the [Gospel of John](https://www.biblegateway.com/passage/?search=John%201&version=LEB). If you have any questions, feel free to reach out to me.