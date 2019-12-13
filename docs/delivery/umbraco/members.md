# Membership

Simple membership solutions are hard to get right, there are so many moving parts and getting it wrong could least to a security hole or UX issues. We intend to solve this problem by creating a simple, adaptable solution for adding membership funationality to Yuzu enabled Umbraco sites. Defintion have complete control of the UI and delivery can change the content of the viewmodels prerender and can define when and where forms are rendered.

This  is not intended to be the solution to creating membership forms on all sites, but as a quick solution for smaller sites that don't need overly complex solutions. It also gives an example of how we work with Yuzu, by separating out the UI and CMS integration we can create packages that are very adaptable.

The following forms are included

- register
- login
- password reminder
- change member
- change password

For form fields we use the standard Viewmodels using in Yuzu forms. 

# Viewmodel

All members forms use the same Viewmodel, `vmBlock_AccountForm`, it contains all the elements common to any form; form fields, error messages, submitButtonText etc and includes a some that we have added specifically; actionLinks (navigation through the forms). 

By using the same Viewmodel we can standardise membership forms through the site, becoming eaiser to maintain and change in the future.

# Umbraco Membership Integration

Umbraco itself comes with some basic membership integration in the form of controllers. These handle login, logout, register and profile changes. We have used the login and register controllers here. 

Using Yuzu membership doesn't preclude for all the other tricks that Umbraco can perform out if the box. For example, need to send an email on user registration, use a member service event.

# Configuration

An interface to change the membership controller settings, they standardise properties whilst making the contoller as adapatble without needing a new controller. 

| Property    			    	| Purpose 			        |Default 			        |
| ----------------------------- | --------------------------|---------------------------|
| **HomeUrl**		            |                           |                           |
| **ForgottenPasswordUrl**      |                           |                           |
| **LoginUrl**		            |                           |                           |
| **RegisterUrl**		        |                           |                           |
| **ChangePasswordUrl**		    |                           |                           |
| ForgottenPasswordLabel        | Button Label              | Forgotten Password?       |
| RegisterLabel                 | Button Label              | Register                  |
| BackLabel                     | Button Label              | Back                      |
| CancelLabel                   | Button Label              | Cancel                    |
| EmailNotFoundErrorMessage     |                           | Email address not found   |
| MemberNotFoundErrorMessage    |                           | Member not found          |
| ForgottenPasswordEmailAction  |                           |                           |

The `ForgottenPasswordEmailAction` is used for you to add your own code to the send the forgotten password link. It is passed the email address, name and a link to change the password.

But these settings can only go so far, outside of the controller we can change the contents of the Viewmodel before its rendered in the partial view. Here is an example of a register form that overrides the Success Message.

```c# 
@inherits Umbraco.Web.Mvc.UmbracoViewPage<vmBlock_AccountForm>

@using System.Web.Mvc.Html
@using Umbraco.Web
@using Umbraco.Web.Controllers  
@{
    if (!Umbraco.MemberIsLoggedOn())
    {
        Func<object> vm = () =>
        {
            if (Html.ValidationSummary() != null)
            {
                Model.Errors = Html.ValidationSummary().ToString();
            }

            if (Model.IsSubmitted)
            {
                Model.SuccessMessage = "Account created successfully. Your request will be reviewed by MRC staff.";
            }

            return Model;
        };

        using (Html.BeginUmbracoForm<UmbRegisterController>("HandleRegisterMember", FormMethod.Post, new Dictionary<string, object>() { { "novalidate", "" } }))
        {
            @Html.RenderFETemplate(new RenderSettings() { Template = "parAccountForm", Data = vm })

            <input type="hidden" name="registerModel.UsernameIsEmail" value="true" />
            <input type="hidden" name="registerModel.LoginOnSuccess" value="false" />
        }
    }
}
```

For forms actions that are more complex it's easy to create a new controller using the same patterns we use to implement your own controller. The viewmodel we have used should be valid for most instances we have com across, but if not its fine to create your own!

## Forgotten / Change Password

Puts into practice a common membership pattern for forgotten passwords. A user enters an email address that then send a link for them to change their password. The link contained in the email expires after 3 hours. 

This requires a new property with alias `forgottenPasswordExpiry` and data type of Date with time is added to member types.

## Implementation

Initialising the config for Yuzu Membership should happen at Startup and it can be tricky to get access to the content then. The code below shows a way that we have found using an Umbraco IComponent.

``` c#
public class YuzuMemberSetupComponent : IComponent
{
    protected EmailService emailService;
    private readonly IUmbracoContextFactory _umbracoContextFactory;

    public YuzuMemberSetupComponent(EmailService emailService, IUmbracoContextFactory umbracoContextFactory)
    {
        _umbracoContextFactory = umbracoContextFactory;
        this.emailService = emailService;
    }

    public void Initialize()
    {
        using (var reference = _umbracoContextFactory.EnsureUmbracoContext())
        {
            var root = reference.UmbracoContext.ContentCache.GetAtRoot().FirstOrDefault();

            var forgottenPasswordNode = root.Descendant<ForgotttenPassword>();
            var registerNode = root.Descendant<Register>();
            var changePasswordNode = root.Descendant<ChangePassword>();

            YuzuDeliveryMembers.Initialize(new YuzuDeliveryMembersConfiguration()
            {
                HomeUrl = accountHome.Url,
                ForgottenPasswordUrl = forgottenPasswordNode.Url,
                RegisterUrl = registerNode.Url,
                ChangePasswordUrl = changePasswordNode.Url,
                ForgottenPasswordEmailAction = (string email, string name, string changePasswordLink) => {
                    emailService.SendEmail(
                        email,
                        "Change Password Link",
                        string.Format(@"
                        <p>Hi {0}</p>
                        <p>
                            Here is a link to our site to change your password 
                            <a href='{1}'>click here<a/></p><p>Note, this link is only valid for 3 hours"
                        , name, changePasswordLink));
                }
            });

        }
    }

    public void Terminate()
    { }

}
```