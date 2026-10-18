# Pol-ler (freshRSS fork)

Part of [dilroopg's FreshRSS extensions](https://github.com/dilroopg/freshrss-extensions) — see the hub for everything else.

> **This is a fork of [aldotroiano/pol-ler](https://github.com/aldotroiano/pol-ler)**
> with one small feature added, made for use with the
> **[pol-ler Sites](https://github.com/dilroopg/freshrss-poller-sites)** FreshRSS
> extension. Upstream is MIT-licensed; so is this.
>
> **You do not need this fork.** Stock upstream works. If you run upstream's own
> compose file the feed URL comes out relative to whatever hostname your browser
> used, which is usually fine. This fork exists for one specific case, described
> below. If that case is not yours, use upstream.

> ### ⚠️ This fork will probably not be updated again
>
> It was made to solve one problem, it solves it, and it is not maintained
> further. Do not expect upstream bug fixes, security patches or new features to
> reach it.
>
> That is a real trade-off, not a formality: this fork is a snapshot of upstream
> as it stood when it was forked, plus twelve lines. If you care about staying
> current, run upstream and correct the feed hostname by hand instead — see
> *If you would rather not run a fork* below.
>
> If you do want to update it yourself, that is straightforward, because the
> change is tiny and lives in three files:
>
> ```bash
> git remote add upstream https://github.com/aldotroiano/pol-ler.git
> git fetch upstream && git merge upstream/master
> ```
>
> Conflicts, if any, will be in the three files listed below and nowhere else.

## What this fork adds

One environment variable: **`FEED_URL_BASE`**.

### The problem it solves

pol-ler shows you a copyable subscribe link, built in the browser from
`location.protocol + '//' + location.hostname`. That is correct when the browser
and the reader are the same machine — but a feed reader fetches the URL
**server-side, from inside its own container**.

So if you open pol-ler in a browser at `https://myhost.ts.net:8444` and copy that
link into FreshRSS, the link is dead: inside the FreshRSS container,
`myhost.ts.net:8444` is either unreachable or resolves to nothing. The same
applies to a LAN address, or to `localhost`.

The address that actually works is pol-ler's name on the shared Docker network —
`http://pol-ler:8088` — which no browser can reach. Two addresses, one of which
is useless to each side.

### The fix

Set `FEED_URL_BASE` and the displayed subscribe link uses it verbatim instead of
guessing from the browser:

```yaml
services:
  pol-ler:
    environment:
      FEED_URL_BASE: 'http://pol-ler:8088'
```

With it set, the link in pol-ler's feed preview is already the correct internal
address, ready to paste into FreshRSS. Unset (the default), behaviour is exactly
upstream's — the browser-derived link, unchanged.

### What changed, precisely

Three files, 12 lines added, 4 removed:

| File | Change |
|---|---|
| `frontend/frontend/settings.py.example` | reads `FEED_URL_BASE` from the environment, defaults empty |
| `frontend/frontend/views.py` | passes it to the preview template |
| `frontend/frontend/templates/frontend/preview.html` | uses it for the link when set; otherwise the original script tag runs unchanged |

No database schema change, no API change, no behaviour change when the variable
is unset. Upstream's compose file is untouched.

### Compatibility

Tested against upstream at commit `a31bb5e`. Because the change is a single
optional environment variable that defaults to the old behaviour, it applies
cleanly to any upstream version with the same templates.

## If you would rather not run a fork

Two alternatives, both fine:

1. **Use upstream, and edit the feed URL by hand** after adding it to your
   reader. Every feed works; you just correct the hostname once.
2. **Publish pol-ler on an address that both your browser and the container can
   reach** — typically the host's LAN IP. Then the browser-derived link is
   already correct and no fork is needed.

The fork is a convenience for the shared-Docker-network setup. It is not
required, and it is not a bug fix for upstream — upstream's behaviour is right
for the case it was written for.

---



## Pol-ler

An improved RSS generator - renovated & fork of the no-longer-maintained [Pol](https://github.com/taroved/pol).

![image info](./asset1.png)

## Quickstart (Docker)

```
git clone https://github.com/aldotroiano/pol-ler
cd pol-ler
make build && make start
```

Visit http://localhost:8088

## Quickstart (Local)

```
git clone https://github.com/aldotroiano/pol-ler
cd pol-ler
make install
make migrate
make runserver
```

Visit http://localhost:8000

## Make commands

Run `make help` to list all targets. Common commands:

- **help**: Show help
- **start**: Start Docker containers
- **stop**: Stop Docker containers
- **restart**: Restart Docker containers
- **build**: Build Docker containers
- **logs**: Follow Docker logs
- **shell**: Open shell in container
- **install**: Install Python dependencies
- **migrate**: Run Django migrations (local)
- **loaddata**: Load initial data (local)
- **runserver**: Run Django development server (local)
- **downloader**: Run downloader server (local)
- **test**: Run tests
- **clean**: Remove Python cache files

## Access (port 8088)
Docker Host IP in browser. 

For exmaple:
````
http://localhost:8088
````

## License

MIT - 2016 Alexandr Nesterenko (https://github.com/taroved/pol)
