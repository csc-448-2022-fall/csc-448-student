---
jupyter:
  jupytext:
    encoding: '# -*- coding: utf-8 -*-'
    formats: ipynb,md,py
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.8.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

<!-- #region slideshow={"slide_type": "slide"} -->
# Assignment 7 - How do we locate disease causing mutations?
## Combinatorial Pattern Matching
<!-- #endregion -->

```python slideshow={"slide_type": "skip"}
%matplotlib inline
%load_ext autoreload
%autoreload 2

## BEGIN SOLUTION
import joblib
answers = {}
## END SOLUTION
    
import pandas as pd
import numpy as np

import Assignment7_helper 

from pathlib import Path
home = str(Path.home()) # all other paths are relative to this path. 
# This is not relevant to most people because I recommended you use my server, but
# change home to where you are storing everything. Again. Not recommended.
```

<!-- #region slideshow={"slide_type": "subslide"} -->
**Exercise 1**: Trie construction problem - Construct a trie from a collection of patterns

Input: A collection of strings (``patterns``)

Output: ``trie(patterns)`` in the form a networkx direct graph.
<!-- #endregion -->

```python
Assignment7_helper.patterns1
```

```python
Assignment7_helper.patterns2
```

```python slideshow={"slide_type": "fragment"}
trie1 = Assignment7_helper.trie_construction(Assignment7_helper.patterns1)
Assignment7_helper.show(trie1)

trie2 = Assignment7_helper.trie_construction(Assignment7_helper.patterns2)
## BEGIN SOLUTION
answers["answer_exercise_1"] = Assignment7_helper.to_adj(trie2)
## END SOLUTION
Assignment7_helper.show(trie2)
```

<!-- #region slideshow={"slide_type": "subslide"} -->
*Applying the trie to multiple pattern matching*

Given a string ``text`` and ``trie(patterns)``, it is easy to see if a prefix of ``text`` matches any of our patterns. 

We just need to simply work through the graph. If we get to a leaf node, then we can output the path from the root to the leaf node.
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
print(Assignment7_helper.prefix_trie_matching("bana",trie2))
print(Assignment7_helper.prefix_trie_matching("bananaabacadaba1",trie2))
```

<!-- #region slideshow={"slide_type": "subslide"} -->
## Can you write a function that finds whether any strings in patterns match a substring of text?
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
*Exercise 2:* Find whether any strings in ``patterns`` match a substring of ``text`` starting at position $i$. 

Input: ``text`` and ``trie(patterns)``

Output: all of the starting locations $i$ where a string in ``patterns`` matches a substring of ``text``.
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
positions = Assignment7_helper.trie_matching("bananablahblahantennanabnablkjdf",trie2)
## BEGIN SOLUTION
answers["answer_exercise_2"] = positions
## END SOLUTION
positions
```

<!-- #region slideshow={"slide_type": "subslide"} -->
*Exercise 3:* Construct a suffix trie

Input: ``text``

Output: a suffix trie as a networkx object.
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
trie3 = Assignment7_helper.suffix_trie("panamabananas$")
## BEGIN SOLUTION
answers["answer_exercise_3"] = Assignment7_helper.to_adj(trie3)
## END SOLUTION
Assignment7_helper.show(trie3)
```

<!-- #region slideshow={"slide_type": "subslide"} -->
**Exercise 4:** Modified suffix trie construction (not collapsed yet)

Input: A string ``text``

Output: Return a modified suffix trie as a networkx object using the method describe above
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
trie4,leaf_nodes = Assignment7_helper.modified_suffix_trie("panamabananas$")
## BEGIN SOLUTION
answers["answer_exercise_4"] = Assignment7_helper.to_adj(trie4)
## END SOLUTION
Assignment7_helper.show(trie4)
```

<!-- #region slideshow={"slide_type": "subslide"} -->
## Putting it together
We are ready to put it together and collapse the edges. It is easy to see what we can collapse when there is a single path that is sequential. We also now have the numbers needed to collapse. We would use pointers (integers) instead of the string itself, but for display purposes we stick to the string.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
**Exercise 5:** Modified suffix trie construction

Input: A string ``text``

Output: Return a suffix tree that is now collapsed with the edges containing the substring. 
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
tree = Assignment7_helper.suffix_tree_construction("panamabananas$")
## BEGIN SOLUTION
answers["answer_exercise_5"] = Assignment7_helper.to_adj(tree)
## END SOLUTION
Assignment7_helper.show(tree)
```

```python
Assignment7_helper.to_adj(tree).index
```

```python slideshow={"slide_type": "skip"}
## BEGIN SOLUTION
joblib.dump(answers,"../tests/answers_Assignment7.joblib");
## END SOLUTION
# Don't forget to push!
```
