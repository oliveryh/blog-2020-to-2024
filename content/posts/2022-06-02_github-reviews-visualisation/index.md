---
title: "Github Review Visualisation with CytoscapeJS"
date: 2022-06-02T21:48:47+01:00
featuredImage: "/posts/2022-06-02_github-reviews-visualisation/header_image.png"
tags: ["python", "graphql", "github", "cytoscape", "javascript", "d3js"] 
categories: ["Software Engineering"]
draft: false
code:
  maxShownLines: 50
fontawesome: true
---

I recently learnt about [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law) and thought about examples of where a system is influenced by an organisations structure and vice versa. For many developers we may spend days at a time communicating only through the means of the humble code review. While the choices of who you might get a code review from, and how often, are often dictated by team structures, what about looking from the outside in.

So I thought, **Can you get a feel for how an organisation or a remote collaborative team is structured by how they request PR reviews?**

## What Do I Want To Build

This is what I'll ultimately be building, for those that like to see the end product before continuing to read:

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_cytoscape_02.html" height="750" width="100%"></iframe>

You can find the source code for the project [here](https://github.com/oliveryh/toybox/tree/master/visualisations/github-reviews)

Philosophical outcomes aside, I have some pretty clearly defined ideas of what I want this visualisation to look like:

- There is a node for each contributor to a code-base
- The edges between the nodes are sized to indicate how many reviews have been completed for a user
- The edges are directional as it's possible and not unusual for people to review each-others code

## Pulling the Data

For this example I'm using GitHub, and in GitHub's words, their [GraphQL API](https://docs.github.com/en/graphql) offers "more precise and flexible queries than the GitHub REST API. So that, plus the knowledge that I'd want to fetch multiple resources (users, pull requests, reviews), made this seem like a good start.

GitHub has a handy [Explorer](https://docs.github.com/en/graphql/overview/explorer) that let's you perform queries in the browser to experiment. With quick access to documentation in the side-bar, this also made it easy to iterate on an idea of the information I wanted to request.

``` gql
query ($owner: String!, $name: String!, $before: String) {
    repository(name: $name, owner: $owner) {
        pullRequests(last:100, before: $before) {
            pageInfo {
                startCursor
                hasNextPage
                endCursor
            }
            nodes {
                author {
                    login
                }
                number
                createdAt
                state
                reviews(last:10, states: [APPROVED, CHANGES_REQUESTED]) {
                    nodes {
                        author {
                            login
                        }
                        publishedAt
                        state
                    }
                }
            }
        }
    }
    rateLimit {
        limit
        cost
        remaining
        resetAt
    }
}
```

There are two parts of the query which aren't directly related to the resources I need. These are `pageInfo` and `rateLimit`. We'll need to paginate the query which I'll briefly go into next, but more importantly, GitHub has [resource limitations](https://docs.github.com/en/graphql/overview/resource-limitations) that we'll be hitting. For the volumes of data I've been consuming this limit has never been a concern, but we might as well collect/display this.

### Making the Request

{{<admonition info>}}
If you are doing this with a private repository, you'll need to [generate a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
{{</admonition>}}

While there are some more mature graphql clients for python, we can just use `requests` and send a `POST` request with the right variables

``` py
url = 'https://api.github.com/graphql'
variables = {
    'owner': owner,
    'name': name,
    'before': before,
}

api_token = '<SUPER SECRET TOKEN>'
headers = {'Authorization': 'token %s' % api_token}

requests.post(
    url, json={'query': query, 'variables': variables}, headers=headers
).json()
```

We'll look towards the [:(fab fa-github fa-fw): django](https://github.com/django/django/) project as an example. We can use `before: None` to get the first page.

```json
{
  "data": {
    "repository": {
      "pullRequests": {
        "pageInfo": {
          "startCursor": "Y3Vyc29yOnYyOpHON1oNVQ==",
          "hasNextPage": false,
          "endCursor": "Y3Vyc29yOnYyOpHOOSKiUA=="
        },
        "nodes": [
          {
            "author": {
              "login": "carltongibson"
            },
            "number": 15665,
            "createdAt": "2022-05-05T12:39:15Z",
            "state": "CLOSED",
            "reviews": {
              "nodes": [
                {
                  "author": {
                    "login": "smithdc1"
                  },
                  "publishedAt": "2022-05-16T18:42:33Z",
                  "state": "APPROVED"
                },
                {
                  "author": {
                    "login": "felixxm"
                  },
                  "publishedAt": "2022-05-17T07:49:26Z",
                  "state": "APPROVED"
                }
              ]
            }
          },
          ...
        ]
      }
    },
    "rateLimit": {
      "limit": 5000,
      "cost": 1,
      "remaining": 4997,
      "resetAt": "2022-06-06T22:09:37Z"
    }
  }
}
```

Right, that's a good start. Now let's paginate the request to get more results.

### Pagination

You'll notice an `startCursor` in our returned request. It seemed to make sense to get the results in reverse cronological order so we can make a second request with `before: <startCursor from request>`. That way we'd get the penultimate page of results, then the 2nd to last and so on.

I won't narrate all my code so feel free to take a closer look at how this was done at the project source code.

## Proposed Architecture

Now before we start jumping into the visualisation of the data we can now collect from the GraphQL API, there are some architectural decisions to make.

1. We don't want to make requests for new data every time we want to see the visualisation
2. We want to aggregate the data from GitHub

Persistant storage and aggregation is easily solved by the trusty sqlite database. For a small project like this especially so. Let's look at what our future architecture is panning out to be:

{{<mermaid>}}
graph LR
    gh(GitHub)
    json1(JSON)
    sqlite(SQLite Database)
    json2(JSON)
    html(HTML)
    browser(Browser Visualisation)
    gh -->|GraphQL| json1
    json1 -->|Ingest| sqlite
    sqlite -->|Aggregate| json2
    html -->|Loaded| browser
    json2 -->|Loaded| browser
{{</mermaid>}}

There might be some redundancy here, however once our data is loaded into the database, we have more freedom to iterate on the design.

## Visualisation Attempt #1: D3JS

This type of visualisation could be best described as a _node-based flow diagram_. I thought I'd start low-level with [D3.js](https://d3js.org/), partly because I could then ensure that I'd understand the low-level implementations of what I wanted to create. Not only that, but the [Mobile Patent Suits](http://bl.ocks.org/mbostock/1153292) block from Mike Bostock (the creator of D3) did a lot of what I needed, and I'd used it once before.

For this next part we'll consider the following basic scenario for illustration purposes:

- Alice has reviewed 10 PRs for Bob
- Bob has reviewed 5 PRs for Alice
- Charlie has reviewed 20 PRs for Bob

{{<mermaid>}}
graph LR
    Alice -->|10| Bob
    Bob -->|5| Alice
    Charlie -->|20| Bob
{{</mermaid>}}

Now let's give this a bit more interaction using D3.js

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_example_d3_01.html" height="300" width="100%"></iframe>

So far so good. I wanted to increase the edge width and the node size depending on the number of PRs reviewed. This was pretty easy for the edge width because I could just convert the constant width to a function

```js
var max_value = Math.max(...links.map(o => o.value))

var path = svg.append("g").selectAll("path")
  .data(force.links())
  .enter().append("path")
  ...
  .attr("stroke-width", function(d) {
      return Math.max(1, 10 * d.value / max_value)
    });
```

But when it came to altering the node sizes, D3 didn't automatically put the end of the arrow to the edge of the node. After a [little help](https://stackoverflow.com/questions/41226734/align-marker-on-node-edges-d3-force-layout/41229068#41229068) [from the internet](https://stackoverflow.com/questions/16660193/get-arrowheads-to-point-at-outer-edge-of-node-in-d3), I ended up with this.

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_example_d3_02.html" height="300" width="100%"></iframe>

I thought it looked a bit odd that the marker end arrows were the same size regardless of the edge width, so I also wanted to try fix this.

In the original example, we can see that different edge types (suit, licensing, resolved) result in different styled markers. D3 does this by creating a marker for each type and giving it an id like `marker-licensing` and then adding this attribute as the `marker-end` value to the edge.

Since we want to define an arrow size that might be unique to each edge, we can instead create markers for each edge id, and then use that id as an attribute for `marker-end`.

``` js
.attr("marker-end", function(d) {
  return "url(#marker" + d.id + ")";
})
...

getArrowWidth = function(d) {
  return Math.floor(5 * d.value / max_value) + 1
}

// Per-type markers, as they don't inherit styles.
svg.append("defs").selectAll("marker")
  .data(force.links()) // Changed from the types of marker
  .enter().append("marker")
  .attr("class", "marker")
  .attr("id", function(d) { return "marker" + d.id })
  .attr("viewBox", "0 -5 10 10")
  .attr("refX", 0)
  .attr("refY", 0)
  .attr("markerWidth", 20)
  .attr("markerHeight", 20)
  .attr("orient", "auto")
  .attr("markerUnits","userSpaceOnUse")
  .style("fill", "#666")
  .append("path")
  .attr("d", function(d) {
    return "M0,-" + getArrowWidth(d) + "L10,0L0," + getArrowWidth(d)
  })
```

After all that, we can then rewrite the path `d` attribute to create a narrower arrow for each edge depending on the value.

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_example_d3_03.html" height="300" width="100%"></iframe>

So I was pretty much done, all that remained was to test this out on a large enough repo, like [:(fab fa-github fa-fw): vue](https://github.com/vuejs/vue), and see how it fared.

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_example_d3_final.html" height="600" width="100%"></iframe>

Well that's not quite how I imagined it. At this point I'd spent so much time in the low-level minutia of tweaking `linkDistance`, `charge`, and arrow shapes that I thought there must be a better way / got what I deserved for starting on one of the lowest level visualisation libraries around.

It was time to go higher-level.

## Visualisation Attempt #2: CytoscapeJS

I arrived at [CytoscapeJS](https://js.cytoscape.org/) as an alternative batteries included node-based flow diagram library. I started with a bit of starter code and then changing the edge width, edge colour and node size was easy. Much like D3JS, you can return a function while defining these attributes. A large proportion of the code I had to write myself in the previous attempt was now being solved for me.

```js
const getEdgeWidth = function(value) {
    return Math.floor(4 * value / max_value) + 1;
}

const getEdgeColor = function(value) {
    if (max_change_request_rate == 0) {
        return '#888'
    }
    var red = Math.floor(255 * value / max_change_request_rate);
    var green = Math.floor(255 * (1 - value / max_change_request_rate));
    return "rgb(" + red + "," + green + ",0)";
}

const getNodeDiameter = function(node) {
    return Math.min(
        MAX_NODE_WIDTH,
        Math.max(node.outgoers("edge").reduce((acc, curr) => acc + curr.data("value"), 0), 0)
    )
}

$(function () {
  var cy = window.cy = cytoscape({
      ...
      style: [
          {
              selector: 'node',
              style: {
                  'width': function (node) {
                      return getNodeDiameter(node);
                  },
                  'height': function(node) {
                      return getNodeDiameter(node);
                  },
                  ...
              }
          },

          {
              selector: 'edge',
              style: {
                  'curve-style': 'bezier',
                  'width': function(ele){
                      return getEdgeWidth(ele.data('value'))
                  },
                  'target-arrow-shape': 'triangle',
                  'line-color': function(ele){
                      return getEdgeColor(ele.data('change_request_rate'))
                  },
                  'target-arrow-color': function(ele){
                      return getEdgeColor(ele.data('change_request_rate'))
                  },
                  'label': 'data(value)',
                  ...
              }
          }
      ],
      layout: {
          name: 'cose-bilkent'
      }
  });
});
```

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_cytoscape_01.html" height="750" width="100%"></iframe>

Cytoscape defaulted to a grid-style layout, but the [range of layouts they provide](https://blog.js.cytoscape.org/2020/05/11/layouts/) are comprehensive, and customisable. The `fcose` algorithm appears to be the best force-directed layout option that also attempts to reduce the amount of overlapping edges.

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2022-06-02_github-reviews-visualisation/viz_cytoscape_02.html" height="750" width="100%"></iframe>

There you have it! In one visualisation you're able to get a good idea about how open source and collaborative projects on GitHub are created. If you want to recreate this result, the source code for the project can be found in my [:(fab fa-github fa-fw): toybox](https://github.com/oliveryh/toybox/tree/master/visualisations/github-reviews) repo.