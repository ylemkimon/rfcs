- Start Date: 2020-08-07
- RFC PR: (leave this empty)
- Yarn Issue: (leave this empty)

# Summary

Preload `dependenciesMeta.built` for popular packages.

# Motivation
Build scripts such as `postinstall` of popular packages can be used as an
exploit, as seen in [`eslint-scope`](https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes).
There are also instances of packages being compromised, such as
[`event-stream`](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident).

Due to the nature of NPM ecosystem, it's almost impossible to review the
whole dependency tree.

# Detailed design

A list of packages and their settings can be maintained and preloaded onto
Yarn builds, like [@yarnpkg/plugin-compat](https://github.com/yarnpkg/berry/tree/master/packages/plugin-compat)
and [HSTS preload list](https://hstspreload.org/). Only the owner of the
package should be able to update them.[1]

Just like TLD Preloading, the entire scope can be preloaded, too. A range
of version may be used.

For example:
```js
export const preloadedDependenciesMeta = [
  [`eslint-scope@*`, {
    built: false,
  }],
  [`@babel/*@^7`, {
    built: false,
  }],
]
```

# How We Teach This

The concept is almost identical to [HSTS preloading](https://hstspreload.org/),
so we can adopt their terminology.

We can start by contacting maintainers of popular packages.

# Drawbacks

1. Yarn may become bloated. This can be mitigated by setting certain
requirements for the package eligible for submission, such as minimum
download count, the number of dependent packages, the number of maintainers, etc.[2]
2. The removal of a package from the list is not retroactive and can take
long time. However, it's unlikely one package would require build scripts
at some point. A warning can be included in the submission form or PR.
3. Some person may squat on a package name, add to the preload list,
and then unpublish the package. This again can be mitigated by setting
certain requirements, possibly stricter than the
[`npm` unpublish policy](https://www.npmjs.com/policies/unpublish).
4. It may give users a false sense of security.

# Alternatives

1. Download the list every time `yarn install` runs, if a flag or config
is set. I think this is a plausible solution, too. This mitigates drawbacks
above but install time may increase.

# Unresolved questions

[1] Who should be considered as an owner? Repo owner, package owner, or
org owner? If there are multiple maintainers, should be the decision
unanimous?

[2] What should be the minumum requirements for the package eligible for submission?
