---
layout: post
title:  "Parsing semver values from CVE records"
date:   2025-06-04 14:00:00 -0700
---

# Parsing semver values from CVE records

## Preliminaries

I've been pushing a schema change to CVE for the last half year or so. I started [a conversation about formalizing what a version type means](https://github.com/CVEProject/cve-schema/issues/362) in a CVE record back in November of last year. The tl;dr though is that versioning in a CVE record is chaos. From that conversation I spun up a [pull request to formalize semantic versioning](https://github.com/CVEProject/cve-schema/pull/371). That pull request has a lot going on, but the salient points are 
1. It introduces a new named version type `semver-2.0.0` so that record authors MUST opt-in to the new behavior.
2. The new behavior is quite strict and opinionated. This is for the benefit of the record reader.
3. The test suite is expanded to both demonstrate what correctness looks like and to guard against accidental changes.

The new behavior is that if one wants to submit a CVE record with a semver-2.0.0 then the version values MUST conform with the official semantic versioning regular expression. The ordering rules that semantic versioning defines SHOULD also be observed, but you can't enforce that in a schema.

Also, if you are reading this blog post and are not already involved in the PR please don't get involved. If you'd like to show your support please just 👍 the first comment.

## But why tho?

So, why make a new version type? Why do we need to add extra complexity of another version type? In short, because the current semver is chaos.

### A quick look at semver

Let's take a quick tour of the version type `semver` as it appears in the wild. Recall that a valid semver string contains three number with two dots which separate them eg. a.b.c where a, b and c can only be numbers. a.b is not semver compliant nor is a.b.c.d. There are build and pre-release extensions to semver, but for simplicity lets ignore those for the moment. If you want to double check me there please see:  
https://semver.org/#is-there-a-suggested-regular-expression-regex-to-check-a-semver-string  
which links out to the fantastic  
https://regex101.com/r/vkijKf/1/


So, what do we observe in the published data?

First a few examples. We have this

```
{
    "lessThanOrEqual": "FW950.90",
    "status": "affected",
    "version": "FW950.00",
    "versionType": "semver"
},
```
https://github.com/CVEProject/cvelistV5/blob/main/cves/2023/33xxx/CVE-2023-33851.json

Which comes to us from IBM and was published in 2023.

We've got this 
```
{
    "lessThanOrEqual": "FW10",
    "status": "affected",
    "version": "0",
    "versionType": "semver"
}
```
https://github.com/CVEProject/cvelistV5/blob/main/cves/2023/1xxx/CVE-2023-1150.json

From CERTVDE and 
```
{
    "lessThanOrEqual": "<=9.3.3.0",
    "status": "affected",
    "version": "ECOS 9.3.x.x: 9.3.3.0 and below",
    "versionType": "semver"
},
```
https://github.com/CVEProject/cvelistV5/blob/main/cves/2024/41xxx/CVE-2024-41133.json

from HPE. 

