# Methodology 

This is just a bit of an experiment, I've been building these trees with a number of friends to map out how attackers might try to win against their services or companies. The result of this exercise is that a tool and a methodoloy are being developed at the same time.

There is value in having a detailed datastructure, containing more information than just the node-name, it's going to be useful for future simulation and analysis work, however, there's equal value in presenting a nice, fast mechanism for generating graphs that make engineering conversations about security go faster. For that reason a lot of defaults and shortcuts have been added to make basic graph generation easier. This should be followed by detail later. 

## Step 1. Sketch the outline of your current reality
Assemble a team (Security, Red Team, Service owners). Agree on a starting position and a goal. Use the best judgement of the team to constrain the start and the goal to a conversation that can be had in an hour. Avoid building "monster" graphs, these graphs are most useful when contrained to something that will fit on a screen or in a document without scrolling.

Don't worry if your graph is getting to big as you build it, it's pretty easy to break a graph into sub-graphs after the fact.

Once you're agreed on the goal and objectives start working through a process of postulating "What will the attacker do next" with a strong bias towards low-cost, low-effort attacks, saving the complicated stuff for when you've exhausted the simple stuff.

In real-life the easiest way to do this is on a whiteboard, with a stack of colored post-it's to represent types of node. That's not the reality for many of us who work remote who need a way to capture the graph as we talk through the problem. To make quick graphing possible each node has the ability to create a child and connect to it in one step. 

Imagine this simple graph:
The start point is "bad guy" and the goal is "break window". Our badguy is going to look for a rock, discover one and throw it at the window!

A simple graph for this might look like this:
![Graph generated by running example_smasher_v1.py](images/example_smasher_v1.png?raw=true)

This graph can most simply be written like this:
```
with Renderer(root = "Bad Guy", goal = "Break Window") as graph:
    graph.root.action("Look for rock") \
        .discovery("Shiny Rock") \
        .action("Throw Rock") \
        .connectTo(graph.goal, "Smash!")

    # call to renderer
```

There's value in being able to graph this quickly, and it's certainly easy to read. However, this graphing style only works when each node has only one child. A more flexible way to do this is to keep track of each object. The code below will generate exactly the same graph but maintain a handle to each node that we can use.
```
with Renderer(root = "Bad Guy", goal = "Break Window") as graph:
    looks = graph.root.action("Look for rock")
    findsRock = looks.discovery("Shiny Rock")
    throws = findsRock.action("Throw Rock")
    throws.connectTo(graph.goal, "Smash!")

    # call to renderer
```

When written this way, it's easier to attempt our second pass - adding mitigations.

## Adding mitigations
There's two types of mitigations described in `models.py` so far; "Block" and "Detect". Blocking indiciates we have a control in place that will stop a certain action. Detect won't stop the bad thing from happening but will detect it, and presumably send that detection somewhere useful like a SEIM. 

Each type of mitigation has a state of `implemented` which can be either `True` or `False`. When we are adding mitigations, a lot of the time we want to postulate that "oh, we could add X here" - there'll be a cost to that (CapEx, OpEx, Complexity, Maintentance etc) as well as other factors that we'll consider later in step 3, however for this step, we only want to add these mitigations in the right places and see how they effect our attacker. 

It's up to those on building their attack graph to decide if a mitigation will completely stop an attack or only partially. Block nodes can have forward `actions` and `discoveries` just like any other node in the graph.

So lets update the example and add a potential mitigation, one that's unimplemented; a rock melting laser (pew pew!):
![Graph generated by running example_smasher_v3.py](images/example_smasher_v3.png?raw=true)

The code for this graph looks similar to previous but with the new `laser` parameter added.
```
with Renderer(root = "Bad Guy", goal = "Break Window") as graph:
    looks = graph.root.action("Look for rock")
    findsRock = looks.discovery("Shiny Rock")
    throws = findsRock.action("Throw Rock")
    laser = throws.block("Rock Melting Laser", implemented=False)
    throws.connectTo(graph.goal, "Smash!")

    # call to renderer
```

Just as we are writing our model, one of the building architects points out that recently the building received unbreakable-glass upgrades; a blocking control. So we add that to our graph, this time as a control that _is_ implemented. Because we belive this is 100% unbreakable, we no longer see a path to the goal

```
with Renderer(root = "Bad Guy", goal = "Break Window") as graph:
    looks = graph.root.action("Look for rock")
    findsRock = looks.discovery("Shiny Rock")
    throws = findsRock.action("Throw Rock")

    laser = throws.block("Rock Melting Laser", implemented=False)
    bonk = throws.block("Unbreakable glass", implemented=True)

    # call to renderer
```

The result is the following graph, which shows that the risk has been effectively mitigated by the new glass, and we probably don't need the lasers after-all...

![Graph generated by running example_smasher_v4.py](images/example_smasher_v4.png?raw=true)

# Step 3. Adding metadata
This project seeks to be more than a quick way to generate graphs. The choice to separate the *model* from the *view* was a deliberate one. In the third stage of this methodology the team is encouraged to add metadata to the nodes, that includes: _cost_, _probability of success_, _sensitivity_, _mitre attack stage_ etc. 

The idea is to model out complex sets of capabilities, attacks and mitigations, both potential and real. Eventually we'll be able to join models together into "big-ugly-models" where the graphs will be largely unintelligble without a wall-plotter but we'll be able to ask questions like "If I have $1m to spend, where should I spend it to improve security" or "How likely am I to suffer a breach of goal _x_ in the next 6 months" ...

More to come on step 3 ...