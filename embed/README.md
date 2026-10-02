# Embeds and oEmbed

Every public song on InternetDJ can be played on another site. There are three ways to do it, from least to most work.

## 1. Paste the link

Tools that support oEmbed discovery, such as Iframely, Embedly and the platforms built on them, turn a pasted link into a player automatically. Song pages advertise their oEmbed endpoint in the page head, so a plain song link is enough:

```
https://internetdj.co/song/196
```

Posting a song link on X shows a playable card.

## 2. Copy the embed code

On any song page, choose **Embed** to copy a ready-made iframe. There are two layouts.

**Compact bar** (full width, 166px tall):

```html
<iframe src="https://internetdj.co/embed/song/196"
  width="100%" height="166" style="border:0" loading="lazy"
  title="Too Close Now by DJ Housefarben on InternetDJ"></iframe>
```

**Large card** (cover art on top, 400 x 568):

```html
<iframe src="https://internetdj.co/embed/song/196?layout=card"
  width="400" height="568" style="border:0" loading="lazy"
  title="Too Close Now by DJ Housefarben on InternetDJ"></iframe>
```

Keep the `title` attribute. It's what screen readers announce for the player.

## 3. Use the oEmbed endpoint

If you're building a platform that unfurls links, call the [oEmbed](https://oembed.com) endpoint directly.

```
GET https://internetdj.co/oembed?url={song URL}&format=json
```

| Parameter | Required | |
|---|---|---|
| `url` | yes | A song page link (`/song/{id}`) or player link (`/embed/song/{id}`) on internetdj.co. `www.` is fine. |
| `format` | no | `json` (the default). XML is not supported and answers `501`. |
| `maxwidth` | no | Largest width you can show, in pixels. |
| `maxheight` | no | Largest height you can show, in pixels. |
| `layout` | no | `card` for the large card. Anything else gets the compact bar. |

### Example

```sh
curl "https://internetdj.co/oembed?url=https://internetdj.co/song/196&format=json"
```

```json
{
  "version": "1.0",
  "type": "rich",
  "provider_name": "InternetDJ",
  "provider_url": "https://internetdj.co",
  "title": "Too Close Now",
  "author_name": "DJ Housefarben",
  "author_url": "https://internetdj.co/profile/138",
  "html": "<iframe src=\"https://internetdj.co/embed/song/196\" width=\"600\" height=\"166\" style=\"border:0\" loading=\"lazy\" title=\"Too Close Now by DJ Housefarben on InternetDJ\"></iframe>",
  "width": 600,
  "height": 166,
  "cache_age": 3600
}
```

### Sizes

- **Compact:** always 166px tall. Width is `maxwidth` if you send one, otherwise 600.
- **Card:** 400 wide by default, and always 168px taller than it is wide (the square cover plus the title and controls). It shrinks to fit `maxwidth` and `maxheight`, but never below 240 wide, so a very small `maxheight` can still be exceeded.

### Errors

| Status | Meaning |
|---|---|
| `404` | The link isn't an InternetDJ song, or the song is private or deleted. |
| `501` | A `format` other than `json` was requested. |

Responses can be cached for an hour (`cache_age: 3600`).

## Discovery

Song pages include the standard discovery link, so consumers that support oEmbed discovery find the endpoint on their own:

```html
<link rel="alternate" type="application/json+oembed"
  href="https://internetdj.co/oembed?url=https%3A%2F%2Finternetdj.co%2Fsong%2F196&format=json"
  title="Too Close Now by DJ Housefarben">
```
