## About
This shortcode lets you automatically fill the parameters of a url without having to worry about whether it's a route or a query parameter. This is helpful for automatically dealing with route parameters in the LinkedPages of some blocks.

Example Usage:
```liquid
{% assign GroupId = 34 %}
{% assign PersonId = 2853 %}
{% assign DoThing = true %}
{[ FormatLink url:'https://example.org/test/{GroupId}/{PersonId}' keys:'GroupId,PersonId,DoThing' ]}
```
Output:
```
https://example.org/test/34/2853?DoThing=true
```

## Shortcode Settings

**Tag Name:** FormatLink

**TagType:** Inline

**Description:** Formats a link with parameters.

**Documentation:** 
```html
<p>Usage:</p>
<pre>{% assign Param1 = 1234 %}
{% assign Param2 = 5678 %}
{% assign Param3 = 90 %}
{[ FormatLink url:'https://example.org/stuff/{Param1}/{Param2}' keys:'Param1,Param2,Param3' ]}</pre>
<p>Output:</p>
<pre>https://example.org/stuff/1234/5678?Param3=90</pre>
```

**Shortcode Markup:**
```liquid
{% assign Keys = keys | Split:',' %}
{% for Key in Keys %}
    {% assign Value = [Key] | AsString | EscapeDataString %}
    {% assign KeyPlaceholder = '{' | Append:Key | Append:'}' %}
    {% if url contains KeyPlaceholder %}
        {% assign url = url | Replace:KeyPlaceholder,Value %}
    {% elseif url contains '?' %}
        {% capture url %}{{ url }}&{{ Key | EscapeDataString }}={{ Value }}{% endcapture %}
    {% else %}
        {% capture url %}{{ url }}?{{ Key | EscapeDataString }}={{ Value }}{% endcapture %}
    {% endif %}
{% endfor %}
{{ url }}
```
