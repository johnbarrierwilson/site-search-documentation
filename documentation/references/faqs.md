---
title: "Frequently Asked Questions 🔮"
slug: "/faqs"
---

### Where can I learn more?

The [Advanced README](./ADVANCED.md) contains several useful guides. :sunglasses:

### Is Search UI only for React?

Nope. Search UI is "headless".

You can write support for it into any JavaScript framework. You can even use vanilla JavaScript.

[Read the Headless Core Guide](./ADVANCED.md#headless-core) for more information, or check out the [Vue.js Example](https://github.com/elastic/vue-search-ui-demo).

### Can I use my own styles?

You can!

Read the [Custom Styles and Layout Guide](./ADVANCED.md#custom-styles-and-layout) to learn more, or check out the [Seattle Indies Expo Demo](https://github.com/elastic/seattle-indies-expo-search).

### Can I build my own Components?

Yes! Absolutely.

Check out the [Build Your Own Component Guide](./ADVANCED.md#build-your-own-component).

### Does Search UI only work with App Search?

Nope! We do have two first party connectors: Site Search and App Search.

But Search UI is headless. You can use _any_ Search API.

Read the [Connectors and Handlers Guide](./ADVANCED.md#connectors-and-handlers).

### How do I use this with Elasticsearch?

First off, we should mention that it is not recommended to make API calls directly to Elasticsearch
from a browser, as noted in the [elasticsearch-js client](https://github.com/elastic/elasticsearch-js#browser).

The safest way to interact with Elasticsearch from a browser is to make all Elasticsearch queries server-side. Or you can use [Elastic App Search](https://www.elastic.co/cloud/app-search-service?ultron=searchui-repo&blade=readme&hulk=product), which can create public, scoped API credentials and be exposed directly to a browser.

That being said, Search UI will still work with Elasticsearch (or any other API, for that matter). Read the
[Connectors and Handlers Guide](./ADVANCED.md#connectors-and-handlers) to learn more, or check out the
[Elasticsearch Example](./examples/elasticsearch).

### Where do I report issues with the Search UI?

If something is not working as expected, please open an [issue](https://github.com/elastic/search-ui/issues/new).

### Where can I go to get help?

Connect with the community and maintainers directly on [Gitter](https://gitter.im/elastic-search-ui/community).

If you are using an Elastic product as your connector, try the Elastic community...

- [Elastic App Search discuss forums](https://discuss.elastic.co/c/app-search)
- [Elastic Site Search discuss forums](https://discuss.elastic.co/c/site-search)
