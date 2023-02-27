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