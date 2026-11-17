{% assign bg_files = site.static_files | where_exp: "file", "file.path contains '/backgrounds/'" | sort: "path" %}

{% for file in bg_files %}
   {{ file.path | relative_url }}
{% endfor %}
