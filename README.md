# Quaderno-Templates
## What is it
Using Quaderno-Templates, you can turn any HTML page into your documents template with just a few simple tags!

Quaderno-Templates is based on [Liquid] (http://liquidmarkup.org/), a template engine widely used and created by [Shopify] (http://www.shopify.com).

Here is an example you can use to start with:
```html
<html>
<head>  
  <title>{{document.number}}</title>  
</head>
<body>
  {% if account.logo_url != "" %}
    <img src="{{account.logo_url}}" width="150" />
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
You can download a complete and functional example [here] (https://raw.githubusercontent.com/quaderno/quaderno-templates/master/professional.html).

## Usage
<sub>Please note that you should have knowledge of the (very) basics of HTML and CSS.</sub>
* First of all, you should take a look at [how Liquid works] (https://github.com/quaderno/quaderno-templates/wiki/Liquid-syntax) and understand the basics of the engine.
 
* Afterwards, we encourage you to follow our [Getting Started] (https://github.com/quaderno/quaderno-templates/wiki/Getting-started) tutorial to learn how to build a template from scratch.

* Finally, you should check the [Template variables] (https://github.com/quaderno/quaderno-templates/wiki/Variables) reference, [Template labels] (https://github.com/quaderno/quaderno-templates/wiki/Labels) reference and [Template filters] (https://github.com/quaderno/quaderno-templates/wiki/Filters) reference as you will need to use them in order to show document-specific data.

And that is pretty much all! You may now start creating your own documents in Quaderno!

## More information
* [Getting started] (https://github.com/quaderno/quaderno-templates/wiki/Getting-started)
* [Liquid syntax] (https://github.com/quaderno/quaderno-templates/wiki/Liquid-syntax)
* [Template variables] (https://github.com/quaderno/quaderno-templates/wiki/Variables)
* [Template labels] (https://github.com/quaderno/quaderno-templates/wiki/Labels)
* [Template filters] (https://github.com/quaderno/quaderno-templates/wiki/Filters)
