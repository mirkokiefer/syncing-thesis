
Pivot:

- diff and merge are both a hierarchical process

- start on layer 1 with diff, merge diffs

- on each conflict do a diff on these objects only

- merge the objects, on each conflict repeat...

Maybe Pivot?:

- use changes stream like couch - each commit with its new blob hashes (diff)

- should work, if you have read until a commit you have all its ancestors, even branches

- a lot simpler than cross-commit diff computation

- ok it might only result in redundant branches propagated - but that should be rare

- can we avoid sending redundant versions?

- if in couchdb we put each substructure in separate docs we might get the same result as in 3-way-merging

- we only need 3-way-merging if substructures can be modified whose individual history is not preserved - in other words: if a new version is created for an entire blob although only a small part of it changed we need 3-way-merging.
so: if we track the history of each atomic structure individually we don't need any merging - only conflict resolution!

- only commit when syncing - this way we dont have redundant versions (only when peer2peer syncing)