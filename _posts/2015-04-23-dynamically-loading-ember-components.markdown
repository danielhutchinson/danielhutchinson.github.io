---
layout: post
title:  "Dynamically loading Ember components"
date:   2015-04-23
categories: javascript ember
---
Today I had to solve a problem where components needed to be rendered on a page dynamically, initially I thought of using Partials for this rather than components, but since each panel needed to have its own isolated functionality this wouldn't have been a very good fit. This kind of functionality is exactly what components are for after all!

Luckily, in Ember version 1.11.0 we have access to the {{component}} helper which lets us render components dynamically by name, which just so happens is exactly what I need.

Say for example I have three components ```dh-panel-red```, ```dh-panel-green```, and ```dh-panel-blue```. Each panel follows a simple format as follows:

{% highlight javascript %}
export default Ember.Component.extend({
  classNames: ['dh-panel'],
  classNameBindings: ['panelColor'],
  panelColor: 'red'
});
{% endhighlight %}

Each component has the same code as this, the only thing that changes is the panelColor property. Which is used to set the background colour of the panel in css.

Note: In practice it'd be best to create a mixin for any repeated code. I haven't done that here for the sake of brevity.

Now on the application template I'll add a drop down select menu that allows the user to pick and choose which component they want. This template is powered by data located on the application controller.

### Application Template
{% highlight javascript %}
  <label>
    Select a component to load:
    {{view "select"
           content=components
           optionValuePath="content"
           optionLabelPath="content.title"
           value=selectedComponent}}
  </label>
{% endhighlight %}

### Application Controller
{% highlight javascript %}
export default Ember.Controller.extend({
  components: [
    { title: "Red Panel", "name": "dh-panel-red" },
    { title: "Blue Panel", "name": "dh-panel-blue" },
    { title: "Green Panel", "name": "dh-panel-green" }
  ]
});
{% endhighlight %}

The magic happens in the new component helper, this helper expects the name of a component to be passed to it, so, for example, if we wanted to load our dh-panel-red component we could do it this way:

{% highlight javascript %}
{{component "dh-panel-red title="Red Panel"}}
{% endhighlight %}

Though hard coding a value like this is a but redundant, we could just use the normal method of rendering a single component on the page, the value of this helper comes from the ability to set the component name to a dynamically bound value, in our example it would look like this:

{% highlight javascript %}
{{component selectedComponent.name title=selectedComponent.title}}
{% endhighlight %}

When the value of the selectedComponent changes, so too does the component itself. This allows us to swap between different components at run time.

I can already think of lots of different uses for this new component. I'd be interested to hear about any ideas you've come up with too!