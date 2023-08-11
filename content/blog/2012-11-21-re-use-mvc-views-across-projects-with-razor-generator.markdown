---
layout: post
title: "Re-Use MVC Views Across Projects With Razor Generator"
date: 2012-11-21 21:56:00 +0000
comments: true
thumbnail: /images/posts/razor/Razor-Generator-Extension_thumb.png
summary: "You may consider storing the content in a resource file and embed it in a class library. Or perhaps do a clever virtual directory mapping in your IIS setup. However, the best solution is simply to compile the views into a class library using Razor Generator."
tags: [Asp.Net, Razor, Visual Studio, C#]
archive: 2012
---

A typical MVC site would consist of a folder containing all the views and partials that are then rendered using a view engine. However, it can get a little tricky when you want to **re-use the same view template across multiple projects without content duplication**. Perhaps you want to re-use a small partial with tracking code or even entire views for common actions such as login / reporting.

You may consider storing the content in a resource file and embed it in a class library. Or perhaps do a clever virtual directory mapping in your IIS setup. However, the best solution is simply to **compile the views into a class library using Razor Generator**. Razor Generator is  tool that allows processing Razor files at design time instead of runtime, allowing them to be built into an assembly.

Install Razor Generator Extension
-------------------

<img alt="Razor-Generator-Extension" src="/images/posts/razor/Razor-Generator-Extension_thumb.png" class="post-image-right" />

In order to get your views compiled, you will need to install a tiny extension. The latest version can be found in the [Visual Studio Extensions Gallery](http://visualstudiogallery.msdn.microsoft.com/1f6ec6ff-e89b-4c47-8e79-d2d68df894ec). Once installed, you will need to re-start Visual Studio.

The extension will enable RazorGenerator Custom Tool. Custom tools in Visual Studio can be attached to various files to generate automatic classes. For example, the *ResXFileCodeGenerator* tool looks after auto-generating classes for resource files. Visual Studio comes with several tools built in, however, [you can also build your own](http://aviadezra.blogspot.co.uk/2008/11/developing-custom-tool-for-visual.html).

Create Your Shared Project
-------------------


<img alt="Razor-Generator-Common-Project" src="/images/posts/razor/Razor-Generator-Common-Project_thumb.png" class="post-image-right" />
First, you will need to **create you shared class library project**. You can use the standard class library.

<br /><br /><br /><br />

<img alt="Razor-Generator-Common-Project-Nuget " src="/images/posts/razor/Razor-Generator-Common-Project-Nuget_thumb.png" class="post-image-right" />
Next thing you need to need to do is include the **RazorGenerator.MVC** package via Nuget. This is going to bring down couple of dependencies and will enable serving of the auto-generated views once they are compiled in the dll.

Add Re-Usable Views
-------------------

Once you have your common project setup, you can create the same directory structure as you would in a normal MVC project. For example, if you wanted your re-usable partial view to be in shared, you would place it in *~/Views/Shared/YourPartial.cshtml* or even *~/Views/Shared/DisplayTemplates/YourPartial.cshtml*.

<img alt="Razor-Generator-Common-Project-View-Properties" src="/images/posts/razor/Razor-Generator-Common-Project-View-Properties_thumb.png" class="post-image-right" />
Finally, you need to make sure you specify RazorGenerator as a custom tool for the Views. Simply right-click on the views and select properties. You will a field for the Custom Tool which you **should set to RazorGenerator**.

<br /><br /><br />

<img alt="Razor-Generator-Common-Project-View-Properties2" src="/images/posts/razor/Razor-Generator-Common-Project-View-Properties2_thumb.png" class="post-image-right" />
Once you re-build the project, you should see an arrow below your views, indicating there is a linked .cs file.


Include a Reference to Your Shared Project
-------------------

Finally, you should reference the class library into your projects. All you need is a reference to the dll produced when the class library is built. You can then call up any views as if they were present in the solution. The example below assumes you have SharedPartial.cshtml in your ~/View/Shared/ folder in the Example.Common.

{{< highlight html "linenos=inline" >}}
... your view content
<div class="container">
    @Html.Partial("SharedPartial")
</div>
... your view content
{{< / highlight >}} 

Have fun! If you have any questions, give me a shout [@mirajavora](http://twitter.com/mirajavora).

Related Links
-------------------

[Razor Generator Extension for Visual Studio](http://visualstudiogallery.msdn.microsoft.com/1f6ec6ff-e89b-4c47-8e79-d2d68df894ec)<br />
[Razor Generator Project On Codeplex](http://razorgenerator.codeplex.com/)