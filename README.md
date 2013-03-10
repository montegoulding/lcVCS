lcVCS
=====

Stackfile export/import for VCS support in LiveCode

lcVCS exports and imports stack files to a structured folder of json, script and image files.

Author
------

Monte Goulding - monte@goulding.ws
http://goulding.ws
http://mergext.com

Folder structure
----------------

Each object is given a UUID during the export. The root folder is saved next to the original stack file with an additional .vcs extension. Inside that folder there is an index.json file identifying the mainstack and substacks of the stackfile. Each stack has a corresponding folder named with its UUID. Inside the stack folder is another index.json file identifing the object heirarchy in the stack. Each object has a folder named with its UUID inside the stack folder. In the folder is a properties.json file containing the class, properties and custom properties of the object. The properties are filtered against the default properties for the corresponding object class to reduce file size. The script of the object (if there is one) is saved as a separate file script.utf8 file. If the object is an image then the image is saved as a file named image with an extension of the paintCompression of the image object.

This structure has been designed to resolve many of the issues there are with merging branches of stack files. Most of these issues relate to the fact that object id conflicts are nigh on unresolvable. The project introduces a uVersion custom property set with the object uuid and its current id as keys. When exporting if the id of the object has changed a new uuid is generated. The uVersion custom property set is then redundant and not exported. The export does not include an object id and object ids should not be used as constants to refer to objects because when the stack is regenerated the id may be different. Properties that record ids (icons, patterns, behaviors) are exported instead as UUIDs so they can be resolved correctly during the import. You may think this is crazy but just tell me what should happen when two branches create a new control with and it's assigned the next available id (as LiveCode does) which therefore creates an id conflict... which one should be given the correct id? My answer... forget ids.

Dependencies
------------
This project depends on mergJSON which is a dual licensed (GPL/Commercial) JSON external for implementing JSONToArray and ArrayToJSON.
