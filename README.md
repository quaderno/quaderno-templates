# Quaderno-Templates
## What is it
Using Quaderno-Templates, you can turn any HTML page into your documents template with only a few simple tags!

Quaderno-Templates is based on [Liquid] (http://liquidmarkup.org/), a template engine widely used and created by [Shopify] (http://www.shopify.com).

Here is an example you can use to start with:
```html
<html>
<head>  
  <title>{{document.number}}</title>  
</head>
<body>
  {% if account.logo_url != "" %}
    <img src="{{account.logo_url}}" width="150" /></td>    
  {% endif %}
  <h1>{{title}}</h1>
  <table>
    {% for item in document.items %}
      <tr>
        <td>{{item.description}}</td>
        <td>{{item.quantity}}</td>
        <td class="right">{{item.unit_price}}</td>
        <td class="right">{{item.subtotal}}</td>
      </tr>
    {% endfor %}
  </table>
</body>
</html>
```
You can download a complete and functional example [here] (https://raw.github.com/recrea/quaderno-templates/master/template.html).

## Usage
<sub>Please note that you should have knowledge of the (very) basics of HTML and CSS.</sub>
* First of all, you should take a look at [how Liquid works] (https://github.com/recrea/quaderno-templates/wiki/Liquid-syntax) and understand the basics of the engine.
 
* Afterwards, you should check the [Quaderno-Templates reference] (https://github.com/recrea/quaderno-templates/wiki/Reference) as you will need to use it in order to show document-specific data.

* And that is pretty much all! You may now start creating your own documents in Quaderno!

## More information
* [Liquid syntax] (https://github.com/recrea/quaderno-templates/wiki/Liquid-syntax)
* [Quaderno-Templates reference] (https://github.com/recrea/quaderno-templates/wiki/Reference)