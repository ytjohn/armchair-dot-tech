---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags:
 - template
 - default
---
{{<load-photoswipe>}}

Type stuff here

{{< gallery >}}
  {{< figure src="example1.jpg" caption="first example">}}
  {{< figure src="example2.jpg" caption="second example">}}
{{< /gallery >}}

