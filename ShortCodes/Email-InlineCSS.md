**Tag name:** InlineCSS

**Type:** Block

**Description:** Inlines css classes for emails.

**Documentation:**
```
<p>Usage:</p>
<pre>{[ InlineCSS ]}
&lt;style&gt;
    .test {
        color: red;
    }
&lt;/style&gt;
&lt;p class="test"&gt;Hello World&lt;/p&gt;
{[ endInlineCSS ]}</pre>
```

**Shortcode Markup:**
```
{% capture realBlockContent %}{{ blockContent }}{% endcapture %}
{% execute %}
    string html = System.Uri.UnescapeDataString({{ realBlockContent | EscapeDataString | ToJSON }});
    try
    {
        var result = PreMailer.Net.PreMailer.MoveCssInline( html, false, ".ignore" );
        return result.Html;
    }
    catch
    {
        return html;
    }
{% endexecute %}
```

**Enabled Lava Commands:** Execute
