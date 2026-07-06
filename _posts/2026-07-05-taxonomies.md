---

layout: post

title:  "The unreasonable effectiveness of component taxonomies in Vuln Mgmt"

date:   2026-07-05 12:00:00 -0700

---

## Flaws exist in things 

Flaws exist in things. If there is no "thing" there can be no flaw as there is nothing at all. I hope you agree that if there is no product/component/library/whatever then by definition there can be no flaw or vulnerability. If not we'll need to grab food sometime and chat, but I'll assume you the reader agree for the time being.

The statement might sound obvious, but in the world of vulnerability management we do not require that the "thing" in the statement of a vulnerability be defined. Advisories are not required to unambiguously state the product in which a vulnerability exists. The CVE program requires that there be an affected product, but that product is not constrained. When there is no constraint on what a name means there is room for it to plausibly mean many things and thus the name is ambiguous; the name `latex` may mean one software specific project to academics writing research papers and a very different thing to patrons of the fashion industry. The name `latex` when presented without constraining context is ambiguous. The sciences address this issue of classification/categorization with systems called [taxonomies](https://en.wikipedia.org/wiki/Taxonomy) where terms have meaning in some contexts.

## The business of vulnerability management

We, as an industry, do require that an advisory be acted upon. Compliance regimes around the world drive the business of vulnerability management and by extension they have driven the design of it. These compliance regimes all basically work from the premise that there exists a big list of vulnerabilities and those organizations which want to be compliant must have an answer for each of these vulnerabilities. That list is the corpus of CVE records for most of the world and the most common answer for any given organization is `CVE-123 does not affect us because we don't use the software in question`. If the organization does use the particular software then the answer becomes more complex.

These compliance regimes may impose audit requirements, fines and in the worst cases legal action should the management of vulnerabilities not be satisfactory. If we know that we do not use a given component we can very quickly assert that we are not affected by some vulnerability. If we know we do use a component then we may face a large remediation timeline, but we can at least route the advisory to the relevant party or parties and begin a deeper analysis. If we're uncertain we're in a pickle and the current data ambiguity leaves us drifting in uncertainty.

## Are you affected and how can you know?

Pretend we have a complete, accurate and timely index of all the software components we use in our org and at any given moment we have high confidence that we can query that index and get a reliable answer. We can ask `Are we using apache httpd v 1.2.3?` and get a quick 👍 / 👎. If we have such an index then we can ask `Are we using {product} {version}` and populate the `product` and `version` values from what's reported in some CVE. If the answer comes back 👎 then we can kick off some automation to inform our stakeholders. If it comes back 👍 then we can look up the relevant internal project owners, and inform them. The project owner is the most relevant party to deal with the hyperspecific details that a vulnerability report tends to be populated with. It may be the case that the vulnerability report is real, has been accurately delivered to the project owner and yet it's not an issue because the particular piece of the vulnerable software isn't used or is shielded in some way.

This 👍 / 👎 ignores a lot of nuance in product naming and in practice it's easier to make a system that only returns 👍 when there is a match. The 👎 side of things requires proving a negative which is generally a pretty hard thing to do. Either way though this 👍 / 👎 requires both an index of what components an organization uses internally as well as an unambiguous indication of what "thing" the vulnerability exists in.

## Building the 👍 machine

Lets consider some examples of what's in CVE. The affected block is a required field in the CVE schema which is intended to make the affected software machine readable. Here are a few examples of what it looks like in practice.
```json
"affected": [
    {
        "defaultStatus": "unaffected",
        "product": "Apache Doris-MCP-Server",
        "vendor": "Apache Software Foundation",
        "versions": [
            {
                "lessThan": "0.6.0",
                "status": "affected",
                "version": "0.1.0",
                "versionType": "semver"
            }
        ]
    }
],
```
https://github.com/CVEProject/cvelistV5/blob/a295321f4cd4549cf20895104f55af4957b2bf71/cves/2025/58xxx/CVE-2025-58337.json#L15-L29

```json
"affected": [
    {
        "vendor": "tltneon",
        "product": "lgsl",
        "versions": [
            {
                "version": "< 7.0.0",
                "status": "affected"
            }
        ]
    }
],
```
https://github.com/CVEProject/cvelistV5/blob/a295321f4cd4549cf20895104f55af4957b2bf71/cves/2024/56xxx/CVE-2024-56361.json#L65-L76

```json
"affected": [
    {
        "vendor": "hupe13",
        "product": "DSGVO snippet for Leaflet Map and its Extensions",
        "versions": [
            {
                "version": "0",
                "status": "affected",
                "lessThanOrEqual": "3.1",
                "versionType": "semver"
            }
        ],
        "defaultStatus": "unaffected"
    }
],
```
https://github.com/CVEProject/cvelistV5/blob/a295321f4cd4549cf20895104f55af4957b2bf71/cves/2026/4xxx/CVE-2026-4389.json#L20-L34

These CVEs all list a product and are technically compliant with the requirements of the program, however these are not product names that are unambiguous. There is no service or index that these product names can be looked up in to cross check against. These names all require that the reader have an intuition about what they mean and this implicit requirement limits the scalability of the practice of vulnerability management. In order for these CVEs to be routed to the relevant project owners there must be an intermediate party which provides a mapping service.

## A tale of two mappings

Given that we can't use the values from a CVE directly it's worth considering the workflow with and without an internal taxonomy. With a taxonomy we can read the incoming CVE, map it to our internal taxonomy and alter the flavor text as need be. At that point we are done with the CVE and some tooling can reference the annotated data, get an alert in front of anyone using the offending product and get them to either stop or to document why they continue to use it. We can scan our projects with whatever frequency we desire and so if either the CVE text or our projects change state we can create new alerts or remove invalid alerts within the time delta of our scan frequency. For really important updates we can do on demand scans.

If we don't have a taxonomy we still need to inform our stakeholders of our affectedness status. By definition we are unable to construct a scanner based system because we have no language with which to annotate the record, so what can we do? We could do a search through all of the projects internal to our organization and pass some judgement. Maybe look at imports or manifest files or something. If we see something we say something, make a paper trail, and job done. Job done unless/until there is either a change in the CVE record or in our projects. Any state change may require expensive reanalysis. Reanalysis also introduces the situation where the second analysis may differ and different users may be altered. We introduce false positive and false negative outcomes because by definition we do not describe what is affected; we inform those we believe are affected at some moment in time.

Consider also the situation that the front line is put into when there is no component taxonomy. Front line analysts make assertions of affectedness. X is affected, Y is not. When X and Y have no defined meaning the analyst is fundamentally guessing and they will be unable to defend their assertions if challenged. This leads to a situation where those put on the front line make assertions, the side affects of those assertions drive alerts which are bound to be fuzzy because there is no definition of correctness, some percentage of the users who recieve false positive alerts get frustrated, demand satisfaction, and the analyst is forced to defend the assertion `component X is affected`. Component X has no clear meaning, cannot be inspected, and claims of affectedness cannot be interrogated. The front line must defend the indefensible and while they may win a skirmish they will lose the war. Over time the front line tunes out, burns out and drops out.

## Consider the production side of the equation

So far we've only considered the consumption side, but advisories need to be produced before they can be consumed and it turns out that a taxonomy makes that easier as well! Let's say we've found a bug in some software; absent any guidance it's one can imagine a number of ways to express what is affected. It could be the marketing name for some end user software, Red Hat Enterprise Linux 7 for example. It could be some python package used in that end user software, the Pillow image library maybe. Or it could be some specific lines of code. All of these could be considered correct and as a reporter you have a choice of how to express the "where" of the vulnerability. Choice can be nice, but it can also induce decision paralysis and at a systemic level it will induce inconsistency in the data. Having one or many taxonomies to use to express the "where" of the vulnerability simplifies the choice by removing ambiguity and it ensures that there is an audience for the report. Faster easier reports with known audiences. Taxomonies are win-win across the publication pipeline.

## Wrap up 

Taxonomies don't have to be perfect to be useful. If we consider the package taxonomy used by GitHub we can notice that it
1. Has obvious gaps for well known package registries
2. Does not gracefully handle the nesting of software. Multiple affected entries are required
3. Does not completely handle version strings for the ecosystems it does support

and yet flaws can be delivered immediately following publication, flaws can be inspected, interrogated, and understood; and a complete set of flaws can be retrieved for any component in the taxonomy.

The CVE specification supports the package url specification and the purl specification is an open community that welcomes growth. If this is the sort of thing you're interested in come join the [PURL community](https://github.com/package-url/purl-spec), expand it with your pet taxonomy, or help clarify and disambiguate the ones we already have. If you're not a specs person then start using the purl spec. Add them to your CVEs if you're a bug hunter, index with them if you're a vulnerability manager. We can't let perfect be the enemy of good. We need visibility into our vulnerabilities and we need more librarians in the world of vulnerability management in order to achieve that.
