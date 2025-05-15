---
Title: "{{title | escape}}" 
Year: {{date | format("YYYY")}} 
Authors: {{authors}} 
Tags: {% if allTags %}{{allTags}}{% endif %}
---

Zotero PDF Link: {{pdfZoteroLink}}
Related:: {% for relation in relations | selectattr("citekey") %} [[{{relation.citekey}}]]{% if not loop.last %}, {% endif%} {% endfor %} 

### Persistent Notes 
{% persist "notes" %}{% if isFirstImport %} 
Write notes here! 
{% endif %} {% endpersist %} 
### In-text annotations
  
{% for annotation in annotations -%}
{%- if annotation.annotatedText -%}
{% if annotation.color %} <mark class="hltr-{{annotation.colorCategory | lower}}">"{{annotation.annotatedText | safe}}"</mark> {% else %} {{annotation.type | capitalize}} {% endif %}[Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
{%- endif %} 
{% if annotation.comment %}
{{annotation.comment | safe}} [Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}}) 
{% endif %} 
{% if annotation.allTags %} 
{{annotation.allTags}} 
{% endif %} {% endfor -%}