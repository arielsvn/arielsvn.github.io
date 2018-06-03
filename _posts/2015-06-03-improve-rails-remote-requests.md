---
title: Improve rails remote requests
date: 2015-06-03 09:00:01 Z
categories:
- rails,
- js
layout: post
---

When using Rails, it's extremely convenient to use `:remote` requests. You can make any link or form to submit an ajax request with some data, out of the box, just adding a parameter to the `link_to` helper or other of the form elements.

```erb
<%= link_to 'Delete', delete_path, remote: true %>
```

or with some form element:

```erb
<%= form_tag delete_path, remote: true, method: :post do ... end %>
```

All this is very good, but most of the time you need to make modifications to the page after the request is completed successfully. These changes can be applied by listening to the event `ajax:success` on the link or form elements marked as remote; however if the changes are simple, like removing, adding a class or replacing the content of an element, it's better if they are sent from the server using some convention.

An option would be to encode all these calls into the JSON response, for example if you wanted to hide the elements marked with the class `.js-hide` then the response could be something like:

```json
{
	"page": {
		"hide": ".js-hide"
	}
}
```

and to replace the HTML of an element marked with the class `.js-foo` we could use:

```json
{
	"page": {
		"html": {
			".js-foo": "<h1>New content here</h1>"
		}
	}
}
```

But this isn't all, we have to add some code on the client side to process these responses upon successful requests.

```coffee
$document.on 'ajax:success', '[data-remote="true"]', (e, data)->
  # get the object with the instructions
  options = data.page
  
  for method in ['hide', 'show', 'remove']
    $(options[method])[method]()

  for method in ['html', 'append']
    if options[method]
      $.each options[method], (element, content)->
        $(element)[method](content)
```


