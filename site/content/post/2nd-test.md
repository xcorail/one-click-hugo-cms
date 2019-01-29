---
title: This is a 2nd test
date: 2019-01-29T11:03:21.488Z
description: 'blurb blurb'
image: /img/blog-chemex.jpg
---
The best predictor of code quality is simply how much code review occurs within a project (see [previous post](https://lgtm.com/blog/does_review_improve_quality)). It's vital to foster a positive review culture that encourages more pairs of eyes, rather than more time staring.

So more independent reviewers looking at code is good. But their efforts can be organised in different ways. In particular, I want to investigate whether the ‘hierarchy-ness’ of the review process impacts code review. Hierarchy means there is a ranking or ordering of members in a group and it’s not necessarily a bad thing. In fact, it, naturally arises in many systems, including among [Wikipedia editors](https://appliednetsci.springeropen.com/articles/10.1007/s41109-017-0043-2). How does hierarchy between code reviewers impact pull requests? 

## Who reviews who?

Developers have many separate - and sometimes conflicting - ideas for a code base. So  developers propose their change ideas in a pull request, which are then reviewed by other developers. Pull requests are great for spotting bugs, discussing improvements, removing vulnerabilities, or simply saying thanks. Once a pull request gets the green light, the changes are merged into the code base.

_Review comments_ reference specific code changes within a pull request, and are directly related to the source code (unlike general comments). To obtain data on review comments, I analyzed the pull requests of over 1000 prominent Python, Java and Javascript projects on GitHub. 

I define the ‘opener' as the developer who opened the pull request, and I define the ‘reviewer' as someone who wrote a review comment on a pull request. For each project, I construct a graph of all interactions between reviewers and openers. I add an edge between developer A and developer B if either has left review comments on the other's pull requests. I take into account if developer A reviews developer B more or vice versa by making the edges _directed_. The image below illustrates a review graph with arrows between developers referring to reviews.

![](images/review_hierarchy_1/who_reviews_who.png)

For example, if Jim has left a total of 3 review comments on pull requests opened by Sarah, but Sarah has left 7 review comments on Jim’s pull requests, then I add a directed edge from Sarah to Jim. The final result is a graphical representation of who-reviews-who.

## A measure of hierarchy

I now have a graph representing the interactions between reviewers, generated from a project’s pull request data. How do I quantify the ‘hierarchy-ness’ of that project? 

Generally, when people think of hierarchy, images of pyramidal structures like food chains come to mind. At the top of the pyramid, you have the apex predator, who feeds on organisms below, but isn't devoured by any organism above. Organisms at the bottom of the food chain don’t feed on anyone. This who-eats-who example is a highly hierarchical system.

Now consider the who-reviews-who graph for a project where only one developer does the majority of reviewing. The other developers in the project merely open pull requests that get reviewed by the ‘apex’ reviewer. I want a hierarchy metric that quantifies this kind of project as having high hierarchy compared to, say, a project where everyone reviews everyone else.

No single metric perfectly captures hierarchy, but the _flow hierarchy_\[^4] is my metric of choice. Formally, a graph’s flow hierarchy is the percentage of edges not involved in cycles. A graph in which every node is connected to every other node, has a flow hierarchy of 0%, since all its edges are involved in cycles. On the other hand, a directed acyclic graph (i.e. a tree), lacks any cycles, and hence its flow hierarchy is 100%. 
Higher flow hierarchy, in general, implies a more hierarchical review process.

![](images/review_hierarchy_1/flow_hierarchy.png)

## Flow hierarchy in action

How does flow hierarchy metric relate to _real_ projects? Let’s look at two Github projects to see if their review structures have visible differences.

The first project is the jupyter/jupyter repository. Its review graph is shown below\[^6], with node sizes proportional to the number of review comments made by that developer, and edge widths proportional to the sum of review comments between two developers.

![](images/review_hierarchy_1/jupyter.png)

Despite a few developers standing out with more reviews, the developers of jupyter tend to review code by many other developers, as implied by the many overlapping edges in the centre of the graph. No matter how you arrange the developers of jupyter, you can't ‘untangle' the maze of overlapping edges in a sensible way. This ‘tangled-ness' is characteristic of low hierarchy review structures, and indeed the flow hierarchy for this graph is 33%.

Let’s now consider a project with a high flow hierarchy value. Below is the review graph for miracle2k/webassets, with a flow hierarchy value of 92.9%. Straight away we see few edges overlap, and correspondingly few cycles. 

Interestingly, two main developers act as central hubs, and the other developers are generally isolated from everyone else (most nodes have only one edge). This review graph is a great example of a high-hierarchy project.



## Less hierarchy, more reviews!

We know that one of the strongest indicators of code quality is simply how many pairs of eyes review each pull request (see the [previous blog post](https://lgtm.com/blog/does_review_improve_quality). But what kind of review culture encourages more pairs of eyes to review in the first place? Spoiler alert: the less hierarchical ones!

To quantify how many pairs of eyes are reviewing in each project, I calculate the number of reviewers _per opener_. This measure is more independent of project size than simply counting the number of reviewers. Higher reviewer-opener ratios are linked to higher code quality scores.

Using pull request data from 1000+ Github projects, I find a negative correlation between the flow hierarchy and the reviewer-opener ratio ([r](https://en.wikipedia.org/wiki/Spearman%27s_rank_correlation_coefficient)=-0.45, [p](https://en.wikipedia.org/wiki/P-value)<1e-6). In other words, less hierarchical projects have more reviewers. This makes sense: if there are no barriers to who you can review, then developers are more likely to engage in the review process\[^7]. 

![](images/review_hierarchy_1/hierarchy_and_reviewers.png)

To plant the seeds of a healthy review process, start fostering a less hierarchical review culture within your project. Easier said than done? Probably, but if you succeed, there are plenty of benefits. As we shall see, improved code quality\[^8] isn’t the only bonus you get.

Note that once the effect of simply “more reviewers” is removed, I don’t observe any direct code quality benefit from a less hierarchical review process. However, lower hierarchy has two other big bonuses... 

## Less hierarchy, faster pull requests!

Quick turnaround times for pull requests are desirable. Maybe you have a project deadline or other developers need to build on your pull request; either way, we don’t want  pull requests to stall.

What kind of project environment encourages a quick turnover?

I define the _pull request duration_ as the time between the opening of the pull request and when it’s merged or closed. For each project, I calculate the median duration across all pull requests. Pull requests that are never closed are excluded from the analysis.

I find that less hierarchy is linked to faster pull requests, as indicated by the positive correlation between flow hierarchy and median pull request duration (r=0.17, p<1e-6). This relationship holds for all three languages (see figure below), with Python and Java projects showing a stronger relationship compared to Javascript. Therefore less hierarchy correlates with faster processing of pull requests, possibly through more efficient transfer of information, or more distributed workloads.

The effect is substantial: projects with flow hierarchy values above 50% take on average 38% longer to process pull requests relative to projects with less than 50% flow hierarchy.

![](images/review_hierarchy_1/hierarchy_and_duration.png)

But perhaps hierarchy is unrelated to faster pull requests, and the effect is simply due to more reviewers? No:

Even when I remove the dependence (linear or quadratic) of the flow hierarchy on the reviewer-opener ratio, the positive correlation between the flow hierarchy and duration remains statistically significant. This implies that given two projects with the same number of reviewers, less hierarchy is indeed linked to faster pull requests. 

Of course, we don’t want to sacrifice the thoroughness of code review for faster pull requests. Reassuringly, I find no relationship between [LGTM’s code quality score](https://lgtm.com/blog/code_quality_1) and pull request duration. So faster turnaround doesn’t imply a decline in review quality. 

## Less hierarchy, more merging!

Pull requests can contain a sizable chunk of work, and if that work never gets merged, it’s a pity since work has been wasted. Of course, you don’t want to merge bad work, but if you can easily salvage it and turn it into good work, that’s certainly preferable. I find that, on average, a higher percentage of merged PRs do not correlate with reduced code quality (again measured by LGTM). 

This suggests that a higher merge rate usually means better utilization of developer effort, as opposed to lower review standards.

So you should aim for higher merge rates, but how do you get there? Let’s see how hierarchy within the review process impacts how many pull requests are merged. For each project, I calculated the percentage of pull requests merged, and then correlated that with the flow hierarchy.

![](images/review_hierarchy_1/hierarchy_vs_merge_success.png)

Again, less hierarchy is a good thing, since less hierarchical projects merge more of their pull requests (r=-0.35, p<1e-6). The relationship holds within all three languages and remains statistically significant even after removing the dependence of flow hierarchy on the reviewer-opener ratio. 

So not only does less hierarchy lead to faster pull requests, but those pull requests are more likely to make it into the code base. 

## What does this mean for you?

Do we want higher code quality? Do we want to merge pull requests faster? Do we want to avoid wasted effort? Of course we do! Then the empirical data suggests we should aim for less hierarchy in the code review process, so everyone gets a chance to review everyone else.
The everyone-review-everyone sentiment is shared by Kelly Sutton, who emphasises this point in his blog post\[^9] on code review:

> I find that distributing reviews around to different members of the team yields healthier team dynamics and better code. One of the most powerful phrases a junior engineer has in a code review is, ‘I find this confusing.’ These are opportunities to make the code clearer and simpler.
> Spread those reviews around.

Good consequences will follow from an inclusive review culture, where reviewing becomes mutual between all developers.  Less hierarchy is an effective method to improve your code review process.

<script async class="speakerdeck-embed" data-slide="35" data-id="bcbfc6b6929b44eb83f530693cd53ce8" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

Maybe you’re ahead of the game, and everyone within your project already reviews everyone else; is there anything else you can do? LGTM’s automated code review is like getting another pair of eyes for free. LGTM will review pull requests opened by anyone, no matter what part of the code base they’re working on. So enabling LGTM’s automated code review instantly gives your review process a boost. 

```javascript
const path = require("path")
const _ = require("lodash")

exports.createPages = ({ actions, graphql }) => {
  const { createPage } = actions

  const blogPostTemplate = path.resolve("src/templates/blog.js")
  const tagTemplate = path.resolve("src/templates/tags.js")

  return graphql(`
    {
      allMarkdownRemark(
        sort: { order: DESC, fields: [frontmatter___date] }
        limit: 2000
      ) {
        edges {
          node {
            frontmatter {
              path
              tags
            }
          }
        }
      }
    }
  `).then(result => {
    if (result.errors) {
      return Promise.reject(result.errors)
    }

    const posts = result.data.allMarkdownRemark.edges

    // Create post detail pages
    posts.forEach(({ node }) => {
      createPage({
        path: node.frontmatter.path,
        component: blogPostTemplate,
      })
    })

    // Tag pages:
    let tags = []
    // Iterate through each post, putting all found tags into `tags`
    _.each(posts, edge => {
      if (_.get(edge, "node.frontmatter.tags")) {
        tags = tags.concat(edge.node.frontmatter.tags)
      }
    })
    // Eliminate duplicate tags
    tags = _.uniq(tags)

    // Make tag pages
    tags.forEach(tag => {
      createPage({
        path: `/tags/${_.kebabCase(tag)}/`,
        component: tagTemplate,
        context: {
          tag,
        },
      })
    })
  })
}
```

Image Credits: Photo by [Robert Bye](https://unsplash.com/photos/QnD4fdaezTI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

\[^1]: Treewidth decompositions are performed on undirected graphs, but there are attempts to extend the definition to directed graphs.
\[^2]: An example of a random walk hierarchy measure: https://www.nature.com/articles/srep17994.
\[^3]: Google’s PageRank algorithm is based on a random surfer traversing links to different websites, so is not separate from other random walk based methods.
\[^4]: The implementation of flow hierarchy in NetworkX: https://networkx.github.io/documentation/networkx-1.9/reference/generated/networkx.algorithms.hierarchy.flow_hierarchy.html, which provides a reference to the original paper.
\[^5]: Some hierarchy measures, like the treewidth from a treewidth decomposition, produce discrete values where most projects have a treewidth of 1, 2 or 3; this is not helpful for quantifying variations in hierarchy between projects.
\[^6]: The low-hierarchy graph was plotted using NetworkX with a Kamada-Kawai layout.
\[^7]: Causation can also run in the opposite direction: projects with a high proportion of reviewers could tend towards less hierarchy.
\[^8]: It should be noted that once the effect of simply “more reviewers” has been removed, I could not observe any additional code quality benefit to having a less hierarchical review process. 
\[^9]: [Kelly Sutton, 8 Tips for Great Code Review](https://kellysutton.com/2018/10/08/8-tips-for-great-code-reviews.html)
