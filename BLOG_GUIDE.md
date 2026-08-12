# Blog authoring and maintenance guide

This repository is a Jekyll blog intended for GitHub Pages. You write the content in Markdown, Jekyll combines it with the HTML layouts and CSS, and the generated website is written to `_site/`.

Do not edit `_site/` directly. It is generated output and is replaced every time Jekyll builds the site.

## How the site works

When Jekyll builds the project, it performs these steps:

1. Reads the global site settings from `_config.yml`.
2. Finds Markdown pages such as `index.md`, `about.md`, and files in `_posts/`.
3. Reads the YAML front matter at the top of each Markdown file.
4. Passes the Markdown content into the selected layout in `_layouts/`.
5. Inserts the shared `<head>` markup from `_includes/head.html`.
6. Compiles `assets/main.scss` into `/assets/main.css`.
7. Creates the final static HTML website in `_site/`.
8. Generates `/feed.xml` through the `jekyll-feed` plugin.

The generated site contains only static HTML and CSS. It does not require a database or server-side application.

## Project structure

| Path | Purpose |
| --- | --- |
| `_config.yml` | Site title, description, URL, email, social profiles, and Jekyll settings. |
| `_posts/` | Published blog posts. Each post is a Markdown file. |
| `_layouts/default.html` | Shared header and footer used by the home and standard pages. |
| `_layouts/home.html` | Builds the automatic list of posts on the homepage. |
| `_layouts/page.html` | Displays standard pages such as About. |
| `_layouts/post.html` | Displays individual articles using the distraction-free post design. |
| `_includes/head.html` | Page metadata, Google Fonts, stylesheet, and RSS feed links. |
| `assets/main.scss` | All site styling, including desktop and mobile rules. |
| `index.md` | Homepage entry point. The post list itself comes from the home layout. |
| `about.md` | Content for the About page. |
| `pages/` | Figma reference images; this directory is excluded from the generated site. |
| `_site/` | Generated output. Never edit or commit it. |

## Change the site identity

Edit `_config.yml` to replace the placeholder details:

```yaml
title: "My Dev Blog"
tagline: "The blog at the corner of the internet"
email: "dev@example.com"
description: "A short description of the blog."
url: "https://your-username.github.io"
baseurl: ""

github_username: your-github-name
telegram_username: your-telegram-name
linkedin_url: "https://www.linkedin.com/in/your-linkedin-name"
```

The values are used in the browser metadata, header, footer, feed, and social links. Restart the local Jekyll server after editing `_config.yml`; configuration changes are not always picked up by live reload.

For a user or organization site named `your-username.github.io`, leave `baseurl` empty. If the site is published from a project repository such as `your-username.github.io/project-name`, set it to:

```yaml
baseurl: "/project-name"
```

## Create a blog post

Create a Markdown file inside `_posts/`. Its name must use this format:

```text
YYYY-MM-DD-short-descriptive-slug.md
```

For example:

```text
_posts/2026-09-03-designing-a-small-api.md
```

Start the file with YAML front matter, followed by the article in Markdown:

````markdown
---
layout: post
title: "Designing a small and dependable API"
excerpt: "A short summary shown beneath the title on the homepage and used in page metadata."
date: 2026-09-03 09:00:00 +0300
---

Opening paragraphs can be written normally. Leave a blank line between paragraphs.

## A section heading

Add the rest of the article here.

### A smaller heading

