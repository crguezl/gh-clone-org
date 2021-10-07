# gh-clone-org

A github-cli extension script to clone all repositories in an organization, optionally filtering by topic or a search string. If the repository has already been cloned it will attempt to switch to the default branch and pull.

## Installation

```bash
gh extension install crguezl/gh-clone-org
```

Here is an example of output:

```
[~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ gh extension install crguezl/gh-clone-org
Clonando en '/Users/casianorodriguezleon/.local/share/gh/extensions/gh-clone-org'...
```

Observe how extensions are stored in `$HOME/.local/share/.gh/extensions`

This repo is a fork of `matt-bartel/gh-clone-org`. To install the original version:

```bash
gh extension install matt-bartel/gh-clone-org
```

## Usage

```txt
gh clone-org [-t TOPIC] [-s QUERY] [-p PATH] [-y] ORG
  ORG
    Github organization. Required if the GITHUB_ORG environment variable is not set.
  -y, --yes
    Clone without prompting for confirmation.
  -p, --path PATH
    Clone path. Default: current directory.
  -t, --topic TOPIC
    Clone repositories with this topic
  -s --search QUERY
    Clone repositories found by this search string. If this is provided '-t' will be ignored.
    Example: -s "is:public language:go"
    See: https://docs.github.com/en/github/searching-for-information-on-github/searching-on-github/searching-for-repositories
  -n, --dry-run
    Do not actually clone, just show what would be cloned
  -h, --help
    Display this message.
```

## Examples of use

```
$ gh clone-org -s iaas -n ULL-ESIT-DMSI-1920
Retrieving the list of repositories: search/repositories?q=org%3AULL-ESIT-DMSI-1920%20iaas
This would have cloned the following 6 repositories to /Users/casianorodriguezleon/campus-virtual/2021/learning/gh-learning/gh-clone-org:
ULL-ESIT-DMSI-1920/p1-t1-iaas-alu0101232812
ULL-ESIT-DMSI-1920/p1-t1-iaas-alu0100592306
ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa
ULL-ESIT-DMSI-1920/p1-t1-iaas-Wololegend
ULL-ESIT-DMSI-1920/p1-t1-iaas-crguezl
ULL-ESIT-DMSI-1920/p1-t1-iaas-lauramanzini
```

### Setting default org with GITHUB_ORG

```
$ export GITHUB_ORG=ULL-ESIT-DMSI-1920
$ gh clone-org -s fuegonella  -n
Retrieving the list of repositories: search/repositories?q=org%3AULL-ESIT-DMSI-1920%20fuegonella
This would have cloned the following 3 repositories to /Users/casianorodriguezleon/campus-virtual/2021/learning/gh-learning/gh-clone-org:
ULL-ESIT-DMSI-1920/markdown-fuegonellaa
ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa
ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa
```

## How it works 

To obtain the repos `gh-clone-org` makes a request like:

```
$ gh api --paginate /search/repositories?q=org%3AULL-ESIT-DMSI-1920%20fuegonella
{
    "total_count": 3,
    "incomplete_results": false,
    "items": [
        {
            "id": 411615845,
            "node_id": "R_kgDOGIjCZQ",
            "name": "markdown-fuegonellaa",
            "full_name": "ULL-ESIT-DMSI-1920/markdown-fuegonellaa",
            "private": false,
            "owner": { ... },
            ...
            "score": 1.0
        },
        {
            ...
            "name": "pb-gh-campus-expert-fuegonellaa",
            "full_name": "ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa",
            "private": false,
            "owner": { ... },
            ...
            "score": 1.0
        },
        {
            ...
            "full_name": "ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa",
            "private": true,
            "owner": { ...  },
            ...
            "score": 1.0
        }
    ]
}
```

### jq to the rescue

The json is very large and contains a lot of information!

We can make use of `jq` to filter the results
(See [jq](https://ull-esit-dmsi-1920.github.io/tema1-introduccion/jq)): 

```
[~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ gh api --paginate /search/repositories?q=org%3AULL-ESIT-DMSI-1920%20fuegonella | jq .items[].full_name -
"ULL-ESIT-DMSI-1920/markdown-fuegonellaa"
"ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa"
"ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa"
```

We can make very complex querys with `jq`:

```
$ gh api --paginate /search/repositories?q=org%3AULL-ESIT-DMSI-1920%20fuegonella | \
  jq '.items[] | .full_name, .private, .commits_url'
"ULL-ESIT-DMSI-1920/markdown-fuegonellaa"
false
"https://api.github.com/repos/ULL-ESIT-DMSI-1920/markdown-fuegonellaa/commits{/sha}"
"ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa"
false
"https://api.github.com/repos/ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa/commits{/sha}"
"ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa"
true
"https://api.github.com/repos/ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa/commits{/sha}"
```

Fortunately, `gh` has a `--jq` option that does the same that the former pipe example.

But first, let avoid the `gh` default paginator:

```
[~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ gh  config set pager cat
[~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ gh  config get pager
cat
```

And here it is the output of `gh` using `--jq`:

```
[~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ gh api --paginate /search/repositories?q=org%3AULL-ESIT-DMSI-1920%20fuegonella --jq .items[].full_name 
ULL-ESIT-DMSI-1920/markdown-fuegonellaa
ULL-ESIT-DMSI-1920/pb-gh-campus-expert-fuegonellaa
ULL-ESIT-DMSI-1920/p1-t1-iaas-fuegonellaa
```

### What happens if one of the repos already exists

If the repo already exists the script attempts to pull the last version of the default branch

```
~/campus-virtual/2021/learning/gh-learning/gh-clone-org(master)]$ ./gh clone-org -s markdown-fuegonellaa -y -p tmp/
Retrieving the list of repositories: QUERY=search/repositories?q=org%3AULL-ESIT-DMSI-1920%20markdown-fuegonellaa JQ_QUERY=.items[].full_name
Cloning 1 repositories to tmp/...
ULL-ESIT-DMSI-1920/markdown-fuegonellaa
ULL-ESIT-DMSI-1920
markdown-fuegonellaa
'markdown-fuegonellaa' already exists, attempting to checkout the default branch and pull
Ya en 'main'
Tu rama está actualizada con 'origin/main'.
Ya está actualizado.
```