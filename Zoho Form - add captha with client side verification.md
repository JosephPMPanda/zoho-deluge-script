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
