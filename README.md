# bigfix-site-reporting-jgstew

A BigFix [Repository Site](https://help.hcl-software.com/bigfix/11.0/platform/Platform/Config/c_repository_site.html)
for reporting content: console dashboards and Web Reports that help write,
test and troubleshoot relevance. BigFix 11.0.7 or later gathers it straight
from this Git repository.

## What is in the site

| Path | What it is |
|---|---|
| `ServerFiles/relevance-debugger-console.ojo` | The Relevance Debugger console dashboard: evaluates client relevance on the console's computer (qna) and with FastQuery, and session relevance. |
| `ServerFiles/relevance-debugger-webreport.besrpt` | The Relevance Debugger for Web Reports: analyzes relevance as you type and evaluates session relevance. Web Reports picks it up from the site automatically. |
| `Fixlets/` | Fixlets, Tasks and Analyses (`.bes`), if any are added. |
| `ClientFiles/` | Files gathered by clients. |
| `site.xml` | Site relevance: the site's content does not apply to proxy agents. |
| `siteConfig.xml` | Maps the folders above to BigFix's names: `Fixlets` for `.bes` files, `ServerFiles` for files present on the server, Web Reports and console but not gathered by clients, `ClientFiles` for files clients gather. |

The Relevance Debugger is built from
[jgstew/bigfix-relevance-analyzer](https://github.com/jgstew/bigfix-relevance-analyzer)
(`tools/playground-wasm/dashboard-poc/`). The Web Report loads the analyzer
from https://www.jgstew.com/files/analyzer/ when it opens: it reads that
folder's `manifest.json` and checks every file's size and sha256 against it
before using it (in JavaScript where the browser offers no `crypto.subtle`,
as on a plain-http Web Reports server), which catches corrupted or
mismatched files. It keeps the verified copy in the browser and downloads
again only when a new analyzer release changes the manifest, so the browser
needs internet access the first time and after each release.

## Add the site to BigFix

### 1. Give the root server a Git SSH key

On the BigFix root server, open an elevated command prompt in the server's
`RepositorySiteGather\GitCredentials` folder:

```
cd "C:\Program Files (x86)\BigFix Enterprise\BES Server\RepositorySiteGather\GitCredentials"
```

Create the key pair (as of 11.0.7 these file names are required) and the
`known_hosts` file:

```
ssh-keygen -m PEM -t rsa -b 4096 -f id_rsa -N ""
ssh-keyscan github.com >> known_hosts
```

If Windows' `ssh-keyscan` fails with `choose_kex: unsupported KEX method`,
use the one from Git for Windows instead:

```
"C:\Program Files\Git\usr\bin\ssh-keyscan.exe" github.com >> known_hosts
```

Add `id_rsa.pub` as an SSH key on a GitHub account that can read this
repository.

### 2. Create the Repository Site

In the BigFix Console, go to **Tools > Create Repository Site** and enter:

- **SSH URL:** `git@github.com:jgstew/bigfix-site-reporting-jgstew.git`
- **Branch:** `main`
- a name and description for the site, if you want them

BigFix checks SSH access with the keys in `GitCredentials`, then clones the
repository. The root server's log shows `Repository site cloned successfully`,
and later `Repository site updated successfully` as it picks up new commits.

Give operators access to the site as for any custom site; the dashboard
appears under the console's Dashboards menu. Subscribing computers is only
needed for content clients evaluate (`Fixlets/`, `ClientFiles/`).

## Contributing

Run the checks before committing (`pre-commit install` runs them on every
commit):

```
pre-commit run --all-files
```

## License

[MIT](LICENSE)
