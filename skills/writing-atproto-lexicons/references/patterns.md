# Lexicon schema patterns

These are starting shapes, not copy-paste domain models. Replace names, limits, descriptions, authentication statements, errors, and constraints from actual product requirements.

## Record

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.recipe",
  "defs": {
    "main": {
      "type": "record",
      "description": "A recipe published by an account.",
      "key": "tid",
      "record": {
        "type": "object",
        "required": ["title", "createdAt"],
        "properties": {
          "title": {
            "type": "string",
            "maxLength": 1000,
            "maxGraphemes": 100,
            "description": "Human-readable recipe title."
          },
          "createdAt": {
            "type": "string",
            "format": "datetime"
          }
        }
      }
    }
  }
}
```

Do not add `$type` to `properties`; encoded records supply it automatically as a protocol discriminator. Choose `required` from functional invariants, not because clients would find a field convenient.

Common record key patterns include `tid` for multiple generated records and `literal:self` for a singleton declaration/profile. Confirm less common key strategies against the record-key specification and local conventions.

## Reusable defs

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.defs",
  "defs": {
    "recipeSummary": {
      "type": "object",
      "required": ["uri", "cid", "title"],
      "properties": {
        "uri": {
          "type": "string",
          "format": "at-uri",
          "description": "AT URI of the recipe record."
        },
        "cid": {
          "type": "string",
          "format": "cid",
          "description": "CID of the recipe record version represented by this summary."
        },
        "title": {
          "type": "string",
          "maxLength": 1000,
          "maxGraphemes": 100
        }
      }
    },
    "visibilityPublic": {
      "type": "token",
      "description": "The resource is visible without authentication."
    }
  }
}
```

Named tokens are referenced as fully qualified strings, for example `com.example.recipe.defs#visibilityPublic` in `knownValues`.

## Query with pagination

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.listRecipes",
  "defs": {
    "main": {
      "type": "query",
      "description": "Lists public recipes. Authentication is optional and does not personalize the response.",
      "parameters": {
        "type": "params",
        "properties": {
          "author": {
            "type": "string",
            "format": "at-identifier",
            "description": "Account whose recipe records should be returned."
          },
          "limit": {
            "type": "integer",
            "minimum": 1,
            "maximum": 100,
            "default": 50
          },
          "cursor": {
            "type": "string",
            "description": "Opaque pagination cursor from a previous response."
          }
        }
      },
      "output": {
        "encoding": "application/json",
        "schema": {
          "type": "object",
          "required": ["recipes"],
          "properties": {
            "cursor": {
              "type": "string",
              "description": "Opaque cursor for the next page; omitted when pagination is complete."
            },
            "recipes": {
              "type": "array",
              "items": {
                "type": "ref",
                "ref": "com.example.recipe.defs#recipeSummary"
              }
            }
          }
        }
      }
    }
  }
}
```

The request's `limit` is an upper bound, not a promised result count. A page can contain fewer or zero visible items and still return a cursor. Pagination ends only when the response omits `cursor`.

## Procedure

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.deleteRecipe",
  "defs": {
    "main": {
      "type": "procedure",
      "description": "Deletes a recipe owned by the authenticated account. Authentication is required.",
      "input": {
        "encoding": "application/json",
        "schema": {
          "type": "object",
          "required": ["uri"],
          "properties": {
            "uri": {
              "type": "string",
              "format": "at-uri",
              "description": "AT URI of the recipe record to delete."
            }
          }
        }
      },
      "output": {
        "encoding": "application/json",
        "schema": {
          "type": "object",
          "properties": {}
        }
      },
      "errors": [
        {
          "name": "RecipeNotFound",
          "description": "The referenced recipe does not exist or is not visible to the caller."
        }
      ]
    }
  }
}
```

Even an endpoint without meaningful response data declares output and encoding.

## Subscription with sequencing

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.subscribeRecipes",
  "defs": {
    "main": {
      "type": "subscription",
      "description": "Streams public recipe events. Authentication is not required.",
      "parameters": {
        "type": "params",
        "properties": {
          "cursor": {
            "type": "integer",
            "minimum": 0,
            "description": "Sequence number from which backfill should begin."
          }
        }
      },
      "message": {
        "schema": {
          "type": "union",
          "refs": ["#recipeEvent", "#info"]
        }
      },
      "errors": [
        {
          "name": "FutureCursor",
          "description": "The requested cursor is newer than the current sequence."
        }
      ]
    },
    "recipeEvent": {
      "type": "object",
      "required": ["seq", "recipe"],
      "properties": {
        "seq": { "type": "integer", "minimum": 0 },
        "recipe": {
          "type": "ref",
          "ref": "com.example.recipe.defs#recipeSummary"
        }
      }
    },
    "info": {
      "type": "object",
      "required": ["name"],
      "properties": {
        "name": {
          "type": "string",
          "knownValues": ["OutdatedCursor"]
        },
        "message": {
          "type": "string",
          "maxLength": 2000,
          "maxGraphemes": 200
        }
      }
    }
  }
}
```

For conventional sequence/backfill behavior:

- omit cursor to start with new messages at the current point
- increase `seq` monotonically, allowing gaps
- reject future cursors and close the connection
- for a cursor older than retained history (or zero), send an info message representing `OutdatedCursor`, then stream from the oldest available event

Confirm the exact info-message naming and wire convention with neighboring subscriptions before standardizing it in a project.

## Permission set

```json
{
  "lexicon": 1,
  "id": "com.example.recipe.authWrite",
  "defs": {
    "main": {
      "type": "permission-set",
      "description": "Allows an application to manage the account's recipe records.",
      "title": "Manage recipes",
      "detail": "Create, update, and delete recipes stored in your account.",
      "permissions": [
        {
          "type": "permission",
          "resource": "repo",
          "collection": ["com.example.recipe.recipe"],
          "action": ["create", "update", "delete"]
        }
      ]
    }
  }
}
```

Re-read both the Lexicon and permission specifications for current security semantics before adding RPC permissions.

## Hydrated detailed view

Define API views separately from stored records. For a detailed view, include the original record verbatim instead of duplicating all record fields:

```json
"recipeView": {
  "type": "object",
  "required": ["uri", "cid", "record", "author"],
  "properties": {
    "uri": { "type": "string", "format": "at-uri" },
    "cid": { "type": "string", "format": "cid" },
    "record": { "type": "unknown" },
    "author": { "type": "ref", "ref": "#profileView" },
    "viewer": {
      "type": "ref",
      "ref": "#viewerState",
      "description": "Relationship state for the authenticated viewer; omitted for anonymous requests."
    }
  }
}
```

Using `unknown` in an API view preserves the original record, including off-schema extension fields; the official warning against `unknown` is specifically strongest for stored record objects. If the endpoint serves only one known record type and local conventions provide a safer established representation, use that instead.

Keep viewer-specific state optional and either clearly described or grouped under a dedicated object.

## Sidecars and modality declarations

A mutable sidecar can supplement an immutable or strongly referenced record:

- use the same record key in a different collection
- keep independently changing context out of the original record
- hydrate sidecar context in API views when useful

For a new app modality, define a representative singleton profile or declaration record. Its presence signals participation; deletion signals deactivation. This gives backfill services a concrete collection to enumerate and track.
