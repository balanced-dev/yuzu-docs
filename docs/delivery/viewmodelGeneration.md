# Viewmodel Generation

This is the process of converting the schema output from definition into c# pocos. We have developed a Visual Studio Extension to make this task as easy as possible. 

It borrows heavily from the structures and code defined in https://github.com/zpqrtbnk/Zbu.ModelsBuilder. The product wouldn't be possible without the hard work that already went into that solution and the learning that I gained from understanding the source code. Thank you Stephan Gay at https://www.zpqrtbnk.net/

Once installed the extension need to be configured. It connects to the web application to discover details about the current website; document types definitions etc. In Visual Studio go to the Tools > Options > Yuzu and then add in the URL, username and password for the CMS. 

Setup of the Viewmodel generation again is very similar to the way that Models Builder works for Umbraco in API mode. 

- create a new c# file where you want to generate the viewmodels
- right click on this file and select properties
- add the text YuzuViewModelGenerator as the Custom Tool

This should start the process of generating the Viewmodels from the current definition release. 