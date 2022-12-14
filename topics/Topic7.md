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
# Topic 7 - How do we locate disease causing mutations?
## Combinatorial Pattern Matching
<!-- #endregion -->

```python slideshow={"slide_type": "skip"}
%matplotlib inline
%load_ext autoreload
%autoreload 2

graphviz_installed=True
display_available=True
from IPython.display import Image
```

<!-- #region slideshow={"slide_type": "subslide"} -->
## Slack ice breaker
Dylan versus the Beatles?
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "slide"} -->
## What is read mapping?
So far we've seen one way to process reads
* Assembly - f(reads) => genome (transcriptome)
We would really like to be able to compare DNA reads from an individual (you, me, everyone) and compare them against a reference human genome.

### Why?
To find mutations for an individual and for a subpopulation/group

But why? ... what do you think...?
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "fragment"} -->
One reason is to identify mutations that are correlated to a disease.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
# What causes Ohdo Syndrome?

* Ohdo syndrome causes expressionless, "mask-like" face
* 2011 - biologists found a handful of mutations shared by multiple patients that then identified a single protein-truncating mutation responsible for Ohdo syndrome
* How can we study this in bioinformatics? 
    * Read mapping
    * Single nucleotide polymorphisms (SNPs)
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
### What is a SNP?
<img src="https://online.stat.psu.edu/stat555/sites/onlinecourses.science.psu.edu.stat555/files/snps/snps_reads_02/index.png">
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
## Why not use the algorithms we already have?
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "fragment"} -->
* Why not just assemble each persons genome? 
   * Expensive and error prone
* Other sequence alignment algorithms become runtime bottlenecks when we scale to millions of reads
* And finally, we can solve the problem in a more efficient way, so we should...
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "slide"} -->
## Introduction to Multiple Pattern Matching
* Reads are a few hundred base pairs long
* When hunting for a cause of a genetic disorder, we can immediately eliminate areas of reference genome with exact matches
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
*Multiple Pattern Matching Problem:*

Input: ``text`` and a collection of ``patterns`` (shorter strings)

Output: All starting positions in ``text`` where a string from ``patterns`` appears as a substring.

*Brute force solution*
* Consider each pattern as its own car driving down a road (``text``)
* It would work but as we move each pattern down the same ``text`` we realize how inefficent this is: $O(|text|*|patterns|)$ ... brutal
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "slide"} -->
## Herding Patterns into a trie
Trie - directed acyclic graph written ``trie(patterns)`` has the following properties:
* Single root node with indegree of 0; all other nodes have indegree of 1
* Each edge of ``trie(patterns)`` is labeled with a letter of the alphabet
* Edges leading out a given node have distinct labels
* Every string in ``patterns`` is spelled out by concatenating the letters along some path from the root downward
* Every path from the root to a leaf spells a string from ``patterns``.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
## Example trie(patterns)
<pre>
patterns = ['ATAGA','ATC','GAT']
</pre>
Take 2 minutes and try to make the trie...
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "fragment"} -->
<img src="Topic7_example_trie.png" width=200>
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
**Exercise 1**: Trie construction problem - Construct a trie from a collection of patterns

Input: A collection of strings (``patterns``)

Output: ``trie(patterns)`` in the form a networkx direct graph.
<!-- #endregion -->

```python slideshow={"slide_type": "skip"}
# For the record here is what worked for me. 
#!sudo chown -R $(whoami) /usr/local/lib/pkgconfig
#!chmod u+w /usr/local/lib/pkgconfig
#!brew install graphviz
#!            pip install pygraphviz --install-option="--include-path=/usr/include/graphviz" --install-option="--library-path=/usr/lib/graphviz/"
#pip install pygraphviz
```

```python slideshow={"slide_type": "skip"}
# Just code to help you out here
%matplotlib inline

import networkx as nx
import pandas as pd
import copy

import matplotlib.pyplot as plt

def draw(A):
    return Image(A.draw(format='png', prog='dot'))

def to_adj(T):
    df = pd.DataFrame(nx.adjacency_matrix(T).todense(),index=T.nodes(),columns=T.nodes())
    for i in range(len(df)):
        for j in range(len(df)):
            if df.iloc[i,j] == 1:
                data = T.get_edge_data(df.index[i],df.columns[j])
                df.iloc[i,j] = data['label']
            else:
                df.iloc[i,j] = ""
    return df

def show(G):
    if graphviz_installed:
        # same layout using matplotlib with no labels
        pos = nx.drawing.nx_agraph.graphviz_layout(G, prog='dot')
        #print(edge_labels)
        # Modify node fillcolor and edge color.
        #D.node_attr.update(color='blue', style='filled', fillcolor='yellow')
        #D.edge_attr.update(color='blue', arrowsize=1)
        A = nx.nx_agraph.to_agraph(G)
        # draw it in the notebook
        if display_available:
            display(draw(A))
        else:
            print(A)
    else:
        if display_available:
            display(to_adj(G))
        else:
            print(to_adj(G))
```

