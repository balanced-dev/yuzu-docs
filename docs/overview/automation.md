# Automation

When viewed at a distance the schema that defines the properties for each block and the relationships between each block starts to take the form of a map of the dynamicity of the site.

Because the core to Yuzu is a managed interface we can use this definition to automate the generation of different areas of the site to accellerate production. 

Currently we have two areas of automation

- yuzu-defintion-import - we have created a bepoke short-hand schema language that makes it easy to define what the schema for a block is. This text fragment can be stored as files or as Trello cards.  
This application takes these text fragment and renders it into a schema. From here the application generates
    - initial empty json state file
    - scss file with bem stubs for all the properties
    - template with mocks up HTML for all the properties
    - schema file