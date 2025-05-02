---
tags:
  - yaml
  - multi-lines
  - tips
---

====== Comment gérer les commandes multi lines ======

Ansible uses YAML syntax in its playbooks. YAML has a number of block operators:

The > is a folding block operator. That is, it joins multiple lines together by spaces. The following syntax:

  key: >
    This text
    has multiple
    lines
    
Would assign the value This text has multiple lines\n to key.

The | character is a literal block operator. This is probably what you want for multi-line shell scripts. The following syntax:

  key: |
    This text
    has multiple
    lines

Would assign the value This text\nhas multiple\nlines\n to key.

You can use this for multiline shell scripts like this:

  - name: iterate user groups
    shell: |
      groupmod -o -g {{ item['guid'] }} {{ item['username'] }} 
      do_some_stuff_here
      and_some_other_stuff
    with_items: "{{ users }}"

There is one caveat: Ansible does some janky manipulation of arguments to the shell command, so while the above will generally work as expected, the following won't:

  - shell: |
      cat <<EOF
      This is a test.
      EOF

Ansible will actually render that text with leading spaces, which means the shell will never find the string EOF at the beginning of a line. You can avoid Ansible's unhelpful heuristics by using the cmd parameter like this:

  - shell:
    cmd: |
      cat <<EOF
      This is a test.
      EOF