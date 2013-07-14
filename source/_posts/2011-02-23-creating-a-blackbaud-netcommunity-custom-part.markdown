---
layout: post
title: "Creating a Blackbaud NetCommunity Custom Part"
date: 2011-02-23 18:32
comments: true
categories: 
- ASP.Net
- C#
- Web
- Blackbaud NetCommunity
---
I've recently been given the task of creating some custom plugins for NetCommunity, so we can do things like unify the design of our Faculty Biography pages at Ambrose. I have prior experience with C#, so I figured it would be a fairly simple process, once I figured out how their API works, however there is minimal documentation, and all of the tutorials online are either outdated or incomplete.

As such, I am going to write up a concise walkthrough for creating a NetCommunity plugin (for 6.15) using C#, from scratch, up until you have a functional part.

Begin by creating a new "ASP.NET Empty Web Application" project in Microsoft Visual Web Developer (<http://cl.ly/4nce>). Right click on References, in the Solution Explorer, and choose Add Reference. Under the Browse tab, navigate to the Bin folder of your NetCommunity installation, and choose `BBNCExtensions.dll` (<http://cl.ly/4nOt>). Finally, go into the Project menu, and click `{ProjectName} properties`, and change Target framework to `.NET Framework 3.5` (under the application tab). You're now ready to start coding some basic functionality into your application.

Right click on your project in the Solution Explorer (should be the bold title at the very top), and choose `Add->New Item`. You will want to create two new `Web User Controls` named `Display.ascx` and `Editor.ascx` (<http://cl.ly/4n69>). In Display.ascx.cs and Editor.ascx.cs, add the following "using" statements:

``` c# Display.ascx.cs and Editor.ascx.cs
using BBNCExtensions;
using BBNCExtensions.Parts;
```

Then, in `Display.ascx.cs`, change `System.WebUI.UserControl` (the class to inherit from) to `BBNCExtensions.Parts.CustomPartDisplayBase`. In `Editor.ascx.cs`, you'll want to change `System.WebUI.UserControl` to `BBNCExtensions.Parts.CustomPartEditorBase`. You'll also want to add two functions, as follow:

``` c# Display.ascx.cs
public override void OnLoadContent() {

}

public override bool OnSaveContent(bool bDialogIsClosing){
    return true;
}
```

Your plugin is now finished, and it is time to build it, and set it up in NetCommunity. Click build, and copy `bin/{ProjectName}.dll` to `{NetCommunityInstall}/Bin`, and copy `Display.ascx` and `Editor.ascx` (from the project root) to `{NetCommunityInstall}/Custom/{ProjectName}/`. Then open up the Custom Part settings page in NetCommunity, click `New custom part`, and give your part a name. For Display/Edit Control Source, enter `~\Custom\{ProjectName}\{Display|Editor}.ascx` (depending on which field you're changing), and leave `Supports Personalization for` set to `Not Supported`.

Save the custom part, and try creating a part, and you should see a blank editor page, and a blank display page (after you save it). Your NetCommunity plugin is now built, so give yourself a pat on the back, take a quick break, and then we'll add some basic functionality.

Every change you make will require that you update either the DLL, the ascx files, or both (depending on what you change), so lets automate that with some build events. Click the Project Menu, and then `{ProjectName} Properties`. Open the Build Events tab, and edit the Post-build event so that it reads (with obvious path adjustments):

``` plain 
copy "$(ProjectDir)*.ascx"  "c:\Program Files\Blackbaud\NetCommunity\Custom\$(ProjectName)"
copy "$(TargetDir)$(ProjectName).dll" "c:\Program Files\Blackbaud\NetCommunity\Bin"
```

Click build, and you should see it automatically copy the three files into their appropriate directories.

Now to bring the plugin to life, and add some actual functionality. Right click on your project in the Solution Explorer, and choose `Add->Class`. Name it something appropriate, as this class will hold the settings for this part. Inside of the class declaration, add a public variable for every property of the part (e.g. public string name = "";).

Open `Editor.ascx` in design mode, and add a text field (you can add labels and formatting whenever you please) for each property that you want to make editable, and set the ID to match the property name. I chose to prefix each with "field" to make it easier to find, which resulted in things like "fieldEducation" as an ID, but as long as you can keep track of each property, you can name it any proper variable name.

Then, go back to `Editor.ascx.cs`, and in your `OnSaveContent` function, add the following (adjusting as necessary). I'm going to use property_class, but replace that with whatever you named the class you created above (containing your property declarations):

``` c# Editor.ascx.cs - OnSaveContent
// Lets create an empty property_class variable, to save our data
property_class save_data = new property_class();
// For each property that you have, add a line similar to this
save_data.education = fieldEducation.Text;
// Now lets save all of those properties to the part
base.Content.SaveContent(save_data);
// Finally, let the browser know that everything worked, and let the window close
// When you add validation, return false if you find an error
return true;
```

While still in `Editor.ascx.cs`, add the following to `OnLoadContent`, again modifying it as needed:

``` c# Editor.ascx.cs - OnLoadContent
// Check if we're not updating the data (e.g. we just opened the editor)
if (Request.RequestType != "POST")
{
    // Load the content from our part, as property_class
    property_class get_data = base.Content.GetContent(typeof(property_class)) as property_class;
    // If get_data is null, this is a brand new part. If it isn't null, lets set our fields to the data
    if (get_data != null)
    {
        // Add a line like this for every property in property_class
        fieldEducation.Text = get_data.education;
    }
}
```

If you build the part, and try it in NetCommunity at this point, you'll find that you can create and edit the part, and it will retain your changes, however it doesn't display your changes anywhere. To add that in, open up `Display.ascx` in design mode, and add Literal tags to your code, again, setting the ID to something logical. Then open up `Display.ascx.cs`, and add the following to your `Page_Load` function, as usual, editing it where needed:

``` c# Display.ascx - Page_Load
// Load the content from our part, as property_class
property_class get_data = base.Content.GetContent(typeof(property_class)) as property_class;
// Has this part been saved before?
if (get_data != null)
{
    // Add a line like this for every property in property_class
    fieldEducation.Text = get_data.education;
}
```

If you save and build your part again, you'll see that your properties are now being displayed as well. Congratulations on successfully building your first Blackbaud NetCommunity Custom Part.
