---
icon: material/new-box
---

!!! quote "Changes in sing-box 1.9.0"

    :material-plus: [client_subnet](#client_subnet)

# DNS

### Structure

```json
{
  "dns": {
    "servers": [],
    "rules": [],
    "final": [
      "local"
    ],
    "strategy": "",
    "disable_cache": false,
    "disable_expire": false,
    "independent_cache": false,
    "reverse_mapping": false,
    "client_subnet": "",
    "fakeip": {}
  }
}

```

### Fields

| Key      | Format                          |
|----------|---------------------------------|
| `server` | List of [DNS Server](./server/) |
| `rules`  | List of [DNS Rule](./rule/)     |
| `fakeip` | [FakeIP](./fakeip/)             |

#### final

!!! note ""

    You can ignore the JSON Array [] tag when the content is only one item

List of default dns server tag.

When the count of list is greater than one, concurrent requests to all dns servers and take the fastest non-empty response.

The first server will be used if empty.

#### strategy

Default domain strategy for resolving the domain names.

One of `prefer_ipv4` `prefer_ipv6` `ipv4_only` `ipv6_only`.

Take no effect if `server.strategy` is set.

#### disable_cache

Disable dns cache.

#### disable_expire

Disable dns cache expire.

#### independent_cache

Make each DNS server's cache independent for special purposes. If enabled, will slightly degrade performance.

#### reverse_mapping

Stores a reverse mapping of IP addresses after responding to a DNS query in order to provide domain names when routing.

Since this process relies on the act of resolving domain names by an application before making a request, it can be
problematic in environments such as macOS, where DNS is proxied and cached by the system.

#### client_subnet

!!! question "Since sing-box 1.9.0"

Append a `edns0-subnet` OPT extra record with the specified IP prefix to every query by default.

If value is an IP address instead of prefix, `/32` or `/128` will be appended automatically.

Can be overrides by `servers.[].client_subnet` or `rules.[].client_subnet`.
