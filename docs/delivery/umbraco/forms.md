# Forms

A collection of data structures, services and mapping helpers that make it easy to define forms in a pattern library and create standalone forms or integrate then into Umbraco Forms. 

By completely separating form element UI definitions, treating each form element as a separate block, forms become much easier to understand, maintain and control.

Integration into Umbraco using a standardized package allows developers to automatically integrate with Umbraco Forms writing no code. For anything bespoke we have added extensions points to append or replace logic that maps Umbraco Forms data to ViewModels. 

## Data Structures

There are two data structures for forms, vmBlock_DataForm and vmBlock_DataFormBuilder. 

- `vmBlock_DataForm` : a necessary switch between rendering the form in the pattern library and production site. .Net needs to generate it's own form tag, adding hidden form elements to prevent cross site scripting attacks. On the production site a prerendered form is added as the LiveForm property.
- `vmBlock_DataFormBuilder` : the actual form structure.

## Form Field Types

Definition includes default viewmodels for the following form elements. These mirror the field types available in Umbraco Forms.

- Button
- Radio / Checkbox
- Radio / Checkbox Lists
- Hidden
- Dropdownlists (Select)
- Input (text, date, url, email)
- TextArea
- Recaptcha 2
- Title and Description

## Extending Field Types

Each form type viewmodel includes a special property named Config with a property type of object. We use this to hold properties that are non standard, because it's an object it will accept any number of properties, including other Yuzu blocks and anonymous types.

``` c#

    new vmBlock_FormTextInput()
    {
        Type = type,
        Name = model.Id,
        Label = model.Caption,
        Value = model.Value
        Config = new {
            Width = "50%"
        },
        _ref = "parFormTextInput"
    };
}

```

## Umbraco Forms

Yuzu field element viewmodels are mapped and combined to create the markup for Umbraco Forms, because there is no direct translation between the the Umbraco FieldViewModel and the Yuzu viewmodel we have created manual mappings for each type. These mapping files are used when the form is rendered (using the form builder) looping over each the form fields (and any fieldsets) in the Umbraco form and rendering the correct Yuzu form block. 

#### Replacing the default manual mapping

To override default mapping, add a new class that implements the interface `IFormFieldMappings` and set the IsValid check to evaluate matching the Umbraco Form Field type name. Add the bespoke code in the Apply method. 

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

?> In manual mappings like this its important to send the _ref property, this will define which block is rendered.

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

## Umbraco Form Partial 

A bespoke partial (Partials/Forms/YuzuUmbracoForm.cs) is used to render the Umbraco Form, it handles, rendering form fields, return state changes, validation summaries and form completion states. 

This form partial dynamically selected the Yuzu template that is used to render the form. In the normal flow of Yuzu the ofType added in the definition defines what form template is used, this gives definition to control which form templates at used where. 

But when the form is rendered on its own then the template must be passed in the AutoMapper options, as seen below.

``` c#

@(Html.RenderYuzu<YuzuDelivery.Umbraco.Forms.vmBlock_DataForm>(template.Contact.Form, false,
new Dictionary<string, object>() { 
    { "FormBuilderTemplate", "parFormBuilder" } 
}))

```

## Standalone Forms

Our experience is that, for standalone forms, working with the form build viewmodel (`vmBlock_FormBuider`) directly quickly becomes too verbose. We have created a FormBuilder factory and a series of extension methods to make this viewmodel easier to work with to create more efficient code.   

### FormBuilder Factory

To generate a form use the Form method

``` c#

var formFields = new List<object>()
{
    new vmBlock_FormTextInput() { Name = "FullName", Label = "Full Name", Type = "text", IsRequired = true },
    new vmBlock_FormTextInput() { Name = "EmailAddress", Label = "Email Address", Type = "email", IsRequired = true },
    new vmBlock_FormTextArea() { Name = "EmailAddress", Label = "Email Address", IsRequired = true }
}

var formBuilder = formBuilderFactory.Form("Send a Message", "Send", formFields)

```

To create a success message for the form use the Success method 

``` c#

var formBuilder = formBuilderFactory.Success("Message Sent", "Thanks, we'll be in touch");

```

To add a single form element use the AddFormField extension method

``` c#

var textfield = new vmBlock_FormTextInput() { Name = "FullName", Label = "Full Name", Type = "text", IsRequired = true };
formBuilder.AddFormField(textfield);

```

