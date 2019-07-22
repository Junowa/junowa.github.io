---
layout: post
author: junowa
---

This post gives some directions how to use Makefile for Devops.

## References

https://www.gnu.org/software/make/manual/html_node/index.html#SEC_Contents

https://makefiletutorial.com/

## Pros and Cons for Devops

Pros:
- No dependencies with CI tools
- Modularity
- Built-in features: error handling, builtin function, shell power !
- Simple to use

modularity:
- Recipe is a group of related tasks
- Recipes have dependencies
- One makefile can have several goals (build, install, uninstall...)
- Makefiles can be hierarchical (makefile per layer and parent makefile on top)

Cons:
- Verbosity


## Rules

```
target ...: prerequisites ...
    recipe
    ...
    ...
```
The target is :
- The name of a file
- An action to carry out (.PHONY target)


A prerequisite is a file that is used to as input to create the target.

A recipe is an action or a list of actions.
Don't forget to be a **tab** at the beginning of the line.

## Variables

### References

**$(foo)** is a valid reference to the variable **foo**.

### Flavors

Two flavors:
- Recursively expanded variables
- Simple expanded variables

```
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:;echo $(foo)
````
will echo **Huh ?**

To avoid all problems of recursively expanded variables, there is simple expanded variables.

```
y := foo bar
x := later
````
There is another assignment operator for variables, ‘?=’. This is called a conditional variable assignment operator, because it only has an effect if the variable is not yet defined.

```
FOO ?= bar
```

### Automatic variables

https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html#Automatic-Variables

@?, @%, ...

To be detailed...

### Environment variables

Every environment variable that make sees when it starts up is transformed into a make variable with the same name and value.

You can export globally a variable that will be available in the recipes shells.
```
export variable := value
```

You can also set an env variable inside recipe in a shell.   
The variable is not shared between the different shell.

```
# Update dns based on ingress address
.PHONY: install_add_dns_entry
install_add_dns_entry:
	dns_rg=$$(cat $(ENV).yaml | shyaml get-value dns_rg); \
	dns_domain=$$(cat $(ENV).yaml | shyaml get-value dns_domain); \
	dns_name=$$(cat $(ENV).yaml | shyaml get-value dns_name); \
	-az network dns record-set a add-record -g $(dns_rg) -n $(dns_name) \
		-a $(shell kubectl -n ingress-controller get svc ingress-controller-nginx-ingress-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}') \
		-z $(dns_domain)
```

Double the $$ or make will try to substitute the variable.




## Recipe execution

When it is time to execute recipes to update a target, they are executed by invoking a new sub-shell for each line of the recipe, unless the .ONESHELL special target is in effect.



### Recipe echoing

Normally make prints each line of the recipe before it is executed. We call this echoing because it gives the appearance that you are typing the lines yourself.

When a line starts with ‘@’, the echoing of that line is suppressed.

```
@echo About to make distribution files
```

### Errors in recipe

To ignore errors in a recipe line, write a ‘-’ at the beginning of the line’s text (after the initial tab). The ‘-’ is discarded before the line is passed to the shell for execution.

For example,
```
clean:
        -rm -f *.o
```
## Run Make

### Overriding variables

An argument that contains ‘=’ specifies the value of a variable: ‘v=x’ sets the value of the variable v to x

### Arguments to specify goals

The goals are the targets that make should strive ultimately to update. 

By default, the goal is the first target in the makefile. 

