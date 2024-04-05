+++
title = 'Scene View Extensions'
date = 2024-03-21T15:37:22+01:00
draft = false
weight = 99
ShowNotice = true
Notice = "Work in Progress"
Author = "Alessa \"Codekitten\" Baker"
Author_url = "/Authors/alessa-codekitten-baker"
status = "WIP"
UnrealVersion = "5.2"
+++

{{< article-header >}}

---

## What are Scene View Extensions?

Scene View Extensions (SVEs) are in essence a relatively simple way to inject into the render pipeline your own shader 
(Pixel, Vertex or Compute Shader) from your own HLSL to be compiled on the Render Dependency Graph. 

Since ``ISceneViewExtension`` is extensible, it allows for this to be called within a plugin making it one of the easier 
ways to add to the render pipeline without requiring Engine Modification.

Unfortunately, SVEs aren't documented by Unreal Engine, however the actual header file ``SceneViewExtension.h`` is 
surprisingly one of the better documented parts of the engine.

There aren't many hook points available, but here is a list of the ones that are mainly accessible [3].

{{% notice style="info warning" title="Note" icon="cat" %}}
Unfortunately as of Unreal Engine 5.1 some of these have calls have been deprecated, see their **UE_DEPRECATED** macro in the
header file for more information.
{{% /notice %}}

| Pass                              | Description                                                                                                                                      |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| SetupViewFamily                   | Called on game thread when creating the view family.                                                                                             |
| SetupView                         | Called on game thread when creating the view.                                                                                                    |
| SetupViewPoint                    | Called when creating the viewpoint, before culling, in case an external tracking device needs to modify the base location of the view            |
| SetupViewProjectionMatrix         | Called when creating the view, in case non-stereo devices need to update projection matrix.                                                      |
| BeginRenderViewFamily             | Called on game thread when view family is about to be rendered.                                                                                  |
| PreRenderViewFamily_RenderThread  | Called on render thread at the start of rendering.                                                                                               |
| PreRenderView_RenderThread        | Called on render thread at the start of rendering, for each view, after PreRenderViewFamily_RenderThread call.                                   |
| PostRenderBasePass_RenderThread   | Called right after Base Pass rendering finished                                                                                                  |
| PrePostProcessPass_RenderThread   | Called right before Post Processing rendering begins                                                                                             |
| SubscribeToPostProcessingPass     | This will be called at the beginning of post processing to make sure that each view extension gets a chance to subscribe to an after pass event. |
| PostRenderViewFamily_RenderThread | Allows to render content after the 3D content scene, useful for debugging                                                                        |
| PostRenderView_RenderThread       | Allows to render content after the 3D content scene, useful for debugging                                                                        |

Overriding any of these allows for you ISceneViewExtensionBase to call on your new pass and have it running on the render
stack.

Any of the ones with _RenderThread give access to their own ```FRDGBuilder``` which is the starting point for building
a graph of passes to be rendered with the Render Dependency Graph.

## How to create a SVE

Before we get started please check out the article on [Custom .Ush files]() as this will get you started on generating
your own virtual shader path to be able to store your own shaders relative to your plugin directory.

The technique in the article above also is just a super helpful way to run custom HLSL in an IDE and not have to rely
on writing code in that horrible tiny text box on the details panel of the Custom Node 
(this appears to be changing with a correctly formatted box in UE5.4+ on UE5-main 👀❤️).


---

## Additional Resources
> [1] [Global shaders in Unreal without engine modification by Caius](https://itscai.us/blog/post/ue-view-extensions/) itscai.us    
> [2] [SceneViewExtensionTest by A57R4L](https://github.com/A57R4L/SceneViewExtTest) github.com  
> [3] [SceneViewExtension.h](https://github.com/EpicGames/UnrealEngine/blob/d94b38ae3446da52224bedd2568c078f828b4039/Engine/Source/Runtime/Engine/Public/SceneViewExtension.h) github.com  