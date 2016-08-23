---
layout: post
title:  "Override-able options in rails partial view"
date:   2016-08-22 09:00:01
categories: rails
---

Sometimes you have partial views with static values that would be nice to change, but adding specific options for each one of them would be painful.

A better approach is to have all these values in a hash, at the beginning of the partial view. And allow changing all of them with a parameter.

```erb
<% 
options = {
  styles: {
    main: stylesheet_url('external/styles'),
    list: stylesheet_url('external/list'),
    side: stylesheet_url('external/side'),
  }
}.merge((defined?(options) && options) || {})
%>

<div>
	Use options here as usual...

	<p><%= options[:styles][:main] %></p>
	<p><%= options[:styles][:list] %></p>
	<p><%= options[:styles][:side] %></p>
</div>
```

This partial view can be rendered as usual, with the only difference that you can pass an additional object `options` and override each one of the values there.

<%= render 'partial_view_name', options: {styles: { main: stylesheet_url('other/main') }} %>

Note that you don't need to pass all the parameters, just the ones you need to change. As both hashes are merged using ![deep merge](http://apidock.com/rails/Hash/deep_merge).