Each of these method calls return a `vmBlock_FormBuilder` viewmodel that describes the structure of the form on the page. On its own this won't be enough to render the form, we need to describe how the form is generated; the controller and action that handle the form response and everything else needed in the form tag and it's supporting elements. For this we use a `YuzuFormViewModel` class.

### YuzuForm ViewModel

This class has a property for the `vmBlock_FormBuilder` viewmodel called `Form` and we use an extension method called `AddHandler` to link the controller action response handler. 

``` c#

var model = new YuzuFormViewModel();

var formFields = new List<object>()
{
    new vmBlock_FormTextInput() { Name = "Password", Type = "password", Label = "Password", IsRequired = true },
    new vmBlock_FormTextInput() { Name = "ConfirmPassword", Type = "password", Label = "Confirm Password", IsRequired = true },
};
model.Form = viewmodelFactory.Form("Change Password", "Change Password", formFields);
model.AddHandler<MemberHandlerController>(x => x.HandleChangePassword(null));

return PartialView("YuzuForms", model);

```

Above is a controller action for a change password form, it renders a partial view called `YuzuForms`. This partial view is installed as part of the Forms package and is a **centralized standalone form renderer**. 

We are defining the form structure in the controller action in the viewmodel, and the partial view uses this model to render a Yuzu template (by default `parFormBuilder`) from the pattern library. Then there is no need for the partial to be different for each of the forms on the site! 

?> This is complete separation between definition and delivery in action; simple, clean, pure and efficient code.

Taking it a step further we can create a complete form using another method on the FormBuilder Factory

``` c#

var model = viewmodelFactory.Create(this, formFields, x => x.HasTempData("FormConfirmSuccess"), x => x.HandleConfirm(null));

```

The third parameter defines when the form is in a `success` state and the forth parameter points to a response handler. 

Where is the content for this form? This method expects uses the content node (from `UmbracoContext.PublishedRequest.PublishedContent`) and looks for the following content properties

| Property Alias		    	| 
| ----------------------------- | 
| completedTitle          		|
| completedBodyText             |
| title               		    |
| submitButtonText              |
| actionLinks               	|
| completedActionLinks     	    |

When we create a site that uses forms we create a Document Type contains these content properties. We compose this with any Document Types that contain forms. 

The final piece of the puzzle is to create a manual map adding the form to the site. Below is a YuzuTypeConvertor that creates the initial state render of the form. For this we include a property on the composed Document Type of `Endpoint` that contains the Controller and Action for the form in CSV. 

``` c#

public class AccountFormConvertor : IYuzuTypeConvertor<AccountForm, vmPage_AccountFormPage>
{
    public vmPage_AccountFormPage Convert(AccountForm source, UmbracoMappingContext context)
    {
        var endpoint = source.Endpoint.ToControllerAndAction();

        return new vmPage_AccountFormPage()
        {
            Form = new vmBlock_DataForm()
            {
                LiveForm = context.Html.Action(endpoint.Action, endpoint.Controller).ToString()
            }
        };
    }
}

```

### Forms And Grids

Forms included in grid items are a special case, because grid items can't rely upon upon content from `UmbracoContext.PublishedRequest.PublishedContent`. We have created their own method on the formBuilder Factory.

``` c#

var model = viewmodelFactory.CreateForGridItem<Controller, AccountPage>(this, formFields, x => x.HasTempData("FormSuccess"), x => x.ChangeMember(), x => x.Content);

```

The fifth parameter here points to the Grid property on the document type, in this case the `AccountPage`. For us, this code is too complex so we usually create an extension method for each document type that contains a grid.

``` c#

public YuzuFormViewModel CreateForAccountPage<C>(C c, List<object> formFields, Expression<Func<C, bool>> isSuccess, Expression<Func<C, ActionResult>> getContentForEndpoint, Expression<Func<C, ActionResult>> handler = null, string template = null)
    where C : Controller
{
    return CreateForGridItem<C, AccountPage>(c, formFields, isSuccess, getContentForEndpoint, x => x.Content, handler, template);
}
```

This keeps it in line with the default property on the formBuilder factory.

``` c#

var model = viewmodelFactory.CreateForAccountPage(this, formFields, x => x.HasTempData("FormSuccess"), x => x.ChangeMember());

```

You may or may not like the way we have defined any of the above. The purpose of releasing this code is not to define the way of solving this problem. Purely to demonstrate how, using the Yuzu pattern library, we can simplify the code needed to handle these complex but common problems.