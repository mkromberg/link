# Link.Break

    ]LINK.Break [<ns>] [-all] [-recursive={on|off|error}]
    
    msg ← {opts} ⎕SE.Link.Break ns

Breaks an existing link: Does not affect the contents of the active workspace except to remove all traces of the link, preventing any further synchronisation from taking place.

#### Arguments

- namespace name(s) or reference(s)

#### Options

- `all`: Break all existing links (arguments are ignored)
- `recursive` {on|off|**error**}: Break child namespaces too if they have separately defined links.


#### Result

- String describing namespaces that were:
  - effectively unlinked
  - not linked in the first place
  - not found