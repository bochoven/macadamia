---
title: make syntax
layout: single
permalink: /tools/the-luggage/make-syntax/
author_profile: true
---

There is some basic stuff you need to know if you want to extend the provided luggage.makefile. I was surprised by the nice structured syntax and by how well it actually fits what you try to do when buiding a package.

## Rules

Makefiles are governed by rules, basic syntax is:

~~~ make
target: dependency1, dependency2, etc.
	command to create target
	optional other command
~~~

If the target exists, *make* skips the rule. The dependencies are also rules, *make* checks if the dependencies are up-to-date and builds them if necessary. The commands are regular shell commands indented by a tab. *make* echoes every command before it is executed unless it is preceded with a @. Here's an example from *luggage.make*:

~~~ make
payload_d:
	@sudo mkdir -p ${PAYLOAD_D}
~~~

The target is payload_d, it has no other dependencies and it is built using `mkdir -p ${PAYLOAD_D}` (`${PAYLOAD_D}` is a variable declared earlier)

## Extended rule syntax

It took some time to understand this one:

~~~ make
unbz2-applications-%: %.tar.bz2 l_Applications
	sudo ${TAR} xjf $< -C ${WORK_D}/Applications
	@sudo chown -R root:admin ${WORK_D}/Applications/$(shell echo $< | sed s/\.tar\.bz2//g)
~~~

It matches all rules that start with `unbz2-applications-`, using '%' as a wildcard (like * in bash). Then it substitutes the contents of '%' in the first dependency (%.tar.bz2) and then uses the first dependency in the first command using `$<`. 

So if we have a rule that states `unbz2-applications-mail.app`, *make* is going to look for a file or rule that is named `mail.app.tar.bz2`, then it is going to extract that using `${TAR} xjf mail.app.tar.bz2`


## Syntax

`@` prevents the line (not the output) being echoed  
`-` continue after an error occurs  
`%` wildcard, matches any chars  
`$<` matches the first dependency  
