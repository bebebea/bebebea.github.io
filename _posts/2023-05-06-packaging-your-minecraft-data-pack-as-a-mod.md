---
layout: post
title: How to package your Minecraft data pack as a mod
tags: Games Minecraft
---

If you've ever created a data pack at some point you're going to ask how you can use it as a mod. And, oh, have I been asked that quite a few times.
  
In this post I want to give you a quick tutorial on how to do that in the most straightforward way possible! Without having to compile a mod and stuff.

### JAR vs ZIP

You probably already know that mods are distributed as JAR files.
And as a data pack creator all you need to know is that [JAR files]((https://en.wikipedia.org/wiki/JAR_(file_format))) are kinda like ZIP archives without any compression.
Obviously it's a little more complex than this, but this is all that matters to us.
  
This effectively means that the process of packaging your data pack as a mod and zipping up your data pack are both essentially the same. You just need to add a few more files to make your data pack be read as a mod by mod loaders.

### Fabric

Fabric requires you to have a [*fabric.mod.json* file](https://fabricmc.net/wiki/documentation:fabric_mod_json) in your root directory (the same place where you have your *pack.mcmeta* file). This file contains metadata about your mod.
  
Here is a basic template.
  
{% highlight json %}
{
  "schemaVersion": 1,
  "version": "1.0.0",
  "id": "mod_id",
  "name": "My Mod",
  "description": "",
  "authors": [
    "Me",
    "My Cat"
  ],
  "contributors": [],
  "contact": {
    "homepage": "",
    "sources": "",
    "issues": ""
  },
  "license": "Probably MIT",
  "icon": "pack.png",
  "environment": "*",
  "depends": {}
}
{% endhighlight %}

### Forge

Forge requires you to have a [*mods.toml* file](https://docs.minecraftforge.net/en/1.19.2/gettingstarted/structuring/) in the *META-INF* directory which is located in your root directory.
  
A template is available in the Forge docs which I linked above. However, instead of using the typical *javafml* mod loader, you'll need to use *lowcodefml*.
This will require you to format the file slightly differently. I don't know why.
Still, here is an example of how it should (has to) look. Unfortunately, I wasn't able to find any documentation about it online.
  
{% highlight js %}
modLoader = 'lowcodefml'
loaderVersion = '[41,45)'
license = 'MIT License'
showAsResourcePack = false

mods = [
	{ modId = 'mod_id', version = '1.0.0', displayName = 'My Mod', description = '', logoFile = '', credits = '', authors = 'My Cat', displayURL = '' },
]
[[dependencies.mod_id]]
    modId="dependency"
    mandatory=true
    versionRange="[1.0.0,]"
    ordering="NONE"
    side="BOTH"
{% endhighlight %}

### Final step

Remember how I've said that a JAR file is like a ZIP archive without compression? That's a useful bit of info we're going to make use of now!
  
Make sure you have a file archiver installed that supports choosing the compression level, such as [7-Zip](https://www.7-zip.org/).
  
When creating the ZIP file, set the compression level to zero. This will only store the files without compressing them.
  
Finally, change the file extension to *.jar*. 
  
That's it!
