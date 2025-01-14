---
layout: default
title: 9. Versioning
nav_order: 9
---

# Versioning

---

As your project evolves and new features are added, you may be supporting different versions of
your code. For this reason, it is a good idea to keep an archive of different versions of your
documentation, so your users can refer to it depending on the software version they are using.

Let's see how you can achieve that.

Before you begin, make sure you are on the `main` branch in your project root and you pull the
latest changes we merged into our remote.

```sh
git checkout main
git pull
```

## Using `sphinx-multiversion`

We will be using [`sphinx-multiversion`](https://holzhaus.github.io/sphinx-multiversion/master/), a
popular extension that makes versioning your documentation easy. This will replace the traditional
`sphinx-build` command (which was executed under the hood when we ran `make html`!)

In your virtual environment, install:

```py
pip install sphinx-multiversion
```

and add the extension to `conf.py`:

```py
extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.githubpages",
    "sphinx.ext.napoleon",
    "sphinx_multiversion",
]
```

Next, to render a list of links to past versions on your site's sidebar, you'll need to add an HTML
template. Let's create a file `docs/_templates/sidebar/versions.html` that will display a list of
versions (git tags and branches):

{% highlight html %}
{% raw %}
{% if versions %}
<div class="sidebar-tree">
  <p class="caption" role="heading"><span class="caption-text">{{ _('Branches') }}</span></p>
  <ul>
    {%- for item in versions.branches %}
    <li class="toctree-l1"><a class="reference" href="{{ item.url }}">{{ item.name }}</a></li>
    {%- endfor %}
  </ul>
  <p class="caption" role="heading"><span class="caption-text">{{ _('Tags') }}</span></p>
  <ul>
    {%- for item in versions.tags %}
    <li class="toctree-l1"><a class="reference" href="{{ item.url }}">{{ item.name }}</a></li>
    {%- endfor %}
  </ul>
</div>
{% endif %}
{% endraw %}
{% endhighlight %}

Next, we need to point Sphinx to use this file. In `conf.py`, add the following:

```py
templates_path = [
    "_templates",
]
html_sidebars = {
    "**": [
        "sidebar/brand.html",
        "sidebar/search.html",
        "sidebar/scroll-start.html",
        "sidebar/navigation.html",
        "sidebar/versions.html",
        "sidebar/scroll-end.html",
    ],
}
```

We've only added a `versions.html` to `_templates/sidebar/` folder. So where are the other `html`
files coming from? They are the default `sidebar` components that ship with `furo`: namely: the
website title, the search bar, and the content tree. Now that we are overriding the html_sidebar
contents, we need to explicitly tell `furo` we still want those components in our sidebar in
addition to our custom `versions` section. `scroll-start.html` and `scroll-end.html` determine the
scrollable section of the sidebar.

Finally, we are ready to test this command locally. From the `docs/` dir, run:

```sh
cd docs
make clean
rm -rf _build/html/*    # wipe out the current html folder contents
sphinx-multiversion . ./_build/html
```

You will notice that now, in your `_build/html` folder, there is a subfolder corresponding to each
branch and tag you have in your repo. For example, there's a `main` folder and an `update_docs`
folder in which their corresponding output `HTML` files live. As you create more branches and tags,
more output `HTML` folders will be created for them.

Let's commit the changes we have done to support `sphinx-multiversion` to our `main` branch:

```sh
# from the project root
git add docs/conf.py docs/_templates/sidebar/versions.html
git commit -m "Add support for sphinx-multiversion extension"
git push origin main
```

### Generating different versions for your docs

Let's test this out. Make a small change in `index.rst`.

```
Welcome to The Office's documentation!
======================================

Explore The Office documentation by browsing the different modules.
```

Commit it to `main` and tag it:

```sh
git checkout main
git add docs
git commit -m "update docs"
git push origin main
git tag -a v0.0.1 -m "first release"
git push origin v0.0.1
```

Now re-run `sphinx-multiversion`:

```sh
sphinx-multiversion docs ./docs/_build/html
```

You should now see that a new HTML folder was generated: `./docs/_build/html/v0.0.1`. Preview all
three `index.html` locally: `./docs/_build/html/main/index.html`,
`./docs/_build/html/v0.0.1/index.html`, and `./docs/_build/html/update_docs/index.html`. As you'd
expect, the modification will appear in the `main` and the tag `v0.0.1`'s `index.html`, but not in
`update_docs`.

You'll also notice the new sidebar section we added titled `Versions`, with links to all available
versions of your documentation.

Finally, let's push the latest contents of the `html/` folder to `gh-pages`:

```sh
cd ./docs/_build/html
git branch # ensure you are on the gh-pages branch
git add .
git commit -m "versioning support"
git push -f origin gh-pages
```

Once the `pages-build-deployment` workflow has completed, refresh your GitHub Pages URL. You'll
notice that you'll get a `404 File not found` error. Why is that?

### Choosing a default version

GitHub Pages looks for an `index.html` file at the root level of the `gh-pages` branch. Since we
have been using `sphinx-multiversion` we're storing `index.html` files under their respective git
branch or tag folders now and GitHub could not find a page to load.

Modify your GitHub Pages URL to append a branch or a tag name name:

`https://<username>.github.io/the-office/main`

You'll be redirected to that version's index page. Of course, this is not convenient. We want to
automatically redirect the user to the latest version. Say, the version on the `main` branch. To do
that, we need to add an new file `index.html` at the root of the `gh-pages` branch that has this
redirection logic (see the
[docs](https://holzhaus.github.io/sphinx-multiversion/master/github_pages.html?highlight=meta%20http%20equiv%20refresh#redirecting-from-the-document-root)).
First, we create a new file:

```sh
cd ./docs/_build/html
touch index.html
```

Now, open up this file, and add the following logic:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Redirecting to main branch</title>
    <meta charset="utf-8" />
    <meta http-equiv="refresh" content="0; url=./main/index.html" />
    <link rel="canonical" href="https://<username>.github.io/the-office/main/index.html" />
  </head>
</html>
```

Let's commit and push it to the `gh-pages` branch:

```sh
# from the docs/_build/html dir
git add index.html
git commit -m "Add index.html to redirect to main branch"
git push origin gh-pages
```

{: .hint }
🙌 You have now reached the
[`9-versioning`](https://github.com/aelsayed95/the-office/tree/9-versioning) part of the tutorial.
If not, check-out that branch and
[`9-gh-pages`](https://github.com/aelsayed95/the-office/tree/9-gh-pages) branch for `gh-pages` and
continue from there.

<br />
[Previous: Automating documentation updates](./8-automating-documentation-updates.md){: .btn .float-left}
[Next: Automating versioning](./10-automating-versioning.md){: .btn .btn-purple .float-right}
<br />
