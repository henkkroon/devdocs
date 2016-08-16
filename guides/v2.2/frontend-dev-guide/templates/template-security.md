---
layout: default
group: fedg
subgroup: C_Templates
title: Templates XSS security
menu_title: Templates XSS security
menu_order: 5
version: 2.2
github_link: frontend-dev-guide/templates/template-security.md
---

<h2>Security measures against XSS attacks</h2>

To prevent <a href="https://en.wikipedia.org/wiki/Cross-site_scripting">XSS</a> issues Magento recommends the following rules of escaping HTML content in templates:

* If a method indicates that the contents are escaped, do not escape: `getTitleHtml()`, `getHtmlTitle()` (the title is ready for the HTML output)

* Escape data using the `$block->escapeHtml()`,  `$block->escapeHtmlAttr()`,  `$block->escapeUrl()`, and `$block->escapeJs()` methods.

* Type casting and php function `count()` don't need escaping  (for example `echo (int)$var`, `echo (bool)$var`, `echo count($var)`)

* Output in single quotes doesn't need escaping (for example `echo 'some text'`)

* Output in double quotes without variables doesn't need escaping (for example `echo "some text"`)

* Otherwise, escape the data using the `$block->escapeHtml()` method

The following code sample illustrates the XSS-safe output in templates:

{% highlight php %}
<?php echo $block->getTitleHtml() ?>
<?php echo $block->getHtmlTitle() ?>
<?php echo $block->escapeHtml($block->getTitle()) ?>
<h1><?php echo (int)$block->getId() ?></h1>
<?php echo count($var); ?>
<?php echo 'some text' ?>
<?php echo "some text" ?>
<a href="<?php echo $block->escapeUrl($block->getUrl()) ?>"><?php echo $block->getAnchorTextHtml() ?
></a>
{% endhighlight %}

#### Escape functions for templates

For the following output cases, use the specified function to generate XSS-safe output.

**Case:** JSON output\\
**Function:** No function needed for JSON output.


{% highlight html %}
  <button class="action" data-post='<?php /* @noEscape */ echo $postData ?>' />
{% endhighlight %}


**Case:** String output that should not contain HTML\\
**Function:** `escapeHtml`


{% highlight html %}
  <span class="label"><?php echo $block->escapeHtml($block->getLabel()) ?></span>
{% endhighlight %}


**Case:** URL output\\
**Function:** `escapeUrl`


{% highlight html %}
  <a href="<?php echo $block->escapeUrl($block->getCategoryUrl()) ?>">Some Link</a>
  <script>
    var categoryUrl = '<?php echo $block->escapeUrl($block->getCategoryUrl()) ?>';
  </script>
{% endhighlight %}


**Case:** Strings inside JavaScript\\
**Function:** `escapeJs`

{% highlight html %}
  <script>
    var a = '<?php echo $block->escapeJs($block->getString()) ?>';
  </script>
 
  <script>
    var field<?php echo $block->escapeJs($block->getFieldNamePostfix()) ?> = window.document.getElementById('my-element');
  </script>
{% endhighlight %}


**Case:** Strings inside HTML attributes\\
**Function:** `escapeHtmlAttr`


{% highlight html %}
  <span class="<?php echo $block->escapeHtmlAttr($block->getSpanClass()) ?>">Some Text</span>
  <input name="field" value="<?php echo $block->escapeHtmlAttr($block->getFieldValue()) ?>" />
{% endhighlight %}

<h4>Static Test</h4>

To improve security against XSS injections, a static test `XssPhtmlTemplateTest.php` is added to `dev\tests\static\testsuite\Magento\Test\Php\`.

This static test finds all echo calls in PHTML-templates and determines if it is properly escaped or not.

It covers the following cases:

* `/* @noEscape */` before output. Output doesn't require escaping. Test is green.

* Methods which contain `"html"` in their names (for example `echo $object->{suffix}Html{postfix}()`). Data is ready for the HTML output. Test is green.

* AbstractBlock methods `escapeHtml`, `escapeHtmlAttr`, `escapeUrl`, `escapeJs` are allowed. Test is green.

* Type casting and php function `count()` are allowed (for example `echo (int)$var`, `(bool)$var`, `count($var)`). Test is green.

* Output in single quotes (for example `echo 'some text'`). Test is green.

* Output in double quotes without variables (for example `echo "some text"`). Test is green.

* Other of previously mentioned. Output is not escaped. Test is red.