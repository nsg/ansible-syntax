# Ansible Syntax

If you are the lone coder of a project, coding style do not matter as long you are consistent. Try not to mix coding styles because it makes the code harder to read and understand.

Things will be more complex when there are more than one person working on a project and I think it's important to decide of a coding style. It's possible that you both have strong opinions and if that's the case try to compromise, or even better try to pick the most common standard for the language or software you are working on, it will make life easier when you add people in the future.

If possible I recommend a formatter or linter that will check/enforce the coding style. This will make your life a lot easier to keep and enforce "the rules".

## Scope of this project

I have written Ansible code for a few years and there are an informal default out there with a few variations I will walk through them and try to make a educated decision, and a recommendation.

## Index

* [Indentation](#indentation)
* [A play](#a-play)

## Indentation

This is a classic discussion when it comes to software projects, there are plenty of strong options like tabs, spaces or even a mix of both. How wide is a tab, 8 spaces, 4? How many spaces to use? 2, 8, 4 or even 3? There are plenty of options out there.

### YAML spec

From the YAML spec [4.2.2. Indentation Spaces](http://yaml.org/spec/current.html#id2519916)

> /../ where indentation is defined as a line break character (or the start of the stream) followed by zero or more space characters. Note that indentation must not contain any tab characters. The amount of indentation is a presentation detail used exclusively to delineate structure and is otherwise ignored. /../

For *me* that translates as:

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

*Personally I prefer the 2 step indentation because I think it's more clear but I have people that have told me that they like the zero indentation because it's more compact.*

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

- name: My playbook
  hosts:
    - bar
    - baz

  tasks:
    - name: Install nginx
      apt:
        name: nginx

    - name: Start nginx
      service:
        name: nginx
        state: started
```

*Note: The extra space is recommended for clarity to separate different units of logic.*

## A play

An Ansible run is a sequence of one or more playbooks so they are one of the most basic concepts of Ansible. So let's discuss some formatting of a playbook.

### Basic

The most simple playbook I can think about, that actually do something, is this:

```yaml
- hosts: all
  tasks:
    - debug:
        msg: "{% raw %}{{ inventory_hostname }}{% endraw %}"
```

This is fine playbook, valid YAML and Jinja2. In the old days of Ansible we could write things like below, and it was even promoted in the documentation (and to some extent there still is a lot of this old code out there):

```yaml
- hosts: all
  tasks:
    - debug: msg=inventory_hostname
```

It's nice and compact but it need a lot of parsing done by Ansible outside of the YAML and Jinja2 parsers so this format is deprecated. The key=value syntax still works but bare variables will complain in modern Ansible.

### Parsing

I have not read any formal specification or the code but from observation the file is parsed like this:

1. The playbook is parsed with the YAML parser
2. The Jinja2 parser is executed on all supported strings in the parsed data structure.
3. Possible extra things like legacy key=value parsing and so on...

This means that if you inject some Jinja logic in to the playbook it still needs to be valid YAML.

**Valid**

```yaml
- hosts: all
  tasks:
    - debug:
        msg: "{% raw %}{% if a == 2 %}foo{% else %}bar{% endif %}{% endraw %}"
```

**Invalid**

```yaml
- hosts: all
  tasks:
    - debug:
{% raw %}{% if a == 2 %}
        msg: "foo"
{% else %}
        msg: "bar"
{% endif %}{% endraw %}
```

### Readable

Ansibles YAML is a simple language that sometimes feels limiting but please do **not** try to write complex inline programs with Jinja, just because you can do that do not mean that it was the right thing to do. In many case you can solve it in a more clever "Ansible way" anyway.

From the example above, do something like this:

```yaml
- hosts: all
  tasks:
    - debug:
        msg: "foo"
      when: a == 2

    - debug:
        msg: "bar"
      when: a != 2
```

... or even better, place a variable in host_vars, group_vars, inventory or wherever and just do this:

```yaml
- hosts: all
  tasks:
    - debug:
        msg: "{% raw %}{{ my_var }}{% endraw %}"
```

Give it some time to structure your playbook in a readable way. If it's large use include to split it in to separate files. Comments are a nice thing and always use name.

```yaml
- name: "A example that outputs a debug message"
  hosts: all
  tasks:
    - name: "Print the debug message"
      debug:
        msg: "{% raw %}{{ inventory_hostname }}{% endraw %}"
```

Notice that I have defined a name for the entire play, and the task. This makes it easier to understand the playbook without the need for a comment. The output from the plays execution is of course also easier to understand.

If you know a little about YAML you know that `foo: bar` will be parsed as `str(bar)` while `foo: 1.2` will result in a `float(1.2)`. If you really like 1.2 to be a string the right thing to do is to use quotes. Y, N, No, False, false and so on will be parsed as boolean value.

People like to omit the quotes mostly to write nice code like this:

```yaml
- name: Do stuff
  template:
    src: "{% raw %}{{ item }}{% endraw %}.j2"
    dest: "/path/{% raw %}{{ item }}{% endraw %}"
  with_items:
    - foo
    - bar
```

Personally I have nothing against keeping foo and bar without quotes. It's a style that I personally like and it's common in it's usage. But please try to be consistent, do not mix and match it like this:

```yaml
- name: Do stuff
  template:
    src: "{% raw %}{{ item }}{% endraw %}.j2"
    dest: "/path/{% raw %}{{ item }}{% endraw %}"
  with_items:
    - foo
    - "bar"
```

The disadvantage to ignore the quotes are that if you start out with a simple list with no quotes and you later on decides to use variables you need to add quotes to every element to stay consistent. That creates a larger diff in your code commit and that is never a good thing. Combine that with the fact that Jinja convertes `yes` to bool(True) and that can cause bugs if you expected a literal yes.

So I will end I recommend to use quotes most of the time, and only omit them if you like to use the Jinja conversions, or possible for the aesthetics.

### Recommendation

```yaml
- name: "This do some more things"
  hosts: "web:db"
  roles:
    - role: foo
      tags:
        - foo

    - role: bar

  tasks:
    - name: "Task: 1"
      debug:
        msg: "foo"

    - name: "Task: 2"
      debug:
        msg: "No {% raw %}{{ item }}{% endraw %}"
      with_items:
        - "one"
        - "{% raw %}{{ number }}{% endraw %}"
        - yes
        - 1.36768

```

In the end, find a style that works for you and who you work with. If you joins a existing project look around in the files and get a feel about the style in use and try to mimic it.
