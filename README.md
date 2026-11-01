# user-agents-ng

[![CI](https://github.com/SAKMZ/python-user-agents/actions/workflows/ci.yml/badge.svg)](https://github.com/SAKMZ/python-user-agents/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/user-agents-ng.svg)](https://pypi.org/project/user-agents-ng/)

> **A maintained, drop-in successor to [`user-agents`](https://pypi.org/project/user-agents/).**
> The original [`selwin/python-user-agents`](https://github.com/selwin/python-user-agents)
> has had no release since 2020. `user-agents-ng` picks up where it left off:
> modern packaging, type hints, and tested support for Python 3.9–3.14.
> **The import stays the same (`from user_agents import parse`) — just change what you `pip install`.**

`user_agents` is a Python library that provides an easy way to identify/detect
devices like mobile phones, tablets and their capabilities by parsing
(browser/HTTP) user agent strings. The goal is to reliably detect whether:

* User agent is a mobile, tablet or PC based device
* User agent has touch capabilities (has touch screen)

`user_agents` relies on the excellent [ua-parser](https://github.com/ua-parser/uap-python)
to do the actual parsing of the raw user agent string.

## Installation

`user-agents-ng` is hosted on [PyPI](https://pypi.org/project/user-agents-ng/) and can be installed with:

    pip install user-agents-ng

For a roughly 20x parsing speedup on uncached user agents, install the optional
Rust-backed parser too. It's a drop-in change, no code needs updating:

    pip install "user-agents-ng[fast]"

### Migrating from `user-agents`

`user-agents-ng` is a drop-in replacement. Swap the dependency and your code
keeps working unchanged:

    pip uninstall user-agents
    pip install user-agents-ng

```python
from user_agents import parse  # <- unchanged
```

## Usage

Various basic information that can help you identify visitors can be accessed via the
`browser`, `device` and `os` attributes. For example:

```python
from user_agents import parse

# iPhone's user agent string
ua_string = 'Mozilla/5.0 (iPhone; CPU iPhone OS 5_1 like Mac OS X) AppleWebKit/534.46 (KHTML, like Gecko) Version/5.1 Mobile/9B179 Safari/7534.48.3'
user_agent = parse(ua_string)

# Accessing user agent's browser attributes
user_agent.browser  # returns Browser(family='Mobile Safari', version=(5, 1), version_string='5.1')
user_agent.browser.family  # returns 'Mobile Safari'
user_agent.browser.version  # returns (5, 1)
user_agent.browser.version_string   # returns '5.1'

# Accessing user agent's operating system properties
user_agent.os  # returns OperatingSystem(family='iOS', version=(5, 1), version_string='5.1')
user_agent.os.family  # returns 'iOS'
user_agent.os.version  # returns (5, 1)
user_agent.os.version_string  # returns '5.1'

# Accessing user agent's device properties
user_agent.device  # returns Device(family='iPhone', brand='Apple', model='iPhone')
user_agent.device.family  # returns 'iPhone'
user_agent.device.brand # returns 'Apple'
user_agent.device.model # returns 'iPhone'

# Viewing a pretty string version
str(user_agent) # returns "iPhone / iOS 5.1 / Mobile Safari 5.1"
```

`user_agents` also exposes a few other more "sophisticated" attributes that are
derived from one or more basic attributes defined above. These attributes should
correctly identify popular platforms/devices; pull requests to support smaller
ones are always welcome.

Currently these attributes are supported:

* `is_mobile`: whether user agent is identified as a mobile phone (iPhone, Android phones, Blackberry, Windows Phone devices etc)
* `is_tablet`: whether user agent is identified as a tablet device (iPad, Kindle Fire, Nexus 7 etc)
* `is_pc`: whether user agent is identified to be running a traditional "desktop" OS (Windows, OS X, Linux)
* `is_tv`: whether user agent is identified as a smart TV or streaming device (Samsung Tizen, LG webOS, Android TV, Fire TV, Roku, Chromecast etc)
* `is_touch_capable`: whether user agent has touch capabilities
* `is_bot`: whether user agent is a crawler, spider or headless browser. Besides the bots ua-parser knows about, this catches any user agent with a bot-style product token (`Applebot/0.1`), a contact URL (`+http://example.com/bot`) or a contact email address
* `is_email_client`: whether user agent is an email client (Outlook, Thunderbird, Apple Mail etc)

`parse()` also accepts `bytes` and `None`, so a raw header or a request with no
`User-Agent` header at all won't raise an exception.

For example:

```python
from user_agents import parse

# Let's start from an old, non touch Blackberry device
ua_string = 'BlackBerry9700/5.0.0.862 Profile/MIDP-2.1 Configuration/CLDC-1.1 VendorID/331 UNTRUSTED/1.0 3gpp-gba'
user_agent = parse(ua_string)
user_agent.is_mobile # returns True
user_agent.is_tablet # returns False
user_agent.is_touch_capable # returns False
user_agent.is_pc # returns False
user_agent.is_bot # returns False
str(user_agent) # returns "BlackBerry 9700 / BlackBerry OS 5.0.0 / BlackBerry 9700"

# Now a Samsung Galaxy S3
ua_string = 'Mozilla/5.0 (Linux; U; Android 4.0.4; en-gb; GT-I9300 Build/IMM76D) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30'
user_agent = parse(ua_string)
user_agent.is_mobile # returns True
user_agent.is_tablet # returns False
user_agent.is_touch_capable # returns True
user_agent.is_pc # returns False
user_agent.is_bot # returns False
str(user_agent) # returns "Samsung GT-I9300 / Android 4.0.4 / Android 4.0.4"

# Touch capable Windows 8 device
ua_string = 'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Trident/6.0; Touch)'
user_agent = parse(ua_string)
user_agent.is_mobile # returns False
user_agent.is_tablet # returns False
user_agent.is_touch_capable # returns True
user_agent.is_pc # returns True
user_agent.is_bot # returns False
str(user_agent) # returns "PC / Windows 8 / IE 10.0"
```

## Running Tests

    python -m unittest user_agents.tests

## Changelog

### Version 3.1.0

* Added `is_tv` for smart TVs and streaming devices, which no longer count as touch capable tablets, phones or PCs.
* `is_bot` now catches bots missed by ua-parser, like Applebot, Amazonbot, Bytespider and headless Chrome.
* Added the `user-agents-ng[fast]` extra, which parses about 20x faster.
* `parse()` accepts `None` and `bytes` instead of raising `TypeError`.
* Firefox on Android tablets is now a tablet, Dalvik clients are no longer tablets, and KaiOS is mobile.
* Now uses the current `ua-parser` API and requires `ua-parser>=1.0.0`, which keeps detection data up to date.

### Version 3.0.0

* Released as `user-agents-ng`, a maintained successor to `user-agents`.
* Tested support for Python 3.9–3.14; dropped Python 2.
* Migrated packaging to `pyproject.toml`; added `py.typed` and type hints.
* Replaced Travis CI with GitHub Actions (Linux + Windows).

For the pre-3.0 history, see the original
[project changelog](https://github.com/selwin/python-user-agents).

## Credits

Originally created by Selwin Ong and the team at
[Stamps](http://stamps.co.id). Maintained as `user-agents-ng` by
[Saif Ali](https://github.com/SAKMZ). Licensed under the MIT License.
