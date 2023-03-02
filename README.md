# Personal Site / Blog

Uses the PaperMod theme for Hugo.

# Notes

[hugo-PaperMod-Mod](https://github.com/arashsm79/hugo-PaperMod-Mod) could be cool for a side toc.

[hugo-PaperModX](https://github.com/reorx/hugo-PaperModX) could also be cool, but has some issues with re-styling link accents.

## Hugo stuff

New Houdini blog post

```
hugo new hou-post posts/new-post
```

New project page

```
hugo new project projects/new-project
```

Start local server
```
hugo server -D
```

Sometimes with the gh-action, the build will fail due to API rate limit stuff when grabbing all the gists. Not sure how to get around this yet (by authenticating somehow?), but retrying tends to work. If it gets annoying I could copy the python script I made for the jekyll version of the site, scrape the data locally, store as a JSON in `/data` and have it use that instead.

Using github actions to deploy on push to `main`, but keeping the **Pages** settings to source **Deploy from a branch** since the gh action builds the site in the `gh-pages` branch.

Static images can be linked to using `/image.jpg`, no need for `/static/image.jpg`

Font Aweseome icons live [here](https://github.com/FortAwesome/Font-Awesome/tree/6.x/svgs/regular)

grab with

```bash
./get_icons.sh <brands/regular/solid> <icon_name>
```

Use as a partial

```html
{{ partial "fontawesome.html" "github" }}
```

works with any SVG in the `/fontawesome` directory, even the houdini badge!

(credit to [Nick Galbreath](https://www.client9.com/using-font-awesome-icons-in-hugo/), with a little tweaking of the script)

### Attachments

Posts and projects can have "attachments" which are just extra files etc. relating to the content. Links are scraped from the front matter.

Add them in the content with the shortcode

```
{{< attachments >}}
```

Front matter example:

```toml
hipfile: "houdini/hip/jamesr_demofile.hip"
repo: "demoreponame"
attachments: ["text/hello.txt", "json/greetings.json"]
```

If `hipfile` is set in the front-matter, you can use the shortcode to jump to the hipfile attachment page anchor.

```
{{< jumpto_hipfile >}}
```
