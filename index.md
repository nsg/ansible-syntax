# Ansible Syntax

If you are the lone coder of a project, coding style do not matter as long you are consistent. Try not to mix coding styles because it makes the code harder to read and understand.

Things will be more complex when there are more than one person working on a project and I think it's important to decide of a coding style. It's possible that you both have strong opinions and if that's the case try to compromise, or even better try to pick the most common standard for the language or software you are working on, it will make life easier when you add people in the future.

If possible I recommend a formatter or linter that will check/enforce the coding style. This will make your life a lot easier to keep and enforce "the rules".

## Scope of this project

I have written Ansible code for a few years and there are an informal default out there with a few variations I will walk through them and try to make a educated decision, and a recommendation.

## Index

* [Indentation](#Indentation)

## Indentation

This is a classic discussion when it comes to software projects, there are plenty of strong options like tabs, spaces or even a mix of both. How wide is a tab, 8 spaces, 4? How many spaces to use? 2, 8, 4 or even 3? There are plenty of options out there.

### YAML spec

From the YAML spec [4.2.2. Indentation Spaces](http://yaml.org/spec/current.html#id2519916)

> /../ where indentation is defined as a line break character (or the start of the stream) followed by zero or more space characters. Note that indentation must not contain any tab characters. The amount of indentation is a presentation detail used exclusively to delineate structure and is otherwise ignored. /../

With for *me* translates as:

> You are free to choose any amount of indentation, they must be spaces and the purpose to keep it flexible is for readability.

So YAML do not enforce anything really as long at the block is indented with one or more space (0 or more for lists). But readability is a thing so keep that in mind.

[yaml.org](http://yaml.org/) is written in YAML to showcase how readable it is and they have used a two space indentation:

```yaml
What It Is: YAML is a human friendly data serialization
  standard for all programming languages.
```

Lists are first not indented, and later on they are:

```yaml
Projects:
  C/C++ Libraries:
  - libyaml       # "C" Fast YAML 1.1
  - Syck          # (dated) "C" YAML 1.0
  - yaml-cpp      # C++ YAML 1.2 implementation
```

```yaml
Related Projects:
  - Rx            # Multi-Language Schemata Tool for JSON/YAML
  - Kwalify       # Ruby Schemata Tool for JSON/YAML
```

Maybe that's to keep it looking nice and aligned with the other list? At least it demonstrates two common ways to write lists.

### Ansible

Ansible do not recommend any specific indentation but by looking at examples at the [Best Practices](http://docs.ansible.com/ansible/playbooks_best_practices.html) page we see this:

```yaml
---
# file: webservers.yml
- hosts: webservers
  roles:
    - common
    - webtier
```

This matches the YAML syntax discussed above. They also provides a [examples repository](https://github.com/ansible/ansible-examples), let's take a peak.

By the look of it *most* code is indented 2 steps, lists are either indented 0 or 2 steps just like on the YAML homepage. The two step indentation seems to be a little more popular.

*Personally I prefer the 2 step indentation because I think it's more clear but I have people that have told me that they like the zero indentation because it's more compact and I do understand that.*

Of course there are also a few odd ones, like a list with a one step indentation but I feels like it's more likely to be a typo than a style choice.

### The community

I took some time looking trough [Ansible Galaxy](https://galaxy.ansible.com) and [GitHub](https://github.com) to get a feel about coding style and this is what I found:

People are not consistent even within their own files but two spaces are really common for most indentations, except lists where the indentation lever varies a lot, but 0 or 2 are most common.

### yamllint

> [yamllint](https://github.com/adrienverge/yamllint) does not only check for syntax validity, but for weirdnesses like key repetition and cosmetic problems such as lines length, trailing spaces, indentation, etc.

I do of course understand that this is just one tool, made by a third part but it is still a interesting metric, let's see how it behaves by default on a zero indexed list:

```yaml
---
hello:
- 1
- 2
```

```yaml
foo.yml
  4:1       error    wrong indentation: expected 2 but found 0  (indentation)
```

It did not like that, at all. But it likes 1, 2, 4, 6, 8 ... spaces so it's somewhat flexible.

### Recommendation

The most common indentation I have found is a two white space indentation. There where a higher variance with lists but with a little weight from yamllint and to keep things the same I will recommend a two step intend *everywhere*.

```yaml

foo:
  - bar
  - baz

foo:
  bar:
    - baz
  baz:
    - qux
    - quux

```