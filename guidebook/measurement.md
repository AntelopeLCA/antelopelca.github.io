


# observing fragments 

A spanner can be described in terms of its graph structure, but with the doesn't have any numbers in it yet- it is a "data-free" model in the sense that we don't know how big any of the flows are, until we "observe" them.  At this point, each fragment (except the reference) is: a specific flow which is either input or output of a specific parent.

The flow's role in LCA computation is determined by two properties: exchange value and anchor.

Each child fragment has an "exchange value". This is an *intensive* measure (amount per amount) that says "if the parent node is operated at a unit node weight, this flow has magnitude x".  So if you are making widgets and the machine used 4500 kWh to make 780 widgets, then the exchange value would be about 5.78 kWh per widget.

Second, the fragment has an "anchor", which is a next destination for the flow. This can be either a background dataset or another model.

The modeler must specify the exchange value and the anchor for each fragment, which is called "observing" it. Both exchange value and anchor can take on different values in different scenarios.

A documented, well-supported exchange value observation is the closest thing to an empirical grounding we can get in LCA.  

** Computing with fragments **

To "run" or compute the model, the spanner is traversed. The reference flow's observed magnitude is used to set the initial node weight. Then, each child flow is traversed, multiplying the parent's node weight by the exchange value to get the child node weight, repeated for its child flows until the leaf nodes are reached.

Leaf nodes that are not anchored to anything become cutoffs during traversal.  

The results of a traversal are reported in two ways:

 activity- lists the nodes visited and their node weights during the traversal
 cutoffs- the fragment inventory- lists the reference flow and cutoff flows and their magnitudes
 
LCIA is performed on the activity result. Given a set of LCIA indicators, Each anchored node computes its unit score for each indicator [these are typically cached].  Each node's impact is its node weight times its unit score, and the LCIA result is the sum of these values.

The inventory result is used for reporting the model results beyond the system boundary. When spanners are nested (anchor is another spanner), then the cutoffs are forwarded up into the the enclosing fragment and are matched to that fragment's child flows.  This happens at traversal time, allowing models to influence the run of other models via the recursive computation.

Sub-models that are linked as anchors get computed recursively during traversal.  When the traversal gets to the leaf nodes (nodes with no child flows), then the flows become cutoffs, which exit the system boundary and are forwarded up to the enclosing fragment.


## Offline tool

At the moment, Antelope is made up of tools that are run on a user's computer, and not on a cloud-based platform. 

