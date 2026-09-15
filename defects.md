# Defects

Each block below is one defect key. A key is a short name used in the `defect`
column of `zerocopy.tsv`, `classes.tsv`, and `compile.tsv`; every model whose
failure has the same cause carries the same key. A block states the attribution
(model defect, compiler defect, or generator defect) and the fix site, so the key
answers "what is wrong and where does it get fixed" once instead of once per model.
A key that starts with `ignore:` means the failure is accepted for now and the rest
of the key is the reason; it is not a fix site and it does not expire on its own.
