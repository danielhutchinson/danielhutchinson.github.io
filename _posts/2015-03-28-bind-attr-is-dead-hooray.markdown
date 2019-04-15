---
layout: post
title:  "{{bind-attr}} is dead... hooray!"
date:   2015-03-28
categories: javascript ember
---
EmberJS 1.11 released today, and with it some nice changes and additions: such as named substates, inline IFs and the new components helper.

But for me the biggest and most exciting change is the new HTMLBars Bound Attribute Syntax. Fancy name.

What this really means is that {{bind-attr}} is gone, and I couldn't be happier.

To show you how big a deal this is, lets create a test page the old way, and then re-create it the new way.

Here's the route and controller that we'll use in both examples.

{% highlight javascript %}
// app/routes/index.js
export default Ember.Route.extend({
  model() {
      return ['red','yellow','blue']; 
  }
});

// app/controllers/index.js
export default Ember.Controller.extend({
  tomster: 'http://www.gravatar.com/avatar/0cf15665a9146ba852bf042b0652780a?s=200',
  emberLink: 'http://emberjs.com'
});
{% endhighlight %}

Here's the old index.hbs file

{% highlight javascript %}
<a {{bind-attr href=emberLink}}>
  <img {{bind-attr src=tomster}} alt="" />
</a>

<ul>
    {{#each model as |colour|}}
        <li {{bind-attr class=":list-item colour}}>{{colour}}</li>
    {{/each}}
</ul>
{% endhighlight %}

This frequent use of {{bind-attr}} can make files pretty messy, pretty quickly.

Here's the new way, using the Bound Attribute Syntax:

{% highlight javascript %}
<a href="{{emberLink}}">
  <img src="{{tomster}}" alt="" />
</a>

<ul>
  {{#each model as |colour|}}
    <li class="list-item {{colour}}">{{colour}}</li>
  {{/each}}
</ul>
{% endhighlight %}

Much tidier. I particularly like the way classes are being handled now, no more colons needed to differentiate between "static" and "dynamic" classes!

If you want to learn more about Ember 1.11 check out the Release blog post.

You should also check out the newly updated Ember Guides. They've now all been updated to use ember-cli.
