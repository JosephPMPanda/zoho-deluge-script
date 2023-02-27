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
