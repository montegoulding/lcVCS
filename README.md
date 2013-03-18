#lcVCS

Stackfile export/import for VCS support in LiveCode

lcVCS exports and imports stack files to a structured folder of json, script and image files.

##Author

Monte Goulding - monte@goulding.ws
http://goulding.ws
http://mergext.com

Folder structure

Each object is given a UUID during the export. The root folder is saved next to the original stack file with an additional .vcs extension. Inside that folder the stackfile structure is replicated with a cards, sharedGroups and substacks directory. Each object apart from the mainstack is inside a directory named with it's UUID. The mainstack properties are just inside the .vcs directory. All objects on a card have a directory inside the card's directory following the stack structure by nesting objects inside group directories. In each folder is a properties.json file containing the class, uuid, properties and custom properties of the object. The properties are filtered against the default properties for the corresponding object class to reduce file size and make it easier to see differences. Addintionally an properties that are redundant are also removed, for example, the rect of a group with `lockLocation` set to false. The script of the object (if there is one) is saved as a separate file script.utf8 file. If the object is an image then the image is saved as a file named image with an extension of the paintCompression of the image object.

All data with the exception of custom properties is exported as utf8 so that it's possible for cross platform developers to work on the files without encoding issues creating conflicts. Any custom property that contains non ASCII characters is base64Encoded and wrapped in `base64Decode()`.

This structure has been designed to resolve many of the issues there are with merging branches of stack files. Most of these issues relate to the fact that object id conflicts are nigh on unresolvable. The project introduces a uVersion custom property set with the object uuid and its current id as keys. When exporting if the id of the object has changed a new uuid is generated. The uVersion custom property set is then redundant and not exported. The export does not include an object id and object ids should not be used as constants to refer to objects because when the stack is regenerated the id may be different. Properties that record ids (icons, patterns, behaviors) are exported instead as UUIDs so they can be resolved correctly during the import. You may think this is crazy but just tell me what should happen when two branches create a new control and it's assigned the next available id (as LiveCode does) which therefore creates an id conflict... which one should be given the correct id? My answer... forget ids.

##Reducing false positive conflicts
Because LiveCode stackfiles retain session changes between saves it's important to clear anything that's likey to cause a false positive conflict between two versions. A good example of this is a resizable stack with a resizeStack handler that moves objects. This could cause a conflict on most of the controls on the stack. What we want to do is reset the stack to it's default state during the export. To help with this each object is dispatched a message `lcVCSExport`. lcVCS unlocks messages to dispatch the command so you can handle the message in the stack script and resize the stack triggering your resizeStack handler. Because the message is sent to every object the objects in the lower parts of the message heirarchy will recieve it multiple times. In any control that has a child control you will want to script your lcVCSExport handler like this:

	on lcVCSExport
		if the target is not me then exit lcVCSExport
	end lcVCSExport

##Dependencies
This project depends on [mergJSON](https://github.com/montegoulding/mergJSON) which is a dual licensed (GPL/Commercial) JSON external for implementing JSONToArray and ArrayToJSON.