- Lists use standard Markdown.
- Links look like [this](https://example.com).
- Inline code uses backticks.

```js
const message = "Fenced code blocks work too";
console.log(message);
```
````

The front matter fields mean:

- `layout`: Use `post` for an article.
- `title`: The article title displayed on both the homepage and post page.
- `excerpt`: The summary displayed on the homepage. Keep it to roughly one or two sentences.
- `date`: Controls publication time and homepage ordering. Include a UTC offset, such as `+0300` for East Africa Time or `+0000` for UTC.

Keep the date in the filename and the `date` value consistent. The filename also supplies the post slug used in its URL. Posts are displayed automatically on the homepage with the newest post first; no change to `index.md` is required.

Jekyll normally hides posts dated in the future until their publication date.

## Write a draft without publishing it

For unfinished work, create an `_drafts/` directory and add a file without a date prefix:

```text
_drafts/designing-a-small-api.md
```

Use the same front matter as a normal post. Preview drafts locally with:

```bash
bundle exec jekyll serve --drafts --livereload
```

When the article is ready, move it into `_posts/` and add the publication date to its filename.

## Edit the About page

Open `about.md` and edit everything below its closing `---` line:

```markdown
---
layout: page
title: "About"
permalink: /about/
---

Write the About content here.
```

The front matter should remain in place. `permalink: /about/` keeps the page URL stable.

## Add another standard page

Create a Markdown file in the repository root. A Contact page could be written as `contact.md`:

```markdown
---
layout: page
title: "Contact"
permalink: /contact/
---

You can contact me at [dev@example.com](mailto:dev@example.com).
```

Jekyll will create `/contact/index.html`, which is available at `/contact/`.

Pages are not added to the navigation automatically. To add Contact to the header, edit the `<nav class="site-nav">` section in `_layouts/default.html`:

```liquid
<a class="nav-link{% if page.url == '/contact/' %} is-active{% endif %}"
   href="{{ '/contact/' | relative_url }}">Contact</a>
```

## Add images and downloadable files

Create an appropriate folder inside `assets/`, for example:

```text
assets/images/my-diagram.png
assets/files/example.pdf
```

Use an image in Markdown like this:

```liquid
![Diagram explaining the API]({{ '/assets/images/my-diagram.png' | relative_url }})
```

Link to a downloadable file like this:

```liquid
[Download the example PDF]({{ '/assets/files/example.pdf' | relative_url }})
```

Use descriptive alternative text for images. Prefer compressed WebP, AVIF, JPEG, or PNG files so pages remain fast.

## Link between posts and pages

For a normal site page, use `relative_url` so the link continues to work if `baseurl` changes:

```liquid
[Read the About page]({{ '/about/' | relative_url }})
```

For a post, Jekyll's `post_url` tag prevents links from breaking if the site's URL settings change:

```liquid
[Read the CSS article]({% post_url 2026-02-12-css-over-heavy-visual-frameworks %})
```

The value passed to `post_url` is the post filename without `.md`.

## Customize the design

All visual rules live in `assets/main.scss`. The most useful values are near the top:

```scss
:root {
  --ink: #111;
  --excerpt: #444;
  --muted: #888;
  --link: #06f;
  --rule: #e5e5e5;
  --content-width: 768px;
  --page-gutter: 24px;
}
```

The Google Font request is in `_includes/head.html`. Headings use Libre Baskerville; body text uses Source Sans 3.

The home and About pages use `_layouts/default.html`, which provides the header and footer. Post pages intentionally use their own standalone layout to match the supplied Figma post design.

## Preview the blog locally

Ruby and Bundler must be installed. From the repository root, install the dependencies:

```bash
bundle install
```

Start the development server:

```bash
bundle exec jekyll serve --livereload
```

Open `http://127.0.0.1:4000/` in a browser. Most content and style changes reload automatically.

To perform a production-style build without starting a server:

```bash
bundle exec jekyll build
```

The finished files will be written to `_site/`. A successful build is the quickest check for invalid front matter, Liquid, or configuration syntax.

## Publish through GitHub Pages

For a repository named `your-username.github.io`:

1. Commit the source files, but not `_site/`.
2. Push the repository to GitHub.
3. In the repository settings, open **Pages**.
4. Configure Pages to deploy the appropriate branch, normally `main`, from the repository root, or use GitHub's Jekyll Pages workflow if the repository is configured for Actions.
5. Visit `https://your-username.github.io` after the Pages build completes.

A typical update is:

```bash
git add _posts/2026-09-03-designing-a-small-api.md
git commit -m "Add API design article"
git push
```

## Publishing checklist

Before publishing a new article:

- Confirm the filename begins with a valid `YYYY-MM-DD` date.
- Confirm the front matter begins and ends with `---`.
- Use `layout: post`.
- Add a title, excerpt, and date with timezone.
- Check headings follow a sensible hierarchy: the layout supplies the page title, so article sections should normally begin with `##`.
- Preview the post at desktop and mobile widths.
- Check links, image paths, spelling, and code samples.
- Run `bundle exec jekyll build` before pushing.

## Common problems

### A post does not appear

Check that it is inside `_posts/`, that the filename contains a valid date, and that the date is not in the future. Also verify that the YAML front matter is enclosed by `---` lines.

### A post appears in the wrong order

The `date` value in the post's front matter determines its position. Newer dates appear first.

### A change to `_config.yml` is missing

Stop and restart `bundle exec jekyll serve`. Configuration changes often require a restart.

### A link works locally but fails on GitHub Pages

Use Jekyll's `relative_url` filter for site paths and ensure `url` and `baseurl` are correct in `_config.yml`.

### Styling changes do not appear

Edit `assets/main.scss`, not the generated `_site/assets/main.css`. If necessary, stop the development server, delete only the generated `_site/` directory, and rebuild.