Three very different ideas of what "semver" is from three different organizations that have a good amount of experience with CVEs. Keep digging through the data and you'll find a number of other variations like the github style where inequality symbols are used in the version strings (sorry). If you want to parse an arbitrary record you end up with a tool that needs to do some heuristic test to identify a sub-pattern and then switches to one of a few dozen parsers to actually parse it. Also it needs to have healthy error handling for each case and for new cases that might exist in the future. It's a lot. I'm lazy so I didn't write any of that. Instead I wrote some code[^code] to help me get a handle on the scope of the problem. This code goes through every CVE record, pulls out version strings which were labeled as `semver` then tries to validate them against [an off the shelf semver parser](https://pypi.org/project/semver/). Is this a good parser? No idea[^1], but a heck of a lot of people use it, so it's an important parser.

Only 44% of CVEs pass the test[^2]. If you pull a "semver" version string out at random it has slightly better odds of failing validation than passing. Oof.

### Are the CNAs wrong?

Are those CNAs publishing semver values wrong? Should we go yell at them? In short. No. You or I or anyone else might look at the string `ECOS 9.3.x.x: 9.3.3.0 and below` and say 
> clearly this is not a semantic versioning compliant string 

but how does one validate correctness? How do we come to that conclusion? To validate correctness we need to go and look at [the docs](https://github.com/CVEProject/cve-schema/blob/a9e9fa9/schema/docs/versions.md#versions-and-version-ranges) and check the version string against the rules laid out. Alas, the docs don't say much about the issue. They introduce the versionType concept with

> The versionType is required when specifying ranges, because there is no single definition of “less than” for versions. Each different version numbering system has its own ordering rules. For example, in semantic versioning, 1.0.0-cr1 < 1.0.0-m1, while in Maven, the opposite is true. Example version types include maven, python, rpm, and semver. Another version type is git, described later.

Semver is mentioned by name and there's even a link out to semver.org when you view the markdown, however the docs follow up with 

> In any version range, the details of the version syntax and semantics depend on the version type, but by convention, "version": "0" means that the range has no lower bound, and a * in an upper bound denotes “infinity”, as in "lessThan": "2.*", which denotes a range where the 2.X version series is the upper bound, or "lessThan": "*", which denotes a range with no upper bound at all.

and then an example using `*` which is labeled as semver. `*` is not a valid character in a semver string. So, the docs have introduced us to the concept of version types, mention that semver is one of the anointed few and then show an example ruleset which is incompatible with what one would assume is ruleset for `semver`. A few more examples follow, but no formal rules are presented. No logic one could follow to check is one string or another valid. We lack a validation check and so we can't actually assert that any string is invalid. Any string is valid and so the type is unbounded. It is chaos.

### Who is the data for?

Part of the problem is that there doesn't seem to be a clear customer persona for who reads CVEs. In my opinion the primary reader of a CVE is someone who wants to fix/address/resolve/paper over whatever vulnerability is described in the CVE. A precondition to fixing a problem is knowing that the problem affects you and to resolve the question 
> does CVE X affect me? 

you need to know what products you use and what products the CVE applies to. There's a product naming problem there too, but lets ignore that for the moment and consider that we've matched on some product identifier. Do the versions match? Lets also restrict the problem space and assert that every piece of software uses semantic versioning. If we have a product match then we simply need to check if the versions of the product we use are contained in the version ranges listed as affected on the CVE record. That means parsing the record and that means making sense of the chaos that may be in it. We can't check if our version is in the semver version range, because we don't know what semver means in concrete terms. Even in this artificially simple scenario the potential complexity of the data is unmanageable. We have failed to make the data consumable to the reader.

## A path forward

If you look at the data captured in the CVE records you can find a number of patterns. Iterating them all isn't really worth it imo. The simple existence of independent patterns sharing a single identifier already tells us what we need to know. Each publisher is trying to express to their reader `Yo, this thing be broken!` with varying levels metadata and prose to enrich that core statement. The record format seems to have been designed in such a way that it didn't want to be a blocker. That on its own is fine, but that's what the `Custom` version type is for. The record format has failed to allow publishers to do better than unstructured strings. It needs to allow for better.

The CVE system is going through a lot right now. I don't want to comment much on that since I don't really understand it, but there's a lot of machinery out there in the world built on CVEs  that I care about. That machinery can and should be better leveraged to secure the software we all use but, it would be a shame to have to rebuild all of it. So, I suggest iterative improvement in small, measurable, and concrete ways. I think a workable step is to start allowing for a progressive precision. CNAs are publishing "semver" today. Let's give them a path to improve and perfect that. An approachable attitude could be to say 
> You want to use something else? Come define it and provide validation. 

A doable goal can be to provide validated input for the input we already get. We can make concrete one type at a time and compose these concrete types to make complex expressions.

So, let the grand ideas be discussed later. Let's work on some small patches.

`Thanks for reading`

----

[^1]: It has a good test suite 👍
[^2]: I pulled the data May 22nd 2025
[^code]:
	```
	import semver
	def find_semver_strings(cve):
		good = []
		bad = []
		with open(cve) as cve_file:
			cve_json = json.load(cve_file)
			cve_id = cve_json["cveMetadata"]["cveId"]
			if not cve_json.get("containers", {}).get("cna", {}).get("affected"):
				return good, bad
			for affects in cve_json["containers"]["cna"]["affected"]:
				for affected_block in affects.items():
					if affected_block[0] == "versions": # Not sure why the affected block decodes as a tuple
						for version_range in affected_block[1]:
							if "versionType" in version_range and version_range["versionType"] == "semver":
								if "version" in version_range:
									try:
										s = semver.Version.parse(version_range["version"])
										good.append(version_range["version"])
									except:
										bad.append(version_range["version"])
								if "lessThan" in version_range:
									try:
										s = semver.Version.parse(version_range["lessThan"])
										good.append(version_range["lessThan"])
									except:
										bad.append(version_range["lessThan"])
								if "lessThanOrEqual" in version_range:
									try:
										s = semver.Version.parse(version_range["lessThanOrEqual"])
										good.append(version_range["lessThanOrEqual"])
									except:
										bad.append(version_range["lessThanOrEqual"])
		return good, bad
	```