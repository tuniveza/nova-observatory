<div align="center">

<img src="docs/media/overview.jpg" alt="Nova Observatory: totals for lines of code, size, files and languages above a grid of project decks" width="820">

# Nova Observatory

**Every project in one sky: a dashboard that measures your work on disk and shows it in screenshots and video.**

[![Static web app](https://img.shields.io/badge/static-web%20app-E34C26?logo=html5&logoColor=white)](#run-it-locally)
[![Node](https://img.shields.io/badge/Node-scanners-339933?logo=nodedotjs&logoColor=white)](#how-it-works)
[![Playwright](https://img.shields.io/badge/Playwright-capture-2EAD33?logo=playwright&logoColor=white)](#capture)
[![ffmpeg](https://img.shields.io/badge/ffmpeg-video-007808?logo=ffmpeg&logoColor=white)](#capture)
[![Nova suite](https://img.shields.io/badge/part%20of-nova--suite-B026FF)](https://github.com/tuniveza/nova-suite)

</div>

---

Nova Observatory is a dashboard of everything being worked on. It's kept true by scanning the
disk rather than by hand: one script measures every project, another opens each one in headless
Chromium and records screenshots and a short demo video, and a static page shows the lot as
project decks and a star map.

The screenshots in this README come from a catalogue of just the Nova suite.

## What it does

Each project gets a **deck** showing:

- where it lives in the folder tree
- its goal and vision, and what it does
- its size on disk, project files, lines of code and languages
- git state and last activity
- screenshots and a demo video

And across all of them:

- **Totals**: lines of code, project files, size on disk, files, languages and last activity,
  with a language bar.
- **Search, filter and sort**: by name, goal, stack or path; by category; by recent activity
  and more.
- **A detailed view** for each project, with its video, screenshots, folder tree, languages and
  facts.
- **A star map** that draws the whole folder structure, with every project as a star.
- **Six colour themes**, shared with [Nova Calendar](https://github.com/tuniveza/nova-calendar).

## Screenshots

<p align="center">
  <img src="docs/media/demo.gif" alt="Animated tour: the totals, scrolling through project decks, opening Nova Bot's detail view, and the star map" width="720">
</p>

<table>
  <tr>
    <td width="50%"><img src="docs/media/decks.jpg" alt="Project decks with video posters, stats and language bars" width="100%"><br><sub>Project decks.</sub></td>
    <td width="50%"><img src="docs/media/project-detail.jpg" alt="Nova Calendar's detail view with its video, goal, vision and features" width="100%"><br><sub>A project up close.</sub></td>
  </tr>
  <tr>
    <td colspan="2" align="center"><img src="docs/media/folder-tree.jpg" alt="The folder tree, languages and facts for a project" width="70%"><br><sub>Folder tree, languages and facts.</sub></td>
  </tr>
</table>

## How it works

```mermaid
flowchart LR
    CFG[projects.config.json<br/>the catalogue] --> SCAN[scan.mjs]
    CFG --> CAP[capture.mjs]
    DISK[(your projects<br/>on disk)] --> SCAN
    DISK --> CAP
    CAP -->|Playwright + ffmpeg| MEDIA[media/&lt;id&gt;/<br/>shots + video]
    MEDIA --> SCAN
    SCAN --> DATA[data/projects.js<br/>data/projects.json]
    DATA --> PAGE[index.html<br/>js/observatory.js]
    MEDIA --> PAGE
```

| File | Job |
|---|---|
| `projects.config.json` | Your catalogue. It holds what a scan can't know: each project's name, goal, vision, features, category, theme, and how to show it. **Add a project here.** Git ignores it; start from `projects.config.example.json`. |
| `scan.mjs` | Measures each project on disk. Writes `data/projects.json` and `data/projects.js`. |
| `capture.mjs` | Drives headless Chromium (Playwright) and ffmpeg. Writes into `media/<id>/`. |
| `index.html`, `css/`, `js/observatory.js` | The page. |
| `js/themes.js`, `js/cosmos.js`, `js/backdrop.js` | Shared with Nova Calendar: the colour themes, the generative space covers and the star backdrop. |

**What counts as what in a scan:**

- **On disk** is everything, from `du`.
- **Project files** and **lines of code** leave out:
  - dependencies and build output: `node_modules`, `build*`, `dist`, `.gradle`, `.godot` and so on
  - vendored third-party code: `lib`, `deps`, `glad`, `Libs`, `vendor`, `thirdparty`
  - anything listed under `exclude` in the catalogue, such as copied-in SDL or cglm headers
- Lines are counted only for code. Markdown, JSON, XML and config files don't count.

### Capture

**Capture modes** (`capture.mode` in the catalogue):

- `page` opens the project in Chromium and plays a short scripted demo, recording the screen.
  The project can be opened as a file, from a local static server (`serve`, with optional
  `mounts`), from the project's own Vite (`vite: true`) or from a `url`. The demo scripts are in
  `capture.mjs` under `SCRIPTS`.
- `images` turns pictures already in the project into a Ken Burns slideshow video.
- `none` shows a generated cosmic cover instead.

**Safety:** during capture, every request that leaves the machine is blocked, except Google
Fonts, and every non-GET request is blocked too. A demo can type into a chat widget, but nothing
can reach a live service.

**Private projects:** projects marked `"private": true` are skipped unless you pass
`--include-private`. Use it for anything whose pages can show personal data (Nova Agent's
visualizer can show client names, so it's marked private in the example).

## Run it locally

```sh
git clone https://github.com/tuniveza/nova-observatory.git
cd nova-observatory
cp projects.config.example.json projects.config.json   # then point it at your projects
npm run scan                                            # a few seconds
npm run serve                                           # http://localhost:4610
```

Or open `index.html` straight from `file://`; it works without a server.

To refresh it:

```sh
npm run scan      # numbers only: sizes, lines, languages, git, folder trees
npm run capture   # screenshots + demo videos (a couple of minutes)
npm run build     # scan, capture, scan again
node capture.mjs nova-calendar nova-bot   # re-capture just some projects
```

Without a `projects.config.json`, both scripts fall back to the example catalogue and say so.
The example assumes the Nova suite repos are cloned side by side (`../nova-bot`,
`../nova-calendar` and so on); relative paths are resolved from the folder you run the scripts in.

**Needs:** Node 18 or newer. For capture, also `npm install` (Playwright is an optional
dependency), a Chromium (the system one at `/usr/bin/chromium` is used if present, otherwise run
`npx playwright install chromium`) and `ffmpeg` on your `PATH`.

## Configuration

There are no secrets or API keys. Everything lives in the catalogue:

| Key | What it's for |
|---|---|
| `root` | The top of your folder tree, for the star map |
| `projects[].id`, `name`, `tagline`, `category`, `theme`, `kind`, `stack`, `status` | How the deck looks and where it's filed |
| `projects[].path` | Where the project lives on disk |
| `projects[].goal`, `vision`, `features`, `run` | The words on the deck |
| `projects[].exclude` | Folders to leave out of the counts |
| `projects[].capture` | `mode`, `file` / `serve` / `mounts` / `vite` / `url`, `script`, `images`, `private` |
| `projects[].group`, `members` | A deck that gathers several small projects |

**What stays private.** Your catalogue, `data/` and `media/` describe every project on your disk
(names, paths, folder trees, git state, screenshots and videos), so `.gitignore` keeps them out
of the repository. Only the code, the example catalogue and the README images are published.

## Tests

There's no automated test suite. `npm run scan` prints a line per project with its line count
and timing, and `npm run capture` reports how many projects were captured, skipped or failed,
which is the quickest health check.

## Project layout

```
index.html                    the page
css/observatory.css           the styling
js/observatory.js             decks, detail view, star map, search, filter and sort
js/themes.js, cosmos.js, backdrop.js   shared with Nova Calendar
assets/sigil.svg              the Nova sigil
scan.mjs                      measures every project
capture.mjs                   screenshots and videos (Playwright + ffmpeg)
projects.config.example.json  example catalogue (copy to projects.config.json)
data/, media/                 generated; ignored by git
docs/media/                   README images
```

## Part of the Nova suite

| Project | What it is |
|---|---|
| [nova-suite](https://github.com/tuniveza/nova-suite) | The Nova suite: an overview of every project |
| [nova-bot](https://github.com/tuniveza/nova-bot) | The website chat assistant, booking card and Nova Hub |
| [nova-agent](https://github.com/tuniveza/nova-agent) | Browser helper that does jobs in Acuity's admin pages |
| [nova-club](https://github.com/tuniveza/nova-club) | Members' Android app that shows the studio's busy times |
| [nova-calendar](https://github.com/tuniveza/nova-calendar) | A cosmic calendar of note cards and day cards |
| [nova-notes](https://github.com/tuniveza/nova-notes) | Nova Notes (in progress) |
| **[nova-observatory](https://github.com/tuniveza/nova-observatory)** | This repo: a dashboard of every project, with screenshots and video |

## Licence

All rights reserved — Novacane Studios.
