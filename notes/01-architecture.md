# 01 — Survey of the old site, and the technology for the new one

*Status: **proposal**, for discussion with Jasper. Drafted 2026-11-26.*

This is the first of a series of notes, one per stage of the website rebuild:
architecture (this note), then structure, then content. Each settled point goes
into [`DECISIONS.md`](../DECISIONS.md).

## Why rebuild

simulistics.com is the home of Simile. It needs replacing for two reasons:

1. **The material is changing.** The new browser-based Simile (the SPA, in
   [robert-mue/simile](https://github.com/robert-mue/simile), on the
   [sienna](https://github.com/robert-mue/sienna) shell) brings new tutorials,
   sample models and technical reference.
2. **The site itself is tired.** Both its look and the software it runs on are
   dated.

The new site will be at **https://robert-mue.github.io/simulistics/**, built
from this repository. Later, the simulistics.com domain could point to it.

## The old site

It runs on **Drupal 7**, which has been out of support since January 2025. We
surveyed it by crawling about 580 pages, three links deep, on 2026-11-26.

The main menu has five sections: Introduction · Products and services ·
Resources and support · Community · About us.

| Area | What is there | Assessment |
|---|---|---|
| Introduction | Home; "Simile at a glance"; guided tour (17 pages on modelling approaches: system dynamics, discrete event, individual-based, spatial, modular, …); model gallery (8 pages); applications (4); case studies (FLORES, ModMED, MRSA, Pastoral Livelihoods) | Good material in a dated style. The tour, the gallery and "Modelling approaches" overlap a lot. |
| Products | Download page, licence, services, "What's in a name", acknowledgements | Partly out of date, because Simile is now free and open-source (GPL). |
| Help (`/help/`) | About 130 pages: diagram elements, working with diagrams, equations, built-in functions, submodels, running models, external data, files, "What's new" (up to 7.0) | The largest and most valuable block. |
| Tutorials | Predator–prey (5 stages), Trees (3 stages), teaching resources | Superseded by the new tutorials in the simile repo. |
| Worked examples | About 26 example pages (population dynamics series, energy flow, land use, Lorenz, Hodgkin–Huxley, …), a model catalogue, a "modeltags" taxonomy | Three generations of catalogue mixed together. |
| Publications | A bibliography, with BibTeX, RIS and MARC exports | Worth keeping. |
| News / blogs | Release announcements from 5.8 to 7.5 (October 2026); a few user blogs | Keep the release history; drop the blogs. |
| Community | Forums, user groups, an issue tracker, user accounts, logins | Largely unused. |

**In short:** there is a lot of good content, buried under CMS clutter. The
same page is reachable at several addresses (`/node/N`,
`/book/export/html/N` and a readable name), there are login and social
features nobody uses, and some pages still talk about paid licences. Almost all
of it is static content that never needed a CMS.

**The old help.** There are two copies:

- **The Windows help file** (`/files/help/simile.chm`): clean HTML, but only
  covers up to Simile 5.0.
- **The online `/help/` pages:** cover up to 7.0, so these are the copy to
  work from.

Much of the help describes the old desktop program's dialogs and menus, so it
will need rewriting for the SPA. The substance of the modelling language
carries over: the diagram elements, the equation language and the built-in
functions. So we treat the old help as raw material, not something to convert
wholesale.

## What the SPA gives the website

The SPA is static by design: no server, no build step. It is published at
**https://robert-mue.github.io/simile/** from its `release` branch. That is
the same origin as the new site, so the site can load the app's models and
scripts without any cross-origin problems.

It offers the website two things:

1. **Opening the full app with a model loaded**, through the address:
   `…/simile/?open=<model URL>` (with optional `&parameters=…`). There is
   also `?gallery=…` for the sample-models gallery.
2. **Embedding individual widgets in a page**, through a single script,
   `embed.js`:

   ```html
   <script src="https://robert-mue.github.io/simile/embed.js"></script>
   <div data-simile="diagram"  data-model="predator-prey"></div>
   <div data-simile="controls" data-model="predator-prey"></div>
   <div data-simile="plotter"  data-model="predator-prey" data-show="rabbits"></div>
   <div data-simile="sliders"  data-model="predator-prey"></div>
   <script src="predator-prey.js"></script>
   <script>Simile.page();</script>
   ```

   Working examples are in `simile/pages/predator-prey/` and `pages/trees/`.

The second is the big opportunity. A page about a model can show **the live
model diagram, run controls and plots of its behaviour**, rather than
screenshots. A reader's changes last only as long as the browser tab.

## Proposal

### A static site, built from Markdown in this repository

There is no CMS, database or server. Pages are Markdown files in git, which
gives us:

- a full history of every change;
- changes that can be reviewed as pull requests;
- nothing to patch for security, and no logins to manage.

GitHub Actions builds the site and publishes it to GitHub Pages on every merge
to `main`.

### Generator: Astro, with Starlight for the documentation

- **[Astro](https://astro.build)** builds the site.
  - Its *content collections* give each kind of content a defined set of
    fields. Kinds include example models, tutorials, news items and
    publications. Fields for a model might be its title, tags, Simile version
    and model file.
  - The index pages are generated from those fields. That replaces the old
    model catalogue, the tag taxonomy and the bibliography module.
- **[Starlight](https://starlight.astro.build)**, Astro's documentation theme,
  serves the reference and the tutorials. It provides:
  - sidebar navigation;
  - previous/next links and a table of contents on each page;
  - dark mode;
  - search that runs entirely in the browser (Pagefind), so no search server
    is needed.
- **Live widgets** are plain HTML elements plus one script, so they drop into
  any page. The one detail is that the embed scripts must be marked
  `is:inline`, so Astro passes them through unchanged. Each model page can
  then show:
  - a live diagram;
  - run controls;
  - plots;
  - an "Open in Simile" link.
- **Each model is stored once.** The `Simile.model(…)` wrapper files that
  embedding needs are generated at build time from the models' `.json` files.

**Alternatives considered:**

- **Hugo:** very fast, but its templating is awkward.
- **MkDocs Material:** excellent for documentation, but weak for a product
  front page.
- **Jekyll:** GitHub Pages' built-in default, but dated.

Astro suits a site that combines a front page, a documentation section and
interactive model embeds.

### Where the SPA comes from

The site loads `embed.js` from the published app (`/simile/`), so every embed
always uses the current release. The app's `release` branch is already a
stable channel. Pinning a fixed copy of the app would only be needed if we
wanted the website frozen against app changes.

### Other points

- **Tutorials:** the new tutorials in the simile repo (`help/tutorials/`) are
  the definitive copies. The site presents them, either by drawing on that
  copy at build time or by linking to it. The old Drupal tutorials are retired.
- **Forums, groups and user accounts:** dropped. Use GitHub Discussions and
  Issues on the simile repo for community questions and bug reports.
- **Publications:** a single BibTeX (or YAML) file, rendered as a list.
- **News:** one Markdown file per item. The release history is carried over.
- **Moving content across:** a script converts the pages we keep from Drupal
  HTML to Markdown as a first draft, and we then edit them. We do not carry
  the old structure over wholesale.
- **Old addresses:** GitHub Pages cannot redirect on the server. If
  simulistics.com is later pointed at this site, we generate small redirect
  pages for the most-linked old addresses (`/help/…`, `/tutorials/…`,
  `/examples/…`).

## Questions for Jasper

1. Is a static site on GitHub Pages acceptable, with nothing on a server?
2. Is Astro (with Starlight) the right generator, or is there a reason to
   prefer something else?
3. Is it acceptable to drop forums and accounts in favour of GitHub
   Discussions and Issues?
4. Should the site always use the current Simile release, or pin a version?
5. What should happen to the simulistics.com domain, and when?
6. Is there anything on the old site, not mentioned above, that must survive?
