#Graph Mode FAQ

This document holds frequently asked questions about the new graph mode, and how various tasks can be accomplished in graph mode and in classic mode.

If you have a relevant question, please add it to this document in a pull request.

##What does a SNP look like in graph versus classic mode?

In "classic" mode, a SNP is represented by a `Variant`, with `referenceBases` set to one base, and `alternateBases` set to the other.

In "graph" mode, a SNP exists as a single-base `Reference` (or novel sequence in a `VariantSet`) with the alternate base, joined onto the `Reference` with the original base, like this:

```
       -G-
      /   \
--A--C--T--G--C--A--
```

To express the genotype of this SNP, a variant caller will need to emit a pair of `Allele`s, one of which follows a single-base path through the original base, and one of which follows a single-base path through the alternate base. It would then emit `AlleleCall`s noting the copy number of each `Allele` in each `CallSet`.

The variant caller may additionally emit a `Variant` tying the two `Allele`s together, and giving genotypes in more traditional notation.

##What does a short indel look like in graph versus classic mode?

In "classic" mode, an indel is represented by a `Variant`, with `referenceBases` set to "" (for an insertion) or some bases (for a deletion), and `alternateBases` set to the inserted bases (for an insertion) or "" (for a deletion).

In "graph" mode, an indel exists as a `Reference` (or novel sequence in a `VariantSet`) with the inserted bases (or no bases for a deletion), joined onto the `Reference` such that it connects the endpoints of the indel, like this:

```
Insertion:

        -C--A-
       / ____/
      / /
      ||
      /\
--A--C--T--G--C--A--

Deletion:

--A--C--T--G--C--A--
   \_________/
```

To express the genotype of an indel, a variant caller will need to emit a pair of `Allele`s, one of which follows the path with the extra bases, and one of which follows the 0-length path consisting of the adjacency broken by the insertion or created by the deletion. The caller would then emit `AlleleCall`s noting the copy number of each `Allele` in each `CallSet`.

The variant caller may additionally emit a `Variant` tying the two `Allele`s together, and giving genotypes in more traditional notation.

##How do I walk the graph to find all variants within 10kbp of a specific position?

In "classic" mode, one can issue a `searchVariants()` call interrogating the range 10kb upstream and downstream of the position of interest. All `Variant`s overlapping that range would be returned.

In "graph" mode, the situation is more complicated. You want to perform a recursive search of the graph out to a distance of 10kb from your start position, following all possible paths. 

You can use `searchReferences()` and `searchVariantSetSequences()` to get information about all the children of the sequence carrying the position you are interested in, and see if any of their join locations on your sequence of interest are within a 10kb window around your position of interest, and attached such that it is possible to read into them in the direction you are traversing the parent. In that case, you would have to recurse down into each such child, work out how far in from the joined end you can get with whatever is left of your 10kb window size after walking out to where the join is, and recursively search that region for more children.

If you come to the end of a sequence in your search, you will need to see if it is joined onto a parent (using `getReference()` or `getVariantSetSequence()` to get its end joins), then explore the parent out from the join position in the appropriate direction, searching for more children.

Once you have determined all the ranges on all the sequences that are "within 10kb" of your starting position, you can make a `searchAlleles()` call on each of them to get all `Allele` objects involving any bases within 10kb of your start position. If any are associated with `Variant` objects, you can use the `getVariant()` call to retrieve those `Variant`s by ID.

If you are only interested in `Variant` objects with reference `Allele`s overlapping your chosen ranges, you can use `searchvariants()` calls instead of `searchAlleles()` calls. This will ignore `Allele`s which are not part of `Variant`s, or which are not the reference `Allele`s for their `Variant`s.



