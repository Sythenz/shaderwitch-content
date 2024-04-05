+++
archetype = "article"
title = "Virtual Shader Paths"
weight = 1
Author = "Alessa \"Codekitten\" Baker"
Author_url = "/Authors/alessa-codekitten-baker"
UnrealVersion = "5.2"
type="article"
+++

{{< article-header >}}

Virtual Shader Paths are probably the simplest way in Unreal Engine to include and use your own custom HLSL Global Shaders.
They can also be included within Plugins making them easily sharable.

A more in-depth method of actually implementing this as a globally accessible shader can be found in the sources below [[1]](#additional-resources).

## C++ Plugin Setup

Setting this up is remarkably easy, and requires a single method called in StartupModule from ``ShaderCore``. It should
also be added to the Runtime Module of your plugin since the shader also needs to be accessible in your cooked game.

``AddShaderSourceDirectoryMapping()`` takes two parameters, the first is the Virtual path - this is a path that just
gives unreal a relative place to reference and is what we will use later in our Custom Node. The second is the actual
path, in this specific example we use the plugin manager to find the base directory of our plugin and a folder called 
"shaders" within that.

Unreal handles all platform support here, since we are using the plugin manager to find the directory - this will return
a path that is platform dependent. From experience this works on consoles and mobile too!

It is also possible to get the relative path to the project here and use a project relative shader directory, but
generally speaking it is better practice to contain everything within a plugin that can be disabled if something goes 
wrong in production.

```cpp
void FMyCoolRuntimeModule::StartupModule()
{
	// This code will execute after your module is loaded into memory;
	// the exact timing is specified in the .uplugin file per-module

	/* Adding our shader directory relative to NoodleRenderingRuntime */
	AddShaderSourceDirectoryMapping("/MyRuntimePlugin", GetShaderDirectory());
}

FString FMyCoolRuntimeModule::GetShaderDirectory() const
{
	return FPaths::Combine(IPluginManager::Get().FindPlugin(TEXT("MyRuntimePlugin"))->GetBaseDir(), TEXT("Shaders"));
}
```

In order to compile correctly, we must include ``ShaderCore`` as a linked library, add this to your in your 
``MyRuntimePlugin.build.cs`` of your plugin:

{{% notice style="info primary" title="4.21 and under" icon="cat" %}}
For older versions of Unreal, the include here is ``RenderCore`` anything above is ``ShaderCore``.
{{% /notice %}}

```csharp

		PrivateDependencyModuleNames.AddRange(
			new string[]
			{
				"CoreUObject",
				"Engine",
				"Slate",
				"SlateCore",
                "ShaderCore" // Add this
				// ... add private dependencies that you statically link with here ...	
			}
			);

```

## Material Custom Node

The Material Custom Node does allow for includes, but unfortunately is somewhat finicky to use with this setup. 

As far as I can tell, outside of creating your own node to handle compilation and overriding something like the 
``OnPostEditChange`` it does require that you either detach the node, reattach and hit Apply in the Material Editor. 
Or just make a change to the code specified within the custom node text box itself.

Here is the code to use within the custom node, as you can see, this is where the virtual path comes in:

```hlsl
#include "/MyRuntimePlugin/Common.usf";
return 1;
```



---

### Additional Resources
> [1] [Using Compute Shaders in Unreal Engine 4](https://medium.com/realities-io/using-compute-shaders-in-unreal-engine-4-f64bac65a907) medium.com/realities-io  
> [2] [Virtual Shader Source Path - Link custom Shaders - Shadertoy Demo download](https://forums.unrealengine.com/t/virtual-shader-source-path-link-custom-shaders-shadertoy-demo-download/119996) forums.unrealengine.com    
> [3] [UnrealEngine HLSL Shader Linking](https://github.com/MaximeDup/UnrealEngine_ShaderLinking/tree/master) github.com/MaximeDup       