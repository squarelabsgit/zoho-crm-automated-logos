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
