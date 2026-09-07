# openshift-docs-docusaurus

OpenShift's documentation as a **Docusaurus 3** site, generated from
the Dogsbay MD in [openshift-docs-markdown](https://github.com/dogsbay/openshift-docs-markdown)
and published to GitHub Pages.

This branch (`main`) holds only the workflow. **The exported project lives
on version branches**, named after the source branch:

| Branch | Source | Published |
|---|---|---|
| [`enterprise-4.22`](../../tree/enterprise-4.22) | [openshift-docs-markdown@enterprise-4.22](https://github.com/dogsbay/openshift-docs-markdown/tree/enterprise-4.22) | https://dogsbay.github.io/openshift-docs-docusaurus/ |

## What this shows

One canonical format, three static-site generators. The same Dogsbay MD is
published as

- an Astro site — [openshift-docs-markdown](https://github.com/dogsbay/openshift-docs-markdown) (`site/`)
- an MkDocs site — [openshift-docs-mkdocs](https://github.com/dogsbay/openshift-docs-mkdocs)
- **this Docusaurus site**

and because all three are `dogsbay site build --to <format>` — one pipeline
pass (attributes, conditionals and includes resolved, `routablesFrom: nav`
applied), then a serializer — they are built from an identical page set and
can be compared page for page.

## How it runs

One workflow, **Build Docusaurus site**, run on dispatch and on Mondays 07:00 UTC
(two hours after the markdown repo's sync, though scheduled runs can slip). It takes the source branch as an
input and:

1. sparse-checks-out `markdown/` + `dogsbay.config.yml` from the source branch;
2. runs the export:

   ```
   dogsbay site build source --to docusaurus --out out \
     --site-url https://dogsbay.github.io/openshift-docs-docusaurus
   ```

3. runs `npm install && docusaurus build` (10–15 minutes for 1,800
   pages on a 4-core runner), refusing a run under 1,000 pages;
4. commits `docs/`, `sidebars.js`, `docusaurus.config.js`, `static/`, `src/`,
   `package.json` and the resolved `package-lock.json` (reused by the next
   build, so a dependency that moves shows up as a diff) to the version branch (the built HTML is not
   committed — Pages deploys it from the artefact);
5. deploys to GitHub Pages and frees the artefact.

The build refuses to deploy a site over 950 MB — GitHub Pages' limit is
1 GB. Pages itself is enabled once at repo creation with source "GitHub
Actions"; the workflow cannot enable it with the default token.

`--site-url` matters on a project Pages site: the exporter splits it into
Docusaurus `url` (origin) and `baseUrl` (`/openshift-docs-docusaurus/`),
because Docusaurus rejects a path inside `url` and 404s every link without
the base.

Broken links and anchors are `warn`, not `throw`: what remains (12 links,
about 350 anchors on 4.22) is the source's own, and the count is reported in
the log rather than made a gate. Definition lists go through
`remark-definition-list`, wired by the exporter's standard profile.

## Fidelity

Measured on `enterprise-4.22` against AsciiBinder's own HTML for a complex
page (bare-metal UPI network customizations): 166/166 code blocks, 117/117
admonitions, ~98.7% token retention. What remains is traced to the source:
`== Next steps` demoted to a related-links fold, `_`-prefixed ids, table
numbering. Details in the Dogsbay repo's `docs-dev/format-docusaurus.md`.
