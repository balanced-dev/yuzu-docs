# Membership

Simple membership solutions are hard to get right, there are so many moving parts. We have created a simple, adaptable solution for adding membership functionality to Yuzu enabled Umbraco sites. Definition have complete control of the UI and delivery can change Viewmodel data before render and choose when and where there forms are delivered and rendered.

It's not intended as a cure all solution, but as a quick approach for smaller sites that don't need overly complex logic and it works well. It also demonstrates how we are working with Yuzu; by separating out the UI and CMS integration we can create packages that are very adaptable with minimum code implementation.

The following forms are included

- register
- login
- password reminder
- change member
- change password

For form fields we use the standard Viewmodels using in Yuzu forms. 

# Viewmodel

All members forms use the same Viewmodel, `vmBlock_DataFormBuilder`, `FormBuilderFactory` and `YuzuFormViewModel` from the Forms package. It also uses the YuzuForms partial view.

By using the same Viewmodel we can standardise membership forms through the site, making it easier to reuse, maintain and change in the future.

# Umbraco Membership Integration

Umbraco itself comes with some basic membership integration in the form of controllers. These handle login, logout, register and profile changes. We have used the login and register controllers here. 

Using Yuzu membership doesn't stop you from using all the other extension points that Umbraco includes. For example, use a member service event to send an email on user registration.

# Configuration

An interface to change the membership controller settings, they standardize properties whilst making the controller as adaptable without needing new controllers. 

| Property    			    	| Purpose 			        |Default 			        |
| ----------------------------- | --------------------------|---------------------------|
| **HomeUrl**		            |                           |                           |
| **ForgottenPasswordUrl**      |                           |                           |
| **ChangePasswordUrl**		    |                           |                           |
| ForgottenPasswordLabel        | Button Label              | Forgotten Password?       |
| EmailNotFoundErrorMessage     |                           | Email address not found   |
| MemberNotFoundErrorMessage    |                           | Member not found          |
| ForgottenPasswordEmailAction  |                           |                           |

The `ForgottenPasswordEmailAction` is to add your own code to the send the forgotten password link. It is passed the email address, name and a link to change the password.

## Controllers

We have split the initial state actions from the response handlers into 2 separate controller (`MemberFormController` and `MemberHandlerController`). We have found that when we create a new using this package the `MemberFormController` needs to be adapted more than the `MemberHandlerController`. Splitting them makes it easier for us to swap out the `MemberFormController` for a project specific implementation.

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
            var changePasswordNode = root.Descendant<ChangePassword>();

            YuzuDeliveryMembers.Initialize(new YuzuDeliveryMembersConfiguration()
            {
                HomeUrl = root.Url,
                ForgottenPasswordUrl = forgottenPasswordNode.Url,
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