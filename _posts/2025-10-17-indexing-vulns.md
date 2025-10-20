---

layout: post

title:  "How and why we index vulnerabilities"

date:   2025-10-17 12:00:00 -0700

---

## Preliminaries

There are two types of vulnerabilities: those that are publicly known and those that are not. For the latter we can spend time searching/testing/researching to make them known and for the former we can collect, index, and enrich them so that interested parties can easily access them and take action based on them. It would be nice if we had some algorithm we could run and test on demand, but that might be difficult to construct. Anyway, if we want to ship more secure code, we can do some librarian-ing for the benefit of those interested parties until someone can make that algorithm. Libraries come in many forms, but one focused on software vulnerabilities probably falls into the "Research Library" category, and perhaps it makes sense to subdivide further depending on specific org goals and resources. Regardless, if we want to run a good library, we need to have a sense of who our readers are, what objects they care about, and what goals they have. We need scope.

 If we have a meaningful index of vulns we can check it against whatever we're shipping and know or reduce our risk exposure to at least a first approximation. We might be introducing code execution vectors or whatever in our code, but maybe we can at least have a little faith that we're only shipping our own flaws. That's the theory anyway. If we buy into this theory, then we need a meaningful index on which to work, and we need people to work it. Many approaches miss one or the other. You need both. One could imagine more advanced use cases with one or more indices that tell us the hows and whys of vulns in addition to the whats and wheres, but I would encourage restraint until there is a foundation to build on.

## Package oriented indexing 

For a while now we've collected advisories about known vulnerabilities, annotated them with useful metadata, clarified language, enriched them with meaningful context, and made them available to a reader base which has tastes and preferences. Package metadata in particular has proven to be pretty useful in that it allows a developer with an arbitrary codebase to check against a set of code that they're likely to have. The term Ecosystem has been used to communicate this metadata and for the purposes of this post I'll be limiting the discussion of the term to mean "a meaningful namespace for the purposes of communicating advisory information". The choice of which ecosystems we care about is also largely informed by popularity.


Take a look at the [most popular languages on github](https://github.blog/news-insights/octoverse/octoverse-2024/#the-most-popular-programming-languages)

![Octoverse 2024 report](https://github.blog/wp-content/uploads/2024/10/GitHub-Octoverse-2024-top-programming-languages.png?w=1400)

You'll note that most of them have a popular package registry (or registry system) that lends a namespace. These are well understood and make for easy ecosystems.

The problem with this approach is that it's inherently limited. Each ecosystem has its own sense of what a version is, what a package name can be, what can be inside, or where they're hosted. They're all quirky in their own ways and to scale you need to keep adding ecosystems which are decreasingly popular and thus increasingly hard to justify. To make things worse the more of the nuance you want to capture (as a benefit to the reader) the harder you make the job of the curator which may limit (in practical terms) the size of your corpus. Just as there's no sense in indexing using some new index if no one cares, so too it may not be worthwhile catering to every niche reader if the cognitive load of the curator is raised to an unworkable level. In practical application it may be worth simplifying your index for the benefit of your curator. Specific goals, timelines, and teams should inform choices about ecosystem specificity. Ecosystem design must consider operational constraints.

## Commit oriented indexing

Ecosystems don't need to be packages though. In theory an ecosystem is just a namespace, but which namespaces make sense? A curator needs a [domain](https://mathworld.wolfram.com/Domain.html) on which to curate and the choice of domain dictates the [range](https://mathworld.wolfram.com/Range.html) of the vulnerability management program. Further, the domain choice must be something the curator can reason about. One domain I think is worth exploring is the domain of the commit. While build systems can introduce (or resolve) bugs it happens to be the case that most bugs that manifest for the operator are in source code of the program and version control gives us a namespace for that.

It might sound daunting to go and read commits and try to determine what fixes what where and it certainly can be, but a shocking number of fix commits are really quite easy to spot. Like this one  
[https://github.com/pyca/cryptography/commit/1ca7adc97b76a9dfbd3d850628b613eb93b78fc3](https://github.com/pyca/cryptography/commit/1ca7adc97b76a9dfbd3d850628b613eb93b78fc3)  
They just tell you right there.  

Go search github for your favorite CVE number and marvel at the number of people fixing that particular problem out there (and good on them).

So, what can we do with these commits? Well, if we have a copy of the source repo we can ask the question "Does the repo at some point contain this fix?". Maybe we can't take as fact that not having the commit implies that the code is vulnerable to whatever issue that commit fixed, but we can be certain that if our codebase has the commit then the issue is fixed. This of course assumes truth and that's why you have curators, but let's explore this example a bit more

The commit  
[https://github.com/pyca/cryptography/commit/1ca7adc97b76a9dfbd3d850628b613eb93b78fc3](https://github.com/pyca/cryptography/commit/1ca7adc97b76a9dfbd3d850628b613eb93b78fc3)  
Which fixes an SSH certificate handling issue and is listed here  
[https://github.com/advisories/GHSA-cf7p-gm2m-833m](https://github.com/advisories/GHSA-cf7p-gm2m-833m)  

in this case we happen to have an introductory commit as well  
[https://github.com/pyca/cryptography/commit/aca8de845e751dd45fe4e48f8492f357d34d1861](https://github.com/pyca/cryptography/commit/aca8de845e751dd45fe4e48f8492f357d34d1861)  
shout outs to github user tiran for that  
[https://github.com/github/advisory-database/pull/2620](https://github.com/github/advisory-database/pull/2620)

Now cryptography is a project that has adopted a provenance solution called sigstore and that lets us trace back the origin of any published artifact like this one  
[https://pypi.org/project/cryptography/46.0.1/#cryptography-46.0.1-pp311-pypy311_pp73-manylinux_2_34_aarch64.whl](https://pypi.org/project/cryptography/46.0.1/#cryptography-46.0.1-pp311-pypy311_pp73-manylinux_2_34_aarch64.whl)  
back to a specific commit  
[https://github.com/pyca/cryptography/commit/e735cfc27502320101c130335c556394a125ba52](https://github.com/pyca/cryptography/commit/e735cfc27502320101c130335c556394a125ba52)  
via a build log  
[https://search.sigstore.dev/?logIndex=527528656](https://search.sigstore.dev/?logIndex=527528656)  


You could imagine building a system to iterate the specific artifacts, go inspect the upstream code at the specific commit for the release and check for both of introductory and fix commits. Now we can derive affected versions rather than having humans index on them. Maybe we could even track these artifacts as they get encapsulated in other objects. There could be other commit types which are interesting too, but intro and fix commits seem like enough to test the theory and to keep the task of the operator scoped. The down side here is that legacy artifacts don't have provenance information and so the readership for such an ecosystem is limited for now.

## Yes and...

Package oriented indexing isn't going anywhere and could even increase precision by breaking out parts of a package; `function X`, `class Y`, etc... to enrich whatever alerting or patching system actions on the data. Other indexing systems could also be interesting for different use cases, but I'm convinced that much meaningful software will find its way to a path of provenance, and so there will be readership for commit oriented advisories. I think there's potential at least. Indexing vulnerabilities on commits does require that the curator on the job can reason about code and coding practice, but I think you want that anyway. Research libraries are often staffed by researchers after all. Provenance is expensive, and that's a hard requirement as well, but the benefits of this approach could be huge. Arbitrary artifact support, freedom from namespace and version type enumeration, and less ambiguity in the curation process. Provenance is expensive, but maybe it's a cost we're going to pay anyway. I'd really like to see this idea develop further and to be prototyped to see if it holds up in practice. I really do believe it could be a force multiplier for vulnerability management.

```
Thanks for reading.
```