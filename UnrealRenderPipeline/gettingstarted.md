+++
title = 'Getting Started'
date = 2024-03-21T14:48:29+01:00
draft = false
weight = -1
Notice = "Work in Progress"
Author = "Alessa \"Codekitten\" Baker"
Author_url = "/authors/alessa-codekitten-baker"
status = "WIP"
description = "This article acts as a getting started on modifying the render pipeline in Unreal, tips on debugging, compiling shaders and some helpful commands."
type="article"
+++

{{< article-header >}}

## Workflow

There are a couple of paths that Unreal Engine's render pipeline can be extended - 
[Scene View Extensions](unrealrenderpipeline/plugins/sceneviewextensions), [Virtual Shader Paths](unrealrenderpipeline/plugins/virtualshaderpaths) 
to externally write shaders and include them in material Custom Nodes, or modifying the existing ush/usf files of the 
engine as an Engine Modification.

I have spent quite a lot of time attempting to figure out the best general workflow when extending Unreal Engine's rendering pipeline, but all of them benefit from some additional parameters that can be enabled in ini files.

I should also preface all of this with a word on specs - Shader Compilation is quite an intensive process, and while I can give some ways to optimize and lower compile times, it requires a pretty hefty machine. 

For context - I currently am running a Ryzen 9 7950x, 128GB DDR5 RAM 3600MHz, an RTX 3080, and all of my storage is on NVMe SSDs - that said, I still find that this is a little overkill for most things 

I just wanted fast engine compile times. 😊

## Compiling Shaders

In any of the three mentioned paths, shaders can be compiled by running ``recompileshaders all`` as a command in the console. Recompile shaders command works by invalidating the render cache by changing a GUID stored in ``Engine/Shaders/Public/ShaderVersion.ush``. 

The All command will force shaders within the Derived Data Cache to be invalidated and recompiled and can take quite some time to recompile everything, a better command I use instead is ``recompileshaders changed`` which instead only recompiles shaders that Unreal has recognized as a modified usf/ush.

Both ush and usf files are captured with the above commands, meaning the editor doesn't need to be closed. Unreal thankfully has set up a standard here, where any global shader ending in Common.ush is used across multiple paths, so modifying these is a pretty easy indicator that shader compilation will take longer.

{{% notice style="info primary" title="Tip" icon="cat" %}}
``recompileshaders changed`` can also be called by simply pressing Ctrl+Shift+. (period) on your keyboard, this runs the same command.
{{% /notice %}}

When calling this command, should you have a compiler error with your HLSL - you will inevitably crash the engine, this is where some ini file commands can come in useful, namely, Shader Development Mode.

## Shader Development Mode

Shader Development Mode, can be enabled by editing ``ConsoleVariables.ini`` on either an Engine or Project level, in this file you will already see the command, ``r.ShaderDevelopmentMode``, is one of the first and is commented out,  sporemoving the semi colon at the beginning will enable this.

This mode when enabled provides two main features, the first is exposing more details on why a specific crash has occured by allowing Unreal to capture the specific compilation error. The second is that it puts the error in a sort of 'loop', giving you a second chance at recompiling before crashing the editor, along with displaying the HLSL error. 


