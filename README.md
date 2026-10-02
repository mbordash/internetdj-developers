# InternetDJ for developers

[InternetDJ](https://internetdj.co) is a free community for electronic music producers: upload tracks, get written feedback, make music in the browser, and mix on the decks.

This repository is the public home for building on InternetDJ. The site itself is closed source; everything here is meant to be used, copied, and built on.

## What's here

| | Status | |
|---|---|---|
| [Embeds and oEmbed](embed/) | **Stable** | Put an InternetDJ player on any site, or turn a pasted song link into one. |
| [Arrangement format](arrangement-format/) | **Draft 0.1** | A whole multitrack project as one JSON file: tempo, tracks, notes, clips, effects, mix. |
| Developer API v1 | Planned | Read-only access to public songs, profiles, projects and articles, with an OpenAPI spec. |
| Personal access tokens | Planned | Tokens with scopes, created and revoked in your settings. |
| Write API | Planned | Create a project from an arrangement file. |
| MCP server | Planned | Let your own AI assistant search music and build projects on your account. |

This list will grow as each piece ships. See the [changelog](CHANGELOG.md).

## Ground rules

- **Artists' rights come first.** Anything built on InternetDJ follows each song's settings. Downloads are only offered where the artist allows them, and no song may be used to train AI unless its artist has opted in.
- **Public means public.** Private songs and projects are never reachable from anything documented here.
- **Stable means stable.** Anything marked stable keeps working. Breaking changes go in a new version, announced in the changelog well ahead of time.

## Questions and ideas

Open an [issue](../../issues). Tell us what you're trying to build; that shapes what we build next.

Found a security problem? Please don't open a public issue. See [SECURITY.md](SECURITY.md).

## License

The docs, schemas and examples in this repository are [MIT licensed](LICENSE). Music on InternetDJ belongs to the artists who made it and is not covered by this license.
