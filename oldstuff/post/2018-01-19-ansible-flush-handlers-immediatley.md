---

title: Ansible Flush Handlers Immediatley
author: ytjohn
date: 2018-01-19 15:30:43
tags: [yourtech-dailies]
layout: post

slug: ansible-flush-handlers-immediatley

---

## ansible flush handlers

In ansible playbooks, handlers (such as to restart a service) normally happen at the end of a run. 
If you need ansible to run a handler between two tasks, there is "flush_handlers".

```
  - name: flush handlers
    meta: flush_handlers
```
