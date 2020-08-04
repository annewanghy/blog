jquery真的很强大，虽然很多人都在说已经过时了，但是有时候，但是用起来还是很帮

* 一般用法，查询dom



|原生|更加简便的方法|jquery|
|---: | :---: | :----|
|`document.getElementById('id')`|`document.querySelector('#id')`|`$('#id')`|
|`document.getElementsByClassName('.className')`|`document.querySelector('.className')`|$('.className')|


form表单用法
```js
$(document).ready(function(){
  $("#form1").validate({
    errorClass: "error",
    errorElement: "div",
    errorPlacement: function(error, element) {   
     element.after(error);
    },
    rules: { 
      username: { required: true, minlength: 6},
      password: { required: true, minlength: 6}
    },
    messages: {
      username: { required: "require", minlength: $.validator.format("must > 0")},
      password: { required: "requied", minlength: $.validator.format("must > 0")}
    },
    success: function(label) {
      alert("success");
    },
    submitHandler: function(form){
        alert("submit");    
        form.submit(); // submit form
    }
  });
});
```
