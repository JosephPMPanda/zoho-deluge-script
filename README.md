# Table Of Content
documenting all deluge script I've known because Zoho documentations are confusing

- [Zoho CRM](#zoho-crm)
  - [Autofill increment number](#autofill-increment-number)
  - [Autofill Lookup Data](#autofill-lookup-data)
  - [Copy Subform Content](#copy-subform-content)
  - [Create New Association to Related Module](#create-new-association-to-related-module)
  - [Custom Widget Basic](#custom-widget-basic)
  - [Execute Deluge Custom Function in Another Deluge Custom Function](#execute-deluge-custom-function-in-another-deluge-custom-function)
  - [Generate Access Token in Deluge](#generate-access-token-in-deluge)
  - [Xero integration - Fetch Invoice Data](#xero-integration---fetch-invoice-data)
  - [Xero Integration - Generate Access Token](#xero-integration---generate-access-token)
  - [Xero Integration - Pull Products](#xero-integration---pull-products)
  - [Zoho Analytics Integration - Pull Data With Criteria](#zoho-analytics-integration---pull-data-with-criteria)
  - [Zoho Writer Integration - Push Subform Data](#zoho-writer-integration---push-subform-data)
  - [Add Attachment](#add-attachment)
  - [Create New Task](#create-new-task)
  - [Round Robin Assignment](#round-robin-assignment)
  - [Send Email Alert](#send-email-alert)
- [Zoho Form](#zoho-form)
  - [Auto Show Element When Checked](#auto-show-element-when-checked)
  - [Captha With Client Side Validation](#captha-with-client-side-validation)
  - [RAW HTML No Redirection Submission](#raw-html-no-redirection-submission)

# Zoho CRM
## Autofill Increment Number
This is made with client script
```
/**
 * Fetch the last biggest project number from a module
 * then display it on record creation
 */
let project_number = ZDK.Apps.CRM.Module_name.fetch(1, 1 ,"hidden_field_to_track", "desc");
let last_project_number = project_number[0]._hidden_field_to_track;
let next_project_number = parseInt(last_project_number, 10) + 1;
let project_number_to_display = "SM-" + next_project_number;
ZDK.Page.getField("Project_Number").setValue(project_number_to_display);
```

## Autofill Lookup Data
This is made with client script
```
//NOTE: a look up field must be present on both
// module! otherwise .getValue() will be undefined!

let fieldValue = ZDK.Page.getField("Contact").getValue();
let contact = ZDK.Apps.CRM.Contacts.fetchById(fieldValue.id);

ZDK.Page.getField("Email").setValue(contact.Email)
ZDK.Page.getField("Phone").setValue(contact.Phone)
```

## Copy Subform Content
This is made with client script
```
subform_1_value = ZDK.Page.getForm().getValues().Subform_1
ZDK.Page.getForm().setValues({"Subform_2": [] }) //reset form to prevent missing row
ZDK.Page.getForm().setValues({"Subform_2": subform_1_value })
```

## Create New Association to Related Module
This is made with deluge

required scope in connection:


```
//for default related module
data = {
         "data" : [
                    {
                      "id":product_id,
                      ...
                    },
                    ...
                  ]
        };

resp = invokeurl
[
  url :"https://www.zohoapis.com/crm/v3/Contacts/contact_id/Products"
	type :PUT
 	parameters: data.toText()
 	connection:"zoho_crm"
];
info resp;
```

## Custom Widget Basic
The HTML
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <script src="https://live.zwidgets.com/js-sdk/1.1/ZohoEmbededAppSDK.min.js"></script>
  </head>

  <body>
    <div>
      <button onclick=execute_crm_custom_function>Submit</button>
    </div>
    <script src="script.js"></script>
  </body>
</html>
```

script.js
```
const execute_crm_custom_function = () => {
    ZOHO.CRM.FUNCTIONS.execute("custom_function_name", { "custom_function_input_1": var1, "custom_function_input_2": var2})
        .then(()=>{
            //do something after execution
        })
}

ZOHO.embeddedApp.on("PageLoad", function (data) {
    console.log(JSON.stringify(data));
    for (let id in data.EntityId) {
        // console.log(`id = ${id}`);
        // get record
        // console.log(`data.EntityId[id] = ${data.EntityId[id]}`)
        ZOHO.CRM.API.getRecord({ Entity: "Contacts", RecordID: data.EntityId[id] }).then(function (contactData) {
            // console.log(contactData)
            document.getElementById("recipient").innerHTML += contactData.data[0].Email
            email_recipient += contactData.data[0].Email + ","
            document.getElementById("recipient").innerHTML += "<br>"
        })

    }
})

ZOHO.embeddedApp.init()
```

## Execute Deluge Custom Function in Another Deluge Custom Function
Reference :
[https://www.zoho.com/developer/help/extensions/rest-api.html](https://www.zoho.com/developer/help/extensions/rest-api.html)

To be able to run the custom function, the custom function that need to be run must have the REST API enabled

For example we have a custom function named display_contact_name
```
//function display_contact_name
//parameters: contact_id

info zoho.crm.getRecordById("Contacts",contact_id);
```

After we enabled the REST API of display_contact_name function, we call this function like below

```
param = {"arguments":{"contact_id":5678912345}};
resp = invokeurl
[
  url :"https://www.zohoapis.com.au/crm/v2/functions/display_contact_name/actions/execute?auth_type=apikey&zapikey=1003.xxxxx.xxxxx"
  type :POST
  parameters:param
  content-type:"multipart/form-data"
];
```
## Generate Access Token in Deluge
**_use connection in CRM for easier token management_**

The refresh token must already generated in another way first

To generate a new access token from refresh token

```
refresh_token = "...";
client_id = "...";
client_secret = "...";

//generate access token
url_access_token = "https://accounts.zoho.com/oauth/v2/token"+"?refresh_token="+refresh_token+"&client_id="+client_id+"&client_secret="+client_secret+"&grant_type=refresh_token";
resp = invokeurl [
	url: url_access_token
	type: POST
]
access_token = resp.get("access_token");
```

## Xero integration - Fetch Invoice Data
```
/**
 * generate new access token
 * https://help.zoho.com/portal/en/community/topic/zoho-crm-call-custom-function-from-another-custom-function
 * https://github.com/gauraputu/zoho-deluge-script/blob/main/Zoho%20CRM%20-%20Xero%20get%20access%20token%20from%20refresh%20token
 */
invokeUrl
[
	url: "https://www.zohoapis.com/crm/v2/functions/get_xero_access_token/actions/execute?auth_type=apikey&zapikey=xxxxxxxx"
	type: GET
];

//GET invoices data from Xero
xero_tenant_id = zoho.crm.getOrgVariable("xero_tenant_id");
xero_access_token = zoho.crm.getOrgVariable("xero_access_token");
get_xero_invoices = invokeurl
[
	url :"https://api.xero.com/api.xro/2.0/Invoices"
	type :GET
	headers:{"xero-tenant-id":xero_tenant_id,"Authorization":"Bearer " + xero_access_token,"Accept":"application/json","Content-Type":"application/json"}
];
/** 
 * response if success
 * {
 *   "Id": "...",
 *   "Status": "...",
 *   "ProviderName": "...",
 *   "DateTimeUTC": "...",
 *   "Invoices":
 *       [
 *            {},
 *            {}
 *       ]
 * }
 *
 * response if error (access token expired)
 * {
 *   "Type": null,
 *   "Title": "Unauthorized",
 *   "Status": 401,
 *   "Detail": "TokenExpired: token expired at 08/12/2022 07:00:40",
 *   "Instance": "a233ca33-cdb9-48b0-81c2-b6217f1fcfdb",
 *   "Extensions": {}
 * }
 */

return "";
```

## Xero Integration - Generate Access Token
```
/** 
 * Fetch access token from zoho CRM variables
 */
xero_tenant_id = zoho.crm.getOrgVariable("xero_tenant_id");
xero_access_token = zoho.crm.getOrgVariable("xero_access_token");
xero_refresh_token = zoho.crm.getOrgVariable("xero_refresh_token");
xero_clientId_and_clientSecret_base64 = zoho.crm.getOrgVariable("xero_clientId_and_clientSecret_base64");

response = invokeurl
[
	url :"https://identity.xero.com/connect/token"
	type :POST
	parameters:{"grant_type":"refresh_token","refresh_token":xero_refresh_token}
	headers:{"Authorization":"Basic " + xero_clientId_and_clientSecret_base64,"Content-Type":"application/x-www-form-urlencoded"}
];

/** 
 * response if success
 * {
 *   "id_token": "...",
 *   "access_token": "...",
 *   "expires_in": 1800,
 *   "token_type": "Bearer",
 *   "refresh_token": "...",
 *   "scope": "..."
 * }
 *
 * response if error
 * {
 *    "error": "..."
 * }
 */
if(response.get("error") != null)
{
	//failed to create access token
}
else
{
	//save the new access token and refresh token
	zoho.crm.invokeConnector("crm.set",{"apiname":"xero_access_token","value":response.get("access_token")});
	zoho.crm.invokeConnector("crm.set",{"apiname":"xero_refresh_token","value":response.get("refresh_token")});
}

return "";
```

## Xero Integration - Pull Products
```
/**
 * generate new access token
 * https://help.zoho.com/portal/en/community/topic/zoho-crm-call-custom-function-from-another-custom-function
 * https://github.com/gauraputu/zoho-deluge-script/blob/main/Zoho%20CRM%20-%20Xero%20get%20access%20token%20from%20refresh%20token
 */
invokeurl
[
	url :"https://www.zohoapis.com/crm/v2/functions/get_xero_access_token/actions/execute?auth_type=apikey&zapikey=1003.ddf1e707217da8929de8ee7f84ecdf92.c15556ad10ec3783e3ba7dc12e358637"
	type :GET
]

//GET product data from Xero
xero_tenant_id = zoho.crm.getOrgVariable("xero_tenant_id");
xero_access_token = zoho.crm.getOrgVariable("xero_access_token");
get_xero_products = invokeurl
[
	url :"https://api.xero.com/api.xro/2.0/Items"
	type :GET
	headers:{"xero-tenant-id":xero_tenant_id,"Authorization":"Bearer " + xero_access_token,"Accept":"application/json","Content-Type":"application/json"}
];
/**
 * response if success
 * {
 *     "Id": "d2b6337f-4c3a-4b3a-a1a1-adfbbe7961a9",
 *     "Status": "OK",
 *     "ProviderName": "zoho crm",
 *     "DateTimeUTC": "/Date(1660534545691)/",
 *     "Items":[{},{},...]
 * }
 */
 
//create record in Zoho CRM
for each  item in get_xero_products.get("Items")
{
	import_to_zoho = {
    "Product_Name":item.get("Name"),
    "Product_Code":item.get("Code")
  };
	zoho.crm.createRecord("Products",import_to_zoho);
}

return "";
```

## Zoho Analytics Integration - Pull Data With Criteria
```
//generate access token
refresh_token = "...";
client_id = "...";
client_secret = "...";
url_access_token = "https://accounts.zoho.com/oauth/v2/token" + "?refresh_token=" + refresh_token + "&client_id=" + client_id + "&client_secret=" + client_secret + "&grant_type=refresh_token";
resp = invokeurl
[
	url :url_access_token
	type :POST
];
access_token = resp.get("access_token");
// get data from zoho analytics
workspace_id = "...";
view_id = "...";
org_id = "...";
//criteria: fetch records last modified within 24 hours
criteria = "%22to_integer(unix_timestamp(now()))-to_integer(unix_timestamp(%5C%22Last%20Modified%20Date%5C%22))%3C36*60*60%22";
//nonalphanumeric character must be converted to hex preceding with %
url_analytic_agencies = "https://analyticsapi.zoho.com/restapi/v2/workspaces/" + workspace_id + "/views/" + view_id + "/data?CONFIG=%7BresponseFormat:json," + "criteria:" + criteria + "%7D";
data_from_analytic = invokeurl
[
	url :url_analytic_agencies
	type :GET
	headers:{"Authorization":"Zoho-oauthtoken " + access_token,"ZANALYTICS-ORGID":org_id}
];
//push data to CRM for every 100 records
data_to_push_to_crm = {"data":List()};
data_size = 0;
temp_list = List();
for each  record in data_from_analytic.get("data")
{
	if(data_size < 100)
	{
		//data to be pushed <100
		temp = Map();
		temp.put("Account_Name",record.get("..."));
    ...
		temp_list.add(temp);
		data_size = data_size + 1;
	}
	else
	{
		//data to be pushed =100
		data_to_push_to_crm.put("data",temp_list);
		resp3 = invokeurl
		[
			url :"https://www.zohoapis.com/crm/v3/Accounts/upsert"
			type :POST
			parameters:data_to_push_to_crm.toText()
			headers:{"Authorization":"Zoho-oauthtoken " + access_token}
		];
		//reset map 
		data_to_push_to_crm.put("data",List());
		temp_list = List();
		data_size = 0;
		//push data again
		temp = Map();
		temp.put("Account_Name",record.get("..."));
		temp_list.add(temp);
		data_size = data_size + 1;
	}
}
data_to_push_to_crm.put("data",temp_list);
// push the last remaining data to CRM
info "=========\n" + data_to_push_to_crm.toText();
resp4 = invokeurl
[
	url :"https://www.zohoapis.com/crm/v3/Accounts/upsert"
	type :POST
	parameters:data_to_push_to_crm.toText()
	headers:{"Authorization":"Zoho-oauthtoken " + access_token}
];
info "resp4: " + resp4;
```

## Zoho Writer Integration - Push Subform Data
```
/**
we have subform in quotation module, which we want to create a button that auto generate quote.
we have generated template and we have mapped a sub-form named "sub_forms" in that template.
*/

getQuote = zoho.crm.getRecordById("Quotations",quoteId);
quoteType = getQuote.get("Type_of_Quote");
/*... get the rest of the field */

/*begin get data from sub-form*/
sub_forms = List();
productDet = ifnull(getQuote.get("Quoted_Products"),"");
for each  prod in productDet
{
	pname = prod.get("Product").get("name");
	spec = prod.get("Specifications");
	qty = prod.get("Quantity");
	amount = prod.get("Amount");

	info "name : " + pname;
	info "spec : " + spec;
	info "qty : " + qty;
	info "amount : " + amount;

	subform = Map();
	subform.put("sub_forms.Product",prod.get("Product").get("name")); //here "sub_forms" is the name of the subform in the template
	subform.put("sub_forms.Quantity",prod.get("Quantity"));
	subform.put("sub_forms.Specifications",prod.get("Specifications"));
	subform.put("sub_forms.Amount",prod.get("Amount"));
	sub_forms.add(subform);
}

/*begin sending data to Zoho Writer*/
mergeFields = {"data":{{"Contact_Name":contactName,"..the rest of the field","sub_forms":sub_forms}}};
writerId = "9kjuyc849f0805cf64d70a981624e2cc9ffa7";
mapData = Map();
mapData.put("output_format","docx");
mapData.put("filename","Battery Quote_" + contactName);
mapData.put("merge_data",mergeFields);
mergeDown = invokeurl
[
  url :"https://zohoapis.com.au/writer/api/v1/documents/" + writerId + "/merge/download"
  type :POST
  parameters:mapData
  connection:"merge_writer"
];
info "Battery: " + mergeDown;
attachFile = zoho.crm.attachFile("Quotations",quoteId,mergeDown);
info "Battery: " + attachFile;
return "Success";
```

## Add Attachment
```
lead_record = zoho.crm.getRecordById("Leads",lId);
signed_doc = zoho.sign.downloadDocument(lead_record.get("Request_ID"));
resp = zoho.crm.attachFile("Leads",lId,signedDoc);
```

## Create New Task
```
lead_record = zoho.crm.getRecordById("Leads",lId);

//create new task
subject = lead_record.get("First_Name") + " " + lead_record.get("Last_Name") + " has signed. notify them";
dueDate = today.addDay(0);
tMap = Map();
tMap.put("$se_module","Leads");
tMap.put("What_Id",lId);
tMap.put("Subject",subject);
tMap.put("Due_Date",dueDate);
tMap.put("Status","Not Started");
tMap.put("Priority","High");
createResp = zoho.crm.createRecord("Tasks",tMap);
```

## Round Robin Assignment
```
//list of user ID assigned for round robin
sales1 = "1083734000001277002";
sales2 = "1083734000002033005";
sales3 = "1083734000000150025";

//check last assigned sales then replace the value with the next sales
last_assigned_sales = zoho.crm.getOrgVariable("smsf_service");
if(last_assigned_sales = sales1)
{
	zoho.crm.invokeConnector("crm.set",{"apiname":"smsf_service","value":sales2});
}
else if(last_assigned_sales = sales2)
{
	zoho.crm.invokeConnector("crm.set",{"apiname":"smsf_service","value":sales3});
}
else if(last_assigned_sales = sales3)
{
	zoho.crm.invokeConnector("crm.set",{"apiname":"smsf_service","value":sales1});
}
else
{
	//to restart the round robin just replace crm variable to value other than defined in user ID
	zoho.crm.invokeConnector("crm.set",{"apiname":"smsf_service","value":sales1});
}
//assign round robin sales
sales_to_assign = zoho.crm.getOrgVariable("smsf_service");
zoho.crm.updateRecord("Leads",id,{"Owner":sales_to_assign});
```

## Send Email Alert
```
getRelatedScopeOfWork = zoho.crm.getRelatedRecords("Scope_of_Work12","Deals",OpportunitiesId);
//info getRelatedScopeOfWork;
emailRecipientList = List();
for each  item in getRelatedScopeOfWork
{
	//info item.get("Scope_of_Work").get("id");
	ScopeOfWorkId = item.get("Scope_of_Work").get("id");
	getRelatedDBM = zoho.crm.getRelatedRecords("DBM1","Scope_of_Work",ScopeOfWorkId);
	//info getRelatedDBM;
	for each  item in getRelatedDBM
	{
		DBMId = item.get("DBM_Relate").get("id");
		//info DBMId;
		for each  item in DBMId
		{
			emailRecipientList.add(zoho.crm.getRecordById("DBM",DBMId).get("Email"));
		}
	}
}
emailRecipientList = emailRecipientList.distinct();
info emailRecipientList;
/*start of --------for testing purprose-------------------------*/
testEmailRecipient = {"01@example.com","02@example.com"};
sendmail
[
	from :zoho.loginuserid
	to :testEmailRecipient
	cc:"03@example.com"
	subject :"New Opportunities Alert"
	message :"<div>There is a new Opportunities!<br></div>"
]
/*end of --------for testing purprose-------------------------*/
/*
sendmail
[ 
	from : zoho.loginuserid
	to : emailRecipientList
	cc : "03@example.com"
	subject: "New Opportunities Alert"
	message : "<div>There is a new Opportunities!<br></div>"
]
*/
return "";
```

# Zoho Form
## Auto Show Element When Checked
```
/** 
 * Checkbox toggler
 */
<form>
  ...
  <input id="checkboxToggler" type="checkbox">
  <div style="display:none" id="elementToToggle">...</div>
  ...
</form>
<script>
  let checkboxToggler = document.getElementById('checkboxToggler');
  checkboxToggler.addEventListener('click', (e) => {
    if (checkboxToggler.checked) {
      document.getElementById('elementToToggle').style.display = "block"
    } else {
      document.getElementById('elementToToggle').style.display = "none" 
    }
  })
</script>

/** 
 * Dropdown toggler
 */
<form>
  ...
  <select id="dropdownToggler">
    <option selected="true" value="-Select-">-Select-</option>
    <option value="Yes">Yes</option>
    <option value="No">No</option>
  </select>
  <div style="display:none" id="elementToToggle">...</div>
  ...
</form>
<script>
  let dropdownToggler = document.getElementById('dropdownToggler');
  dropdownToggler.addEventListener('click', (e) => { 
    if (dropdownToggler.value === "Yes") {
      document.getElementById('elementToToggle').style.display = "block"
  } else {
    document.getElementById('elementToToggle').style.display = "none"
  }
  })
</script>

/** 
 * radio button toggler
 */
<form>
  ...
  <input type="radio" id="radioToggler1" name="Radio" value="Yes">...
  <input type="radio" id="radioToggler2" name="Radio" value="No">...
  <div style="display:none" id="elementToToggle">...</div>
  ...
</form>
<script>
  let radioToggler1 = document.getElementById('radioToggler1');
  let radioToggler2 = document.getElementById('radioToggler2');
  radioToggler1.addEventListener('change', (e) => {
      if (radioToggler1.checked) {
          document.getElementById('elementToToggle').style.display = "block"
      }
  })
  radioToggler2.addEventListener('change', (e) => {
      if (radioToggler2.checked) {
          document.getElementById('elementToToggle').style.display = "none"
      }
  })
</script>
```

## Captha With Client Side Validation
When zoho form is embeded as raw HTML, the captcha that set in Zoho form does not get exported.

We could use 3rd party captha but we need a server to validate the captha.

The workaround is to validate the captha on client side. The disadvantage is this client side validation can't really tell if the captha answer is correct or not. What it can tell is if the captha is answered or not.

The example below will use h-captha.

```
<form id="form">
  <div id="h-captha"></div>
<form>
<script>
  //add capthca verification
  document.getElementById("form").addEventListener("submit", function (e) {
    var hcaptchaVal = $('iframe').attr("data-hcaptcha-response");
    console.log(hcaptchaVal);
    if (hcaptchaVal == null || hcaptchaVal == "") {
      e.preventDefault();
      alert("Please complete the hCaptcha");
      return false
   }
  });
</script>
```
## RAW HTML No Redirection Submission
To submit embeded zoho form without redirection to thank you page we can use AJAX when submitting the form.


```
<form id="test-form">
.....
</form>
<script>
  let submitForm = function (e){
    fetch('https://forms.zohopublic.com/.../htmlRecords/submit', {
          method: 'POST',
          headers: {},
          body: new FormData(document.getElementById("test-form"))
      }).then((response) => {
        //do something on success
      })
        .catch((error) => {
          //do something on error
          console.log(error)
        });
    e.preventDefault();
  };
  document.getElementById("test-form").addEventListener("submit", submitForm, false);
</script>
```
