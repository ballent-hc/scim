![scim-logo](LOGO.png)

[![GoVersion](https://img.shields.io/github/go-mod/go-version/elimity-com/scim.svg)](https://github.com/elimity-com/scim)
[![GoDoc](https://img.shields.io/badge/godoc-reference-blue.svg)](https://pkg.go.dev/github.com/elimity-com/scim)

[![Tag](https://img.shields.io/github/tag/elimity-com/scim.svg)](https://gitHub.com/elimity-com/scim/releases)

🤠 Yeehaw, partner! Saddle up, because this is one *slick* open source ride through the
[SCIM v2.0](http://www.simplecloud.info/#Specification) specification for Golang. 🐄🚗💨

SCIM keeps your identity data wrangled with a flexible schema mechanism and a REST API, so managing users doesn't
feel like herding cats across the whole HTTP prairie.

Wanna bring your own *custom* schemas and extensions to the rodeo? Easy — just plug 'em into the provided
structures. Every resource that rides in gets *validated* against its schema before it's handed off to your
callbacks, so nothing gets past the fence unchecked. 🐘

Here's what's in the corral:

- GET for `/Schemas`, `/ServiceProviderConfig` and `/ResourceTypes`
- CRUD (POST/GET/PUT/DELETE and PATCH) for your own resource types (i.e. `/Users`, `/Groups`, `/Employees`, ...)

Fancier stuff like sorting and bulk operations? Not in this rodeo — **not** supported in this version. 🤷

## 🐘 Installation

Got a (recent) version of Go saddled up already? Great, giddy up and grab the code with go get:

```bash
$ go get github.com/elimity-com/scim
```

## 🐄 Usage

**!** errors are ignored for simplicity — we're keeping this trail ride breezy.

### 1. Create a service provider configuration.

[RFC Config](https://tools.ietf.org/html/rfc7643#section-5) |
[Example Config](https://tools.ietf.org/html/rfc7643#section-8.5)

```go
config := scim.ServiceProviderConfig{
    DocumentationURI: optional.NewString("www.example.com/scim"),
}
```

**!** no additional features/operations are supported in this version — this ain't a rodeo with extra tricks.

### 2. Create all supported schemas and extensions.

[RFC Schema](https://tools.ietf.org/html/rfc7643#section-2) |
[User Schema](https://tools.ietf.org/html/rfc7643#section-4.1) |
[Group Schema](https://tools.ietf.org/html/rfc7643#section-4.2) |
[Extension Schema](https://tools.ietf.org/html/rfc7643#section-4.3)

```go
schema := schema.Schema{
    ID:          "urn:ietf:params:scim:schemas:core:2.0:User",
    Name:        optional.NewString("User"),
    Description: optional.NewString("User Account"),
    Attributes:  []schema.CoreAttribute{
        schema.SimpleCoreAttribute(schema.SimpleStringParams(schema.StringParams{
            Name:       "userName",
            Required:   true,
            Uniqueness: schema.AttributeUniquenessServer(),
        })),
    },
}

extension := schema.Schema{
    ID:          "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
    Name:        optional.NewString("EnterpriseUser"),
    Description: optional.NewString("Enterprise User"),
    Attributes: []schema.CoreAttribute{
        schema.SimpleCoreAttribute(schema.SimpleStringParams(schema.StringParams{
            Name: "employeeNumber",
        })),
        schema.SimpleCoreAttribute(schema.SimpleStringParams(schema.StringParams{
            Name: "organization",
        })),
    },
}
```

### 3. Create all resource types and their callbacks.

[RFC Resource Type](https://tools.ietf.org/html/rfc7643#section-6) |
[Example Resource Type](https://tools.ietf.org/html/rfc7643#section-8.6)

#### 3.1 Callback (implementation of `ResourceHandler`)

[Simple In Memory Example](resource_handler_test.go)

```go
var userResourceHandler scim.ResourceHandler
// initialize w/ own implementation
```

**!** each resource type should have its own resource handler — every critter gets its own wrangler. 🐄

#### 3.2 Resource Type

```go
resourceTypes := []ResourceType{
    {
        ID:          optional.NewString("User"),
        Name:        "User",
        Endpoint:    "/Users",
        Description: optional.NewString("User Account"),
        Schema:      schema,
        SchemaExtensions: []SchemaExtension{
            {Schema: extension},
        },
        Handler:     userResourceHandler,
    },
},
```

### 4. Create Server

```go
serverArgs := &ServerArgs{
    ServiceProviderConfig: config,
    ResourceTypes: resourceTypes,
}

serverOpts := []ServerOption{
    WithLogger(logger), // optional, default is no logging
}

server, err := NewServer(serverArgs, serverOpts...)
```

## 🚗 Backwards Compatibility

Even though this SCIM rig has been cruising through production for a while, it's still an early-stage build and not
every feature is bolted on yet. Heads up: a minor version bump could still rattle your implementation. We won't
throw any breaking changes at you that take hours to fix, but a function name or signature might shift gears now
and then.

That's exactly what happened going from `v0.1` to `v0.2.0`.

## 🐘 String Values for Attributes

By default, this rig will NOT use the `string` type for all attributes, since that's NOT compliant with the SCIM
spec. Still want that behavior anyway? Flip the switch in the `schema` package:

```go
import "github.com/elimity-com/scim/schema"

schema.SetAllowStringValues(true)
```

## 🤠 Additional Checks/Tests

Not everything can be checked by the SCIM server itself — some things you gotta wrangle yourself.
Below's a list of stuff we expect your implementation to cover.

**!** this list is currently incomplete!

We want to keep this list as short as a cowboy's coffee break.
Got ideas on how we could bake these rules into the server itself? Don't be shy, open
[an issue](https://github.com/elimity-com/scim/issues/new) or a PR.

### Mutability

#### Immutable Attributes

*PUT Handler*: If one or more values are already set for the attribute, the input value(s) MUST match.

#### WriteOnly Attributes

*ALL Handlers*: Attribute values SHALL NOT be returned. \
Note: These attributes usually also has a returned setting of "never".

## 🤠 Fork Notice

🚗 This repository is a fork of [elimity-com/scim](https://github.com/elimity-com/scim), owned and maintained by **ballent-hc**.
