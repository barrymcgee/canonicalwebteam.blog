# Ubuntu blog extension

This extension allows you to add a simple frontend section to your flask app. All the articles
are pulled from the (Ubuntu blog)[https://admin.insights.ubuntu.com] WordPress API using JSON.

This extension provides a blueprint with 3 routes:

- "/": that returns the list of articles
- "/<slug>": the article page
- "/feed": provides a RSS feed for the page.

## How to install

To install this extension as a requirement in your project, you can use PIP;

```bash
pip install canonicalwebteam.blog
```

See also the documentation for (pip install)[https://pip.pypa.io/en/stable/reference/pip_install/].

## How to use

### Templates

The module expects HTML templates at `blog/index.html`, `blog/article.html`, `blog/blog-card.html`, `blog/archives.html`, `blog/upcoming.html` and `blog/author.html`.

An example of these templates can be found at https://github.com/canonical-websites/jp.ubuntu.com/tree/master/templates/blog.

### Flask

In your app you can then:

```python
    import canonicalwebteam.blog_extension import BlogExtension
    blog = BlogExtension(app, "Blog title", [1], "tag_name", "/url-prefix")
```

If you use the factory pattern you can also:

```python
    import canonicalwebteam.blog_extension import BlogExtension
    blog = BlogExtension()
    blog.init_app(app, "Blog title", [1], "tag_name", "/url-prefix")
```

`BlogExtension()` accepts five parameters;

- (obj) `app`
- (str) "The title of your blog"
- (arr) Array of tags from blog entries on which to filter e.g `['1234', '4321']`
- (str) Language string e.g. `"lang:en"`
- (str) "/path/to/display/your/blog"

### Django

- Add the blog module as a dependency to your Django project
- Load it at the desired path (e.g. "/blog") in the `urls.py` file

```python
from django.urls import path, include
urlpatterns = [path(r"blog/", include("canonicalwebteam.blog.django.urls"))]
```

- In your Django project settings (`settings.py`) you have to specify the following parameters:

```python
BLOG_CONFIG = {
    # the id for tags that should be fetched for this blog
    "TAGS_ID": [3184],
    # the title of the blog
    "BLOG_TITLE": "TITLE OF THE BLOG",
    # the tag name for generating a feed
    "TAG_NAME": "TAG NAME FOR GENERATING A FEED",
}
```

- Run your project and verify that the blog is displaying at the path you specified (e.g. '/blog')

#### Groups pages

- Group pages are optional and can be enabled by using the view `canonicalwebteam.blog.django.views.group`. The view takes the group slug to fetch data for and a template path to load the correct template from.
- Group pages can be filtered by category, by adding a `category=CATEGORY_NAME` query parameter to the URL (e.g. `http://localhost:8080/blog/cloud-and-server?category=articles`).

```python
from canonicalwebteam.blog.django.views import group

urlpatterns = [
    url(r"blog", include("canonicalwebteam.blog.django.urls")),
    url(
        r"blog/cloud-and-server",
        group,
        {
            "slug": "cloud-and-server",
            "template_path": "blog/cloud-and-server.html"
        }
    )
```

#### Topic pages

- Topic pages are optional as well and can be enabled by using the view `canonicalwebteam.blog.django.views.topic`. The view takes the topic slug to fetch data for and a template path to load the correct template from.

**urls.py**

```python
path(
		r"blog/topics/kubernetes",
		topic,
		{"slug": "kubernetes", "template_path": "blog/kubernetes.html"},
		name="topic",
),
```

## Development

The blog extension leverages [poetry](https://poetry.eustace.io/) for dependency management.

## Testing

All tests can be run with `poetry run pytest`.

### Regenerating Fixtures

All API calls are caught with [VCR](https://vcrpy.readthedocs.io/en/latest/) and saved as fixtures in the `fixtures` directory. If the API updates, all fixtures can easily be updated by just removing the `fixtures` directory and rerunning the tests.

To do this run `rm -rf fixtures && poetry run pytest`.
