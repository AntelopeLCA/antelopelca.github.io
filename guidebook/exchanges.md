---
layout: page
title: Guidebook -- Exchanges
permalink: /guidebook-exchanges
---

![A life cycle inventory "fragment" -- an observed material exchange](/assets/img/exchange.png)

# Exchanges

Exchanges of industrial material flows are at the core of every product model. Every life cycle inventory model can be depicted as a directed graph with nodes connected by edges.  Each edge represents the "exchange" of some flow between two nodes. Every exchange has the following parts:

 1. A "parent" node or reference node, which is the *thing that causes the exchange to occur*
 2. A "flow", which is the *thing being exchanged*
 3. A "direction", which is "input" or "output", stated with respect to the parent
 4. A "target" or terminal node that is the *partner* to the exchange

In addition to these required features, each exchange has some information that is not inherently part of the graph, and these are the observed characteristics of the exchange:

 5. The *magnitude* of the flow
 6. The flow's properties

Whereas items 1-4 are properties of the model graph and are non-numeric, items 5 and 6 are both quantitative (numerical, measured) properties of the individual flow or exchange.

## Observing exchanges

The defining feature of an exchange is that it can be "observed" or measured by a person who is "standing at" the parent node. To observe an exchange means observing three things:

 * *that* the exchange occurred (including when and where it occurred)
 * the *magnitude* of the exchange (how much of the flow could be observed)
 * the *target* of the exchange (what was on the "other end" of the flow)

"Observation" is a crucial aspect of modeling in Antelope-- only a person who is familiar (and reasonably proximate) to the exchange can credibly observe it.  Often an LCA practitioner is *neither familiar nor proximate* to the exchanges they are modeling, and they must therefore ask for this information secondhand, typically through preparing a questionnaire or data collection survey.  

Observations of material flows being exchanged between industrial activities is the empirical basis for all of LCA. LCA cannot be done without these observations.  

A key implication of this is the following:

> If there is value in *accuracy* in LCA, then the source of that value is the accurate *observation* of material flow exchanges.


## Exchange Targets

The existence of an "exchange" implies the existence of **two nodes** which are the parties to the exchange.  These nodes are  like "points" or "locations" in the model that correspond to activities of some kind.  Each exchange has a "parent" node and a "target" node.  The "parent" node is the activity that is *responsible* for the exchange occurring, and the "target" node is therefore an activity that is being *caused to occur* because of the parent. 

If the target for a given exchange is "nearby" according to the modeler, then the two nodes can be described as part of the same spanner graph.  These exchanges are called "foreground" exchanges.  The modeler can change positions from the "parent" to the "target" and continue to observe exchanges.  

If the target is "far away", then the exchange is called a "background" exchange. The modeler cannot necessarily observe the exchange anymore, and must rely on third-party information to describe the flow thereafter.  The "source of truth" for this remote end of the exchange is called the "anchor" for the exchange.  Here, the modeler's work ends with the selection of a credible source of information describing the flow. The most frequent source of credible third-party information about background flows is a specific reference flow selected from a background LCI database.  

If the target for an exchange is not an industrial activity but an environmental context, such as an emission into the air, then the exchange is an "elementary" exchange. The LCA modeler's work ends at this point because the flow can no longer be observed.

If the target for the exchange is unclear or not specified, then the flow becomes a "cutoff" exchange.  These cutoffs become important in model reuse later on.

# ExchangeRefs

Exchanges are tracked in Antelope in the form of `ExchangeRef` objects, which include the following properties:

 - `process`: the parent process (`ProcessRef`) that generates the exchange
 - `flow`: the flow (`FlowRef`) being exchanged
 - `direction`: a string (`Input` or `Output`) relative to the parent process
 - `type` a string indicating the type of termination; one of (`reference`, `node`, `context`, `cutoff`)
 - `termination`: the "other end" of the exchange: either
   - the `external_ref` of a linked process (a string) that includes the flow as a reference exchange
   - a `Context` if the exchange is not linked to a providing process
   - `None` if the exchange is a cut-off or a reference exchange.
 - `value`: the magnitude of the exchange
   - if the exchange is an `UnallocatedExchange` then this will be the magnitude of the un-allocated flow
   - if the exchange is an `AllocatedExchange` then this will be the magnitude *per unit* of the allocated reference
 - `comment`: a text-based comment (implementation varies by provider)
 - `locale`: the locale of the exchange (deprecated: this will be changed to `location' in the future)
 - other properties. ExchangeRefs can be assigned other properties at creation.


[Home](/guidebook/)
