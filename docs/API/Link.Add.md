# Link.Add

    ]LINK.Add <items>
    
    msg ← ⎕SE.Link.Add items                                      

This function allows you to add one or more existing APL items to the link, creating the appropriate representation in the linked directory. A source file will be created/updated whether the linked namespace is watched or not.

This is useful to write a new or modified array to a source file: arrays are normally not written to file by Link.

It is also useful when a change has been made to a linked item using any mechanism other than the APL editor, for example the definition of a new dfn using assignment, or the use of `)COPY` to bring new objects into the workspace.

Note: You can create or update an item from source while adding it to the Link by calling [Link.Fix](Link.Fix.md)

#### Arguments

- APL item name(s)

#### Result

- String describing items that were:
  - added (they belong in a linked namespace and were successfully added)
  - not linked (they do not belong to a linked namespace)
  - not found (the name doesn't exist at all)