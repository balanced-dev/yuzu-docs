# Schema creation

Using the viewmodel definition to generate all the document types and datatypes required to store the content for the specific UI. Yuzu Import uses the standard Umbraco API's to do this, it's no different from manually creating document types and data types. It's just far quicker and, as a bonus, the result is standardised. 

## How does it work

Starting with a viewmodel, the schema creation engine finds / creates a document types with the same name and iterates through the viewmodel properties adding new properties to the document type. 

If a property references another viewmodel it runs the process above for the child viewmodel. It also generated a Nested Content instance for this child viewmodel and uses it as the property types in the parent document type. 

Whilst working through the tree, each document type and property is only created once. We use this to add anything new without affecting the existing document type structure. 

This process can be run from any point in the viewmodel structure. Our favorite button is the Map All Viewmodels which generates all viewmodels. 

## Standardisation

Every type of element is always added in exactly the same way. Inline child property types such as Nested Content has given us great possibilities but is also open to different interpretation by different developers and over time. By removing this human element we have found that we have improved the quality of our Umbraco sites by standardising how everything is created.

## Mappings

Json schema property types are mapped to the following Umbraco data types

| ViewModel DataType	    	| Umbraco DataType                 |
| ----------------------------- |----------------------------------|
| string    		            | Textstring                       |
| integer / double   		    | Number                           |
| boolean    		            | True / False                     |
| string array   		        | Umbraco.MultipleTextstring       |
| enum             		        | Umbraco.DropDown.Flexible        |
| object (inline / ref)         | Umbraco.NestedContent (single)   |
| object array (inline / ref)   | Umbraco.NestedContent (multiple) |

For object and object array sub blocks, when content is stored globally the Umbraco data type is changed to a  Umbraco.MultiNodeTreePicker.

Yuzu

| Data structures   	    	| Umbraco DataType            |
| ----------------------------- |-----------------------------|
| vmBlock_DataImage    		    | Media Picker                |
| vmBlock_DataLink (single)     | Single URL Picker           |
| vmBlock_DataLink (array)      | Multi URL Picker            |
| vmBlock_DataGridRows	        | Umbraco.Grid                |

### Grid 

When grid schemas contain Config settings at row or column level we convert these to the config json file on the Umbraco grid property type. 

We use Our.Umbraco.DocTypeGridEditor to define the blocks that are allowing in the grid adding a new editor type to the grid_editors.config file for each grid in the schema.

## Extending behaviour

As part of our roadmap this year we want to add the ability for developers to swap out or add their own schema mappers. We want companies to be able to define and standardise how they create Document Types and Data Types to their style and needs.

All objects in Yuzu Umbraco Import have been added to the Umbraco IOC controller and can be swapped out for your own implementation.