```python slideshow={"slide_type": "fragment"}
import networkx as nx

# Inputs: G - networkx graph, current - node name, c - character on edge
# Output: a neighbor of current that is reached by an edge that has label==c; otherwise None
def find_edge(G,current,c): 
    for n in G.neighbors(current):
        if n is None:
            return None
        data = G.get_edge_data(current,n)
        if data['label'] == c:
            return n
    return None

def trie_construction(patterns):
    G = nx.DiGraph()
    G.add_node('root')
    # Your solution here
    ## BEGIN SOLUTION
    cnt = 1
    for pattern in patterns:
        current = 'root'
        for i in range(len(pattern)):
            c = pattern[i]
            node = find_edge(G,current,c)
            if node is None:
                #G.add_node(str(cnt))
                G.add_edge(current,str(cnt),label=c)
                current = str(cnt)
                cnt += 1
            else:
                current = node
    ## END SOLUTION
    return G

patterns1 = ['ATAGA','ATC','GAT']
trie1 = trie_construction(patterns1)
show(trie1)

patterns2 = ['ananas','and','antenna','banana','bandana','nab','nana','pan']
trie2 = trie_construction(patterns2)
show(trie2)
```

<!-- #region slideshow={"slide_type": "subslide"} -->
*Applying the trie to multiple pattern matching*

Given a string ``text`` and ``trie(patterns)``, it is easy to see if a prefix of ``text`` matches any of our patterns. 

We just need to simply work through the graph. If we get to a leaf node, then we can output the path from the root to the leaf node.
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
def prefix_trie_matching(text,trie):
    # Your solution here
    ## BEGIN SOLUTION
    symbol = ""
    v = "root"
    i = 0
    while i < len(text):            
        if len(list(trie.neighbors(v))) == 0:
            return symbol
        else:
            w = find_edge(trie,v,text[i])
            if w is None:
                return None
            symbol += text[i]
            i += 1
            v = w
    ## END SOLUTION
    return None
            
print(prefix_trie_matching("bana",trie2))
print(prefix_trie_matching("bananaabacadaba1",trie2))
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
def trie_matching(text,trie):
    positions = []
    # YOUR SOLUTION HERE
    ## BEGIN SOLUTION
    i = 0
    while i < len(text):
        r = prefix_trie_matching(text[i:],trie)
        if r is not None:
            positions.append(i)
        i += 1
    ## END SOLUTION
    return positions

positions = trie_matching("bananablahblahantennanabnablkjdf",trie2)
positions
```

<!-- #region slideshow={"slide_type": "slide"} -->
## Suffix Tries and Trees
### An assumption and problems
There is a key assumption we are making with our prefix_trie_matching function. The assumption is that no string in pattern is a prefix of another string in pattern. 

We also have a problem that comes up in practice. We are storing the entire trie in memory. A collection of reads may consume upwards of 1 TB. That is a problem for most systems.

To solve this problem we will switch things up and instead of focusing on a data structure (trie) for ``patterns`` we will focus on a suffix trie for ``text``.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
A suffix trie is the trie formed from all suffixes of ``text``.

We will be appending a dollar-sign ("$") to ``text`` in order to mark the end of ``text``. There is nothing special about this symbol.

We will also label each leaf of the resulting trie by the starting position of the suffix whose path through the tre ends at this leaf. 
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
### How do we use suffix tries for pattern matching?
If we want to match a single string ``pattern`` to ``text``, then we first note that if ``pattern`` matches a substring of ``text`` starting at position $i$, then ``pattern`` must also appear at the beginning of the suffix of ``text`` starting at position $i$.

We can easily determine if ``pattern`` occurs in the suffix trie by starting at the root and spelling the symbols of pattern downward.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
**Take 2 minutes**, and see if you can construct a portion of a suffix trie for
**panamabananas$**

Make sure you put the starting position at the leaf

How would determine if bana (pattern) matches?
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
*Exercise 3:* Construct a suffix trie

Input: ``text``

Output: a suffix trie as a networkx object.
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
def suffix_trie(text):
    G = nx.DiGraph()
    G.add_node('root')
    # Your solution here
    ## BEGIN SOLUTION
    i = 0
    cnt = 1
    while i < len(text):
        current = "root"
        suffix = text[i:]
        for c in suffix:
            if c == "$":
                G.add_edge(current,"[%d]"%i,label=c)
            else:
                n = find_edge(G,current,c)
                if n is None:
                    G.add_edge(current,str(cnt),label=c)
                    current = str(cnt)
                    cnt += 1
                else:
                    current = n  
        i+=1
    ## END SOLUTION
    return G

trie3 = suffix_trie("panamabananas$")
show(trie3)
```

