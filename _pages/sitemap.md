---
layout: blog
title: "Sitemap"
permalink: /sitemap/
author_profile: false
---

<div class="col-lg-12 col-md-12">
  <div class="section-title">
    <div class="section-title-name">
      <span>The List of all Pages and Posts</span>
      <h2>Sitemap</h2>
    </div>
    <div class="title-name-gray">
      <strong>sitemap</strong>
    </div>
  </div>
</div>

A list of all the posts and pages found on the site. For you robots out there is an [XML version]({{ base_path }}/sitemap.xml) available for digesting as well.

## Pages :

{% for post in site.pages %}
  {% include archive-single.html %}
{% endfor %}


<a href="#page-title" class="back-to-top">{{ site.data.ui-text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
<hr>

## Posts :

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}

{% capture written_label %}'None'{% endcapture %}

{% for collection in site.collections %}
{% unless collection.output == false or collection.label == "posts" %}
  {% capture label %}{{ collection.label }}{% endcapture %}
  {% if label != written_label %}
  <h2>{{ label }}</h2>
  {% capture written_label %}{{ label }}{% endcapture %}
  {% endif %}
{% endunless %}
{% for post in collection.docs %}
  {% unless collection.output == false or collection.label == "posts" %}
  {% include archive-single.html %}
  {% endunless %}
{% endfor %}
{% endfor %}

<a href="#page-title" class="back-to-top">{{ site.data.ui-text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
<hr>