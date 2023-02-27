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
