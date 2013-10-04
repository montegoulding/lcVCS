#lcVCS

Stackfile export/import for VCS support in LiveCode

lcVCS exports and imports stack files to a structured folder of json, script and image files.

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=Z772WTBXRTLMQ)

##Author

Monte Goulding - monte@goulding.ws

M E R Goulding Software Development http://goulding.ws

mergExt LiveCode Extensions http://mergext.com

##Project File
A lcVCSProject.json file is used to order the stackFiles in a project and manage the build path of the stackFiles. It is posible to use multiple project files in a single code repository so the projects can be imported and exported independently.

##Folder structure

Each object is given a UUID during the stackFile export. A stackFile is exported to a directory with the name of the stackFile and a .vcs extension. Inside this directory the data for each object can be found at a relative path created from the first two characters of the UUID as a parent directory and the rest of the UUID as a child directory.

In each folder is a properties.json file containing the class, uuid, properties and custom properties of the object. The script of the object (if there is one) is saved as a separate file script.utf8 file. If the object is an image then the image is saved as a file named image with an extension of the paintCompression of the image object.

An example path to an object properties file is therefore:
stack.livecode.vcs/f7/8b57e1-8aed-4f01-9022-c3d47e231a2e/properties.json

lcVCS also uses empty files named with a UUID as a form of alias or reference to an object. For example, a file with the UUID of the mainstack of the stackFile can be found inside the *.vcs directory. If an object is a stack it will have a cards subdirectory and may have a substacks and sharedGroups subdirectory containing these object reference files. If an object is a card or group it's directory may also contain these object reference files informing lcVCS of child objects.

All data with the exception of custom properties is exported as utf8 so that it's possible for cross platform developers to work on the files without encoding issues creating conflicts. Any custom property that contains non ASCII characters is base64Encoded and wrapped in `base64Decode()`.

This structure has been designed to resolve many of the issues there are with merging branches of stack files. Most of these issues relate to the fact that object ID conflicts are nigh on unresolvable. The project introduces a uVersion custom property set with the object UUID and its current ID as keys. When exporting if the ID of the object has changed a new UUID is generated. The uVersion custom property set is then redundant and not exported. The export includes object IDs, however, object IDs should not be used as constants to refer to objects because when the stack is regenerated the ID may be different if there was merge of two branches that created objects. Properties that record IDs (icons, patterns, behaviors) are exported instead as UUIDs so they can be resolved correctly during the import. You may think this is crazy but just tell me what should happen when two branches create a new control and it's assigned the next available IDs (as LiveCode does) which therefore creates an id conflict... which one should be given the correct id?

A plugin system supports the export and import of object references found in custom property sets.

##Reducing false positive conflicts
Because LiveCode stackfiles retain session changes between saves it's important to clear anything that's likey to cause a false positive conflict between two versions. A good example of this is a resizable stack with a resizeStack handler that moves objects. This could cause a conflict on most of the controls on the stack. What we want to do is reset the stack to it's default state during the export. To help with this each object is dispatched a message `lcVCSExport`. lcVCS unlocks messages to dispatch the command so you can handle the message in the stack script and resize the stack triggering your resizeStack handler. Because the message is sent to every object the objects in the lower parts of the message heirarchy will recieve it multiple times. In any control that has a child control you will want to script your lcVCSExport handler like this:

	on lcVCSExport
		if the target is not me then exit lcVCSExport
	end lcVCSExport

##Rules for successful use of lcVCS
* If you use IDs in scripts anywhere (such as setting the icon of a button) then replace it with a name based reference.
* Implement lcVCSExport handlers to ensure everything is set back to defaults during the export.
* If you have any custom objects or libraries that maintain custom property sets that store IDs then implement a plugin to support it. The plugin api is quite simple and there’s a number of examples demonstrating their use. If possible contribute the plugin back to the project so we can maintain them centrally.
* Limit object references between stackFiles where possible. Where not possible ensure that there are no circular references. For example, stackFile A has an object reference to an object in stackFile B while stackFile B has an object reference to an object in stackFile A.
* Order your stackFiles so that the stackFiles that have object references to objects in other stackFiles are imported and exported after the stackFiles they refer to. The stackFiles list in the lcVCSProjects stack can be re-ordered by drag and drop.
* If you have object references to objects that no longer exist (such as icons referencing deleted images) clear the property.

##Dependencies
* [mergJSON](https://github.com/montegoulding/mergJSON) which is a dual licensed (GPL/Commercial) JSON external for implementing JSONToArray and ArrayToJSON.
* LiveCode 6.1.1
