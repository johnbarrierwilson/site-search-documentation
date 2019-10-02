---
title: "Adding Site Search to any CMS"
slug: "/adding-site-search-to-any-cms"
---
# Adding Site Search Search to any CMS

Content management systems are great for creating websites. But their search experiences often aren't too great...

Site Search can provide a tremendous upgrade to the basic CMS search experience.

Best of all, you can add your CMS content automatically using the [Site Search Crawler](https://swiftype.com/documentation/site-search/crawler-overview).

## Site Search Crawler [#crawler]

The Crawler is the easiest way to get started with Site Search.

It works well for most CMS-based sites.

There are a few things you can do that will make the crawler work better with your site...

* Follow the [Crawler Optimization Guide](documentation/site-search/guides/crawler-optimization) to refine your sitemaps for the most precise crawling.
* Read the [Schema Design Guide](https://swiftype.com/documentation/site-search/guides/schema-design#crawler) to learn how to create custom fields upon which you can search.

An example of a unique CMS page with custom field contents might look like so:

{% examplecode %}
  {% description %}
	Site Search meta tags using Jekyll's Liquid template syntax and variables
  {% enddescription %}
  {% code html %}
<head>
	<title>{{ page.title }} | Site Search Crawler Demo Site</title>
	<meta class="swiftype" name="title" data-type="string" content="{{ page.title }}" >
	<meta class="swiftype" name="published_at" data-type="date" content="{{ page.date | date_to_xmlschema }}">
</head>
  {% endcode %}
{% endexamplecode %}

You can also use the [`data-swiftype-index` attribute](/documentation/site-search/crawler-configuration/content-inclusion-exclusion) for content inclusions and exclusions.

An example of HTML which will include only content inside a `div` with `data-swiftype-index="true"`:

{% examplecode %}
  {% description %}
    Using data-swiftype-index="true" to index only part of a page's content
  {% enddescription %}
  {% code html %}
<html>
  <body>
    <div>
      This is navigation: <a href="/">home</a>
    </div>

    <div data-swiftype-index="true">
      This is the page's content. It will be indexed.
    </div>

    <div>
      This is a footer. &copy; 2013 YourSite.com
    </div>
  </body>
</html>
  {% endcode %}
{% endexamplecode %}

Sometimes it may be easier to exclude certain parts of the page instead.

You can do that with `data-swiftype-index="false"`:

{% examplecode %}
  {% description %}
    Using data-swiftype-index="false" to index only part of a page's content
  {% enddescription %}
  {% code html %}
<html>
  <body>
    <div data-swiftype-index="false">
      This is navigation: <a href="/">home</a>
    </div>

    <div>
      This is the page's content. It will be indexed.
    </div>

    <div data-swiftype-index="false">
      This is a footer. &copy; 2013 YourSite.com
    </div>
  </body>
</html>
  {% endcode %}
{% endexamplecode %}

The Site Search data attribute is easy to add to your CMS's template, and will help Site Search extract the most meaningful content from your site.

You can also provide our custom embed code, which can be calibrated to provide out-of-the-box search bars and result pages:

```html
<script type="text/javascript">
  (function(w,d,t,u,n,s,e){w['SwiftypeObject']=n;w[n]=w[n]||function(){
  (w[n].q=w[n].q||[]).push(arguments);};s=d.createElement(t);
  e=d.getElementsByTagName(t)[0];s.async=1;s.src=u;e.parentNode.insertBefore(s,e);
  })(window,document,'script','//s.swiftypecdn.com/install/v2/st.js','_st');

  _st('install','YOUR_INSTALL_KEY','2.0.0');
</script>
```

**For more information on how to design and then search your ideal search experience, see the [Design and Customization Guide](documentation/site-search/guides/design-and-customization).**

### Ping Site Search when content is published [#ping-site-search]

On the more advanced end, many CMSes offer a callback that can be used to "hook into" saving a piece of content.

For example:
1. WordPress has [`save_post`](http://codex.wordpress.org/Plugin_API/Action_Reference/save_post)
2. Drupal has [`hook_node_insert`](http://api.drupal.org/api/drupal/modules!node!node.api.php/function/hook_node_insert/7) and [`hook_node_update`](http://api.drupal.org/api/drupal/modules!node!node.api.php/function/hook_node_update/7)
3. Ruby on Rails models have [`after_save` callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)

You can use these callbacks to notify Site Search of new content to index.

This is optional, but will bring your search engine up-to-date quickly after publishing content.

Site Search offers an API endpoint to request the indexing of a URL.

If the URL exists in the search engine, Site Search will update its contents. If the URL does not exist in the search engine, Site Search will add it.

You need to know the your Engine Slug, the ID of your domain, and the URL you want to crawl (along with your API key) to use this API.

* Your Engine Slug can be found on your Site Search dashboard
* Your Domain ID can be found by using the `GET /api/v1/engines/YOUR_ENGINE_SLUG/domains.json` API endpoint

From the command line, the API call looks like this:

```bash
curl -X PUT 'https://api.swiftype.com/api/v1/engines/YOUR_ENGINE_SLUG/domains/YOUR_DOMAIN_ID/crawl_url.json' \
-H 'Content-Type: application/json' \
-d '{
      "auth_token": "YOUR_AUTH_TOKEN",
      "url": "http://example.com/new-page"
    }'
```

You can make the equivalent HTTP request using whichever facility your CMS provides.

If possible, it is recommended to make this API call outside of the web request cycle using a background worker.

This will prevent content updates from failing due to timeouts or request failures.

If you are using a static-file CMS like Jekyll, you can request a full recrawl of your site when you publish the site.

As above, you'll need your Engine Slug, Domain ID, and API Key to issue the recrawl API request.

An example Rake task that publishes a [Jekyll](http://jekyllrb.com/) site with rsync, then issues a recrawl API request:

{% examplecode %}
  {% description %}
    Rake task to publish a Jekyll site and request a recrawl
  {% enddescription %}
  {% code ruby %}
desc 'Build and deploy to web servers then recrawl.'
task :deploy => :jekyll do
  sh "jekyll --no-auto"
  sh "rsync --recursive --times --compress --human-readable --progress --delete _site/ deploy@yoursite.com:/var/www/blog/"
  sh "curl -XPUT -H 'Content-Length: 0' 'https://api.swiftype.com/api/v1/engines/YOUR_ENGINE_SLUG/domains/YOUR_DOMAIN_ID/recrawl.json?auth_token=YOUR_AUTH_TOKEN'"
end
  {% endcode %}
{% endexamplecode %}

Once your Site Search search engine is filled with content, you'll be able to use all of the relevance features to further perfect the quality out-of-the-box relevance:

* [Analytics](/documentation/site-search/guides/analytics): Track search performance, and drive results.
* [Synonyms](/documentation/site-search/guides/synonyms), [Result Rankings](/documentation/site-search/guides/result-rankings), and [Weights](/documentation/site-search/guides/weights) help you fine-tune search relevance.
