# Automatically add Company Logos to Zoho CRM
Blog Post: https://www.squarelabs.com.au/blogs/post/automatically-add-company-logos-zoho-crm

YouTube: https://youtu.be/UvcJBB8ZgOw

A standard Zoho CRM list view is just rows of text. It's boring, and it's hard to scan because every row looks the same. Add a logo next to each account name and that problem disappears instantly, you recognise the brand before you've even read the text.

In this post we'll walk through exactly how we built it: a function that grabs the logo the moment an account is created or its website is updated, and a Canvas list view that shows it off. No searching for logos, no downloading them, no uploading them into the record by hand.

## Configuration Guide

Here are the steps to take to configure this automation within Zoho CRM.

### Get your Logo.dev API Key

First you must create an account at [logo.dev](https://www.logo.dev). Once you have your account you will be able to retrieve your API Key.

### Create Connection

Before we build the function, we need a connection that lets it upload the logo to the account record in the background using the Zoho API.

- Head to Setup > Developer Space > Connections
- Click New Connection > Zoho CRM
- Select the scope `ZohoCRM.modules.ALL` (if you have an existing connection with this scope you can use that one instead)
- Authorise the connection

### Build the Function

Now that the connection is ready we can create our custom function.

- Head to Setup > Developer Space > Functions
- Create a new function called `getAccountLogos` with the category of Automation
- Set the Argument to `string accountId`
- Copy in the below code snippet between the default curly brackets
- Insert your logo.dev API Key
- Ensure your connection name matches the one you created
- Ensure the API domain is correct based on your Data Centre
- Click Save

```js
//Get the Account Record
accountMap = zoho.crm.getRecordById("Accounts",accountId);
//
website = accountMap.get("Website");
//If Website field is empty do nothing
if(!isnull(website))
{
	//Remove any http at the start of the URL
	if(website.startsWith("http"))
	{
		website = website.getSuffix("//");
	}
	//Remove any URL Parameters at the end of the URL
	if(website.contains("?"))
	{
		website = website.getPrefix("?");
	}
	//Remove any URL Paths at the end of the URL
	if(website.contains("/"))
	{
		website = website.getPrefix("/");
	}
	//Call the Logo.dev API
	accountLogoResponse = invokeurl
	[
		url :"https://img.logo.dev/" + website + "?token=<<LOGO.DEV API KEY>>&retina=true"
		type :GET
		detailed:true
	];
	info accountLogoResponse;
	//If the response is a 200 Success continue.
	if(accountLogoResponse.get("responseCode") == 200)
	{
		//Get the Logo
		accountLogo = accountLogoResponse.get("responseText");
		accountLogo.setParamName("file");
		//Upload the logo to the account.
		uploadImageResponse = invokeurl
		[
			url :"https://www.zohoapis.com.au/crm/v8/Accounts/" + accountId + "/photo?restrict_triggers=workflow"
			type :POST
			files:accountLogo
			connection:"<<CONNECTION NAME>>"
		];
		info uploadImageResponse;
	}
	else
	{
		throws "Failed to Retrieve Logo from Logo.dev";
	}
}
```

### Create Workflow Rules

We are going to use 2 workflow rules so the function only triggers when required.

- Head to Setup > Automation > Workflow Rules
- Create a new Workflow Rule called `Get Account Logo - Create`
- Add in Condition: Website is not empty
- Select Function > Configure Function > find the new function we created and click Configure
- Map the `accountId` argument by clicking the `#` and selecting Account > Account ID
- Click Save
- Clone this newly created Workflow Rule
- Change the Trigger to Edit > Specific Field > Website > Any Value (select the Repeat checkbox)

### Custom List View

Now the last component is to have a custom list view.

- Head to Setup > Customisation > Canvas
- Select the Accounts Module > Custom List View (if it doesn't exist, click New)
- Ensure that you have the Account Image field in the list view
- Design up your other view components

### Test

Now it's all complete, you can select Custom List View in your Accounts Module and view the account logos.

## Need Help?

[Contact us](https://squarelabs.com.au/contact)!

## Resources

- Zoho Custom List Views: https://help.zoho.com/portal/en/kb/crm/customize-crm-account/canvas/articles/customizing-module-view-using-the-all-new-canvas-design-suite
- Logo.dev API: https://www.logo.dev/products/logo-api

<a href="http://www.youtube.com/watch?feature=player_embedded&v=UvcJBB8ZgOw" target="_blank"><img src="http://img.youtube.com/vi/UvcJBB8ZgOw/0.jpg" 
alt="YouTube Video" width="240" height="180" border="10" /></a>