<!-- #region slideshow={"slide_type": "subslide"} -->
### Shortcomings
One shortcoming with the suffix trie are the trivial edges (edges that just form a simple sequence with no branching). 

We can overcome this by producing a suffix tree that concatenates edges together when possible. 

There is a linear way to construct the suffix tree, but we will **not** do this in this lab. Further, we would not store the simple sequence itself. We would store a pointer and a length.

We are going to keep things on a conceptual level. In other words, to solve the next exercise, don't worry about memory.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
In our current situation, each edge in a suffix trie is labeled by a single symbol. It is **unclear where this symbol came from in text**. 

To solve this, we can add another label for each edge referring to the position of this symbol in text.

If an edge in a trie corresponds to more than one position in text, then we assign it to its minimum starting position. For example consider the first **n** in a**n**amabananas, a**n**anas, and a**n**as. These are positions 2, 8,and 10. We assign this edge the minimum of these positions.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
**Take a couple of minutes** and work on constructing the modified trie for panamabananas$.
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
Here is the pseudocode.

<img src="Topic7_modifiedsuffixtrie.jpg" width=400>
<!-- #endregion -->

<!-- #region slideshow={"slide_type": "subslide"} -->
**Exercise 4:** Modified suffix trie construction (not collapsed yet)

Input: A string ``text``

Output: Return a modified suffix trie as a networkx object using the method describe above
<!-- #endregion -->

```python slideshow={"slide_type": "subslide"}
# Inputs: G - networkx graph, current - node name, c - character on edge
# Output: a neighbor of current that is reached by an edge that has label==c; otherwise None
def modified_find_edge(G,current,c):
    cv,j = c.split(",")
    j = int(j)
    for n in G.neighbors(current):
        if n is None:
            return None
        data = G.get_edge_data(current,n)
        cw,i = data['label'].split(",")
        i = int(i)
        if cw == cv and j > i:
            return n
    return None

def modified_suffix_trie(text):
    G = nx.DiGraph()
    G.add_node('root')
    leaf_nodes = []
    # Your solution here
    ## BEGIN SOLUTION
    i = 0
    cnt = 1
    while i < len(text):
        current = "root"
        suffix = text[i:]
        for j,c in enumerate(suffix):
            if c == "$":
                G.add_edge(current,"[%d]"%i,label="%s,%d"%(c,i+j))
                leaf_nodes.append("[%d]"%i)
            else:
                n = modified_find_edge(G,current,"%s,%d"%(c,i+j))
                if n is None:
                    G.add_edge(current,str(cnt),label="%s,%d"%(c,i+j))
                    current = str(cnt)
                    cnt += 1
                else:
                    current = n  
        i+=1
    ## END SOLUTION
    return G,leaf_nodes

trie4,leaf_nodes = modified_suffix_trie("panamabananas$")
display(leaf_nodes)
show(trie4)
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
def suffix_tree_construction(text):
    trie,leaf_nodes = modified_suffix_trie(text)
    ## BEGIN SOLUTION
    to_remove = []
    for leaf_node in leaf_nodes:
        current = "root"
        full_paths = list(nx.all_simple_paths(trie,current,leaf_node))
        for full_path in full_paths: # this loop isn't necessary as there is only a single simple path
            if len(full_path) == 2:
                data = trie.get_edge_data(full_path[-2],full_path[-1])
                cw,i = data['label'].split(",")
                trie.remove_node(full_path[-1])
                trie.add_edge(full_path[-2],full_path[-1],label=cw)
            else:
                while True:
                    path = [full_path[0]]
                    for k,n in enumerate(full_path[1:]):
                        path += [n]
                        if len(list(trie.neighbors(n))) > 1:
                            break
                    if len(path) > 2:
                        data = trie.get_edge_data(path[0],path[1])
                        cw,i = data['label'].split(",")
                        i = int(i)
                        data = trie.get_edge_data(path[-2],path[-1])
                        cv,j = data['label'].split(",")
                        j = int(j)
                        trie.add_edge(path[0],path[-1],label=text[i:j+1])
                        for n in path[1:-1]:
                            to_remove.append(n)
                    else:
                        if path[-1] not in leaf_nodes:
                            data = trie.get_edge_data(path[-2],path[-1])
                            if "," in data["label"]:
                                cw,i = data['label'].split(",")
                                nx.set_edge_attributes(trie, {(path[-2],path[-1]): {'label':cw}})
                    if k+1 < len(full_path) and full_path[k+1] not in leaf_nodes:
                        current = full_path[k+1]
                        full_path = full_path[k+1:]
                    else:
                        break
    for n in set(to_remove):
        trie.remove_node(n)
    ## END SOLUTION
    return trie

tree = suffix_tree_construction("panamabananas$")
show(tree)
```

```python
to_adj(tree).index
```

```python slideshow={"slide_type": "skip"}
# Don't forget to push!
```
