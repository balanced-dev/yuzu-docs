# Forms

A collection of data structures and services and mapping helpers that make it easy to define forms in a pattern library and automatically integrate then into Umbraco Forms and also Umbraco. 

By completely separating form element UI definitions, treating each form element as a separate block, forms become much easier to understand, maintain and control.

Integration into Umbraco using a standardised package allows developers to automatically integrate with Umbraco Forms writing no code. For anything bespoke we have added extensions points to append or replace logic that maps Umbraco Forms data to ViewModels.   

## Config

| Property    			    	| Purpose 			                                                    |
| ----------------------------- | ----------------------------------------------------------------------|
| **FormElementAssemblies**		| Assemblies that contaion bespoke field types mappers or preprocessors |


## Umbraco Import Config

Ignore the form, form builder and all form elements from IgnoreViewmodels and ExcludeViewmodelsAtGeneration.

Also globally ignores all properties named Form and FormElement.

## Mapping

There are two data structures for forms, vmBlock_DataForm and vmBlock_DataFormBuilder. 

- `vmBlock_DataForm` : a necessary switch between rendering the form in the pattern library and production site. .Net needs to generate it's own form, adding hidden form elements to prevent cross site scripting attacks. On the production site a prerendered form is added as the LiveForm property.
- `vmBlock_DataFormBuilder` : the actual form structure.

We have created some mapping wrapper classes that make it easier to work with forms in Yuzu. Which one is used depends on how the other properties in the same Viewmodel are mapped.

### Form

When only the form mapping is needed for a viewmodel the command is very simple

```c#
cfg.AddGridWithRows<Contact, vmBlock_Contact>(src => src.Form, dest => dest.Form);
```

When other viewmodel properties require manual mapping then it's a little more complex.

```c#
CreateMap<Contact, vmBlock_Contact>()
    .ForMember(x => x.Form, opt => opt.MapFrom<FormMemberValueResolver<Contact, vmBlock_Contact>(y => y.Form));
```

## Form Field Types

Definition includes default viewmodels for the following form elements that mirror the field types use in Umbraco Forms.

- Button
- Radio / Checkbox
- Radio / Checkbox Lists
- Hidden
- Dropdownlists (Select)
- Input (text, date, url, email)
- TextArea
- Recaptcha 2
- Title and Descrition

Because these form field types are available globally within the pattern library their use is not  limited to Umbraco Forms, they can be used anywhere within Umbraco. For example, we use these same form field definitions in our Members package. 

The Yuzu viewmodels are mapped and combined to create the markup for Umbraco Forms but because there is no direct translation between the two sets of data we have created a manual mapping file for each Umbraco Form Field types. These mapping files are used when the form is rendered (in the form builder) looping over each the form fields (and fieldsets) in the Umbraco form and rendering the correct Yuzu form block.

## Extending Field Types

Each form type viewmodel includes a special property named Config with a property type of object. We use this to hold properties that are non standard, because it's an object it will accept any type, including Yuzu viewmodels and anonymous types.

We have added two ways to change Form Field mapping;

#### Replacing the default manual

To override default mapping add a new class that implements the interface `IFormFieldMappings` and setting the IsValid check to match the Umbraco Form Field type name. Add the bespoke code in the Apply method. 

``` c#

public class BespokeShortAnswerFieldMapping : IFormFieldMappings
{

    public bool IsValid(string name)
    {
        return name == "Short answer";
    }

    public object Apply(FieldViewModel model)
    {
        return new vmBlock_FormTextInput()
        {
            Type = type,
            Name = model.Id,
            Label = model.Caption,
            Value = model.Values.Select(x => x.ToString()).FirstOrDefault(),
            Placeholder = model.PlaceholderText,
            IsRequired = model.Mandatory,
            RequiredMessage = model.RequiredErrorMessage,
            Pattern = model.Regex,
            PatternMessage = model.InvalidErrorMessage,
            _ref = "parFormTextInput"
        };
    }
}

```

?> In manual mappings like this its important to send the _ref property as above this will define the block rendered.

#### Adding a Postprocessor

Sometimes the best option is not to replace mappings but build on top of it. Post processors can amend a viewmodel after the original mapping.

The following code sets all Short Answer fields to required. 

``` c#

public class RequiredShortAnswersPostProcessor : IFormFieldPostProcessor
{

    public bool IsValid(string name)
    {
        return name == "Short answer";
    }

    public object Apply(FieldViewModel model, IFormFieldElementConfig viewmodel)
    {
        var textbox = viewmodel as vmBlock_FormTextInput;
        textbox.IsRequired = true;
        return textbox;
    }
}

```

Another benefit to post processors is that we can affect multiple mapping outputs in one file. 

``` c#

public class AddTooltipFieldPostProcessor : IFormFieldPostProcessor
{

    public bool IsValid(string name)
    {
        return true;
    }

    public object Apply(FieldViewModel model, IFormFieldElementConfig viewmodel)
    {
        viewmodel.Config = new {
            Tooltip = "test"
        };

        return viewmodel;
    }
}

```

This code adds a tooltip to the config object of all Field Type viewmodels.

## Form Partial 

A bespoke partial (Partials/Forms/YuzuForm.cs) is used to render the Umbraco Form, it handles, rendering form fields, return state changes, validation summaries and form completion states. 

This form partial is dynamic on the Yuzu template that is used to render the form. In the normal flow of Yuzu the ofType added in the definition defines what form template is used, this gives definition to control which form templates at used where. 

But when the form is rendered on its own then the template must be passed in the AutoMapper options, as seen below.

``` c#

@(Html.RenderFETemplate<YuzuDelivery.Umbraco.Forms.vmBlock_DataForm>(new RenderSettings()
{
    Template = "parForm",
    MapFrom = template.Contact.Form,
    ShowJson = false
},
new Dictionary<string, object>() { { "HtmlHelper", Html }, { "FormBuilderTemplate", "parFormBuilder" } }))


```