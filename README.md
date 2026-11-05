# chickensmoothie

Unofficial standalone asynchronous client for Chicken Smoothie. Requires Python 3.11+.
Not affiliated with or endorsed by Chicken Smoothie.
No Discord, database, environment variables, or bot startup dependencies.

## Installation

From a local checkout:

```powershell
uv sync
```

Or install with pip: `python -m pip install .`.
After the first PyPI release, users can install with `python -m pip install chickensmoothie`.

## Usage

```python
import asyncio
import chickensmoothie

async def main():
    pet = await chickensmoothie.get_pet("IMAGE_K_ID", years=[2026])
    matches = await chickensmoothie.get_archive("IMAGE_K_ID", years=[2026])
    async with chickensmoothie.Client() as client:
        # Login is needed for account-dependent opening information.
        await client.login("username", "password")
        status = await client.get_opening_status()
        events = await client.get_archive_events(2026, "pets")
        page = await client.get_archive_event(events[0], "pets", 2026)
        for entry in page.entries:
            print(entry)

asyncio.run(main())
```

All network methods are async. `get_pet` returns the first matching `Pet` or
`None`; `get_archive` returns all matching `Pet` archive records. IDs refer to
image URL `k` values, as used by the bot's identification database, not numeric
IDs for individually owned pets. Individual pet profiles are not supported.
Without `years`, lookups scan archives from 2008 through the current UTC year;
this can make many requests. Restrict years when possible. The bot retains its
SQLite index for fast identification instead of scanning on every command.

`get_archive_events(year, "items")` and `get_archive_event(event, "items", year)`
return item records, including both image IDs, name, year, event and URLs.
Archive extraction retains the bot's known duplicate-pet exclusions and
`pageSize=3000` behavior. Events beyond that server response limit are not paged.

Use one `Client` context for repeated calls. `Client(session)` accepts an
existing aiohttp session and never closes it. Credentials are explicit.
HTTP failures propagate as aiohttp exceptions; failed login raises
`AuthenticationError`. Missing archive matches return empty results, while
HTTP errors are never treated as missing pets.

The bot owns SQLite/MongoDB storage, RSS feed caching, reminder scheduling,
Discord formatting and image uploads. `NEWS_FEED_URL` exposes the news source
for other applications to use with their preferred RSS reader.

## Development

Run offline regression tests: `uv run python -m unittest discover -s tests -v`.
Build release files: `uv build --no-sources`.
See [RELEASING.md](RELEASING.md) for validation and PyPI upload instructions.

## License

ISC. See [LICENSE](LICENSE). This license applies to the package code, not to
Chicken Smoothie's images, content, or branding.
