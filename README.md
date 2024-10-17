<div align="center">

# LinkedIn API for Python

![Build](https://img.shields.io/github/actions/workflow/status/tomquirk/linkedin-api/ci.yml?label=Build&logo=github) [![Documentation](https://img.shields.io/readthedocs/linkedin-api?label=Docs)](https://linkedin-api.readthedocs.io) [![GitHub Release](https://img.shields.io/github/v/release/tomquirk/linkedin-api?label=PyPI&logo=python)](https://pypi.org/project/linkedin-api/) [![Discord](https://img.shields.io/badge/Discord-5865F2?logo=discord&logoColor=ffffff)](https://discord.gg/BHSWgrdc)

Search profiles, send messages, find jobs and more in Python. No official API access required.

<p align="center">
    <a href="https://linkedin-api.readthedocs.io">Documentation</a>
    ·
    <a href="#quick-start">Quick Start</a>
    ·
    <a href="#how-it-works">How it works</a>
</p>

</div>

<br>

<h3 align="center">Sponsors</h3>

<p align="center" dir="auto" >
  <a href="https://bit.ly/4fUyE9J" target="_blank">
    <img  height="60px" src="https://raw.githubusercontent.com/tomquirk/linkedin-api/main/docs/assets/logos/scrapin.png" alt="Scrapin">
  </a>
</p>

<br>

## Features

- ✅ No official API access required. Just use a valid LinkedIn user account.
- ✅ Direct HTTP API interface. No Selenium, Pupeteer, or other browser-based scraping methods.
- ✅ Get and search people, companies, jobs, posts
- ✅ Send and retrieve messages
- ✅ Send and accept connection requests
- ✅ Get and react to posts

And more! [Read the docs](https://linkedin-api.readthedocs.io/en/latest/api.html) for all API methods.

> [!IMPORTANT]
> This library is not officially supported by LinkedIn. Using this library might violate LinkedIn's Terms of Service. Use it at your own risk.

## Installation

> [!NOTE]
> Python >= 3.10 required

###

```bash
pip install linkedin-api
```

Or, for bleeding edge:

```bash
pip install git+https://github.com/tomquirk/linkedin-api.git
```

### Quick Start

> [!TIP]
> See all API methods on the [docs](https://linkedin-api.readthedocs.io/en/latest/api.html).

The following snippet demonstrates a few basic linkedin_api use cases:

```python
from linkedin_api import Linkedin

# Authenticate using any Linkedin user account credentials
api = Linkedin('reedhoffman@linkedin.com', '*******')

# GET a profile
profile = api.get_profile('billy-g')

# GET a profiles contact info
contact_info = api.get_profile_contact_info('billy-g')

# GET 1st degree connections of a given profile
connections = api.get_profile_connections('1234asc12304')
```

## Commercial alternatives

> This is a sponsored section

<h3>
<a href="https://www.scrapin.io/?utm_campaign=influencer&utm_source=github&utm_medium=social&utm_content=scrapin-io">
ScrapIn
</a>
</h3>

Scrape Any Data from LinkedIn, without limit with [ScrapIn API](https://bit.ly/4fUyE9J).

<details>
  <summary>Learn more</summary>
  
- Real time data (no-cache)
- Built for SaaS developers
- GDPR, CCPA, SOC2 compliant
- Interactive API documentation
- A highly stable API, backed by over 4 years of experience in data provisioning, with the added reliability of two additional data provider brands owned by the company behind ScrapIn.

Try it for free. [Get your API key now](https://bit.ly/4fUyE9J)

</details>


> End sponsored section

## Development

### Dependencies

- [`poetry`](https://python-poetry.org/docs/)
- A valid Linkedin user account (don't use your personal account, if possible)

### Development installation

1. Create a `.env` config file (use `.env.example` as a reference)
2. Install dependencies using `poetry`:

   ```bash
   poetry install
   poetry self add poetry-plugin-dotenv
   ```

### Run tests

Run all tests:

```bash
poetry run pytest
```

Run unit tests:

```bash
poetry run pytest tests/unit
```

Run E2E tests:

```bash
poetry run pytest tests/e2e
```

### Lint

```bash
poetry run black --check .
```

Or to fix:

```bash
poetry run black .
```

### Troubleshooting

#### I keep getting a `CHALLENGE`

Linkedin will throw you a curve ball in the form of a Challenge URL. We currently don't handle this, and so you're kinda screwed. We think it could be only IP-based (i.e. logging in from different location). Your best chance at resolution is to log out and log back in on your browser.

**Known reasons for Challenge** include:

- 2FA
- Rate-limit - "It looks like you’re visiting a very high number of pages on LinkedIn.". Note - n=1 experiment where this page was hit after ~900 contiguous requests in a single session (within the hour) (these included random delays between each request), as well as a bunch of testing, so who knows the actual limit.

Please add more as you come across them.

#### Search problems

- Mileage may vary when searching general keywords like "software" using the standard `search` method. They've recently added some smarts around search whereby they group results by people, company, jobs etc. if the query is general enough. Try to use an entity-specific search method (i.e. search_people) where possible.

## How it works

This project attempts to provide a simple Python interface for the LinkedIn API.

> Do you mean the [legit LinkedIn API](https://developer.linkedin.com/)?

NO! To retrieve structured data, the [LinkedIn Website](https://linkedin.com) uses a service they call **Voyager**. Voyager endpoints give us access to pretty much everything we could want from LinkedIn: profiles, companies, connections, messages, etc. - anything that you can see on linkedin.com, we can get from Voyager.

This project aims to provide complete coverage for Voyager.

[How does it work?](#deep-dive)

### Deep dive

Voyager endpoints look like this:

```text
https://www.linkedin.com/voyager/api/identity/profileView/tom-quirk
```

Or, more clearly

```text
 ___________________________________ _______________________________
|             base path             |            resource           |
https://www.linkedin.com/voyager/api /identity/profileView/tom-quirk
```

They are authenticated with a simple cookie, which we send with every request, along with a bunch of headers.

To get a cookie, we POST a given username and password (of a valid LinkedIn user account) to `https://www.linkedin.com/uas/authenticate`.

### Find new endpoints

We're looking at the LinkedIn website and we spot some data we want. What now?

The following describes the most reliable method to find relevant endpoints:

1. `view source`
1. `command-f`/search the page for some keyword in the data. This will exist inside of a `<code>` tag.
1. Scroll down to the **next adjacent element** which will be another `<code>` tag, probably with an `id` that looks something like

   ```html
   <code style="display: none" id="datalet-bpr-guid-3900675">
     {"request":"/voyager/api/identity/profiles/tom-quirk/profileView","status":200,"body":"bpr-guid-3900675"}
   </code>
   ```

The value of `request` is the url! 🤘

You can also use the `network` tab in you browsers developer tools, but you will encounter mixed results.

### How Clients query Voyager

linkedin.com uses the [Rest-li Protocol](https://linkedin.github.io/rest.li/spec/protocol) for querying data. Rest-li is an internal query language/syntax where clients (like linkedin.com) specify what data they want. It's conceptually similar to the GraphQL.

Here's an example of making a request for an organisation's `name` and `groups` (the Linkedin groups it manages):

```text
/voyager/api/organization/companies?decoration=(name,groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url))&q=universalName&universalName=linkedin
```

The "querying" happens in the `decoration` parameter, which looks like the following:

```text
(
    name,
    groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url)
)
```

Here, we request an organisation name and a list of groups, where for each group we want `largeLogo`, `groupName`, and so on.

Different endpoints use different parameters (and perhaps even different syntaxes) to specify these queries. Notice that the above query had a parameter `q` whose value was `universalName`; the query was then specified with the `decoration` parameter.

In contrast, the `/search/cluster` endpoint uses `q=guided`, and specifies its query with the `guided` parameter, whose value is something like

```text
List(v->PEOPLE)
```

It could be possible to document (and implement a nice interface for) this query language - as we add more endpoints to this project, I'm sure it will become more clear if such a thing would be possible (and if it's worth it).

### Release a new version

1. Bump `version` in `pyproject.toml`
1. `poetry build`
1. `poetry publish`
1. Draft release notes in GitHub.

## Disclaimer

This library is not endorsed or supported by LinkedIn. It is an unofficial library intended for educational purposes and personal use only. By using this library, you agree to not hold the author or contributors responsible for any consequences resulting from its usage.
