---
title: "{{ replace (replace .Name "-" " ") "_" " " | title }}"
date: {{ .Date }}
tags:
  - tag1
  - tag2
  - tag3
draft: true
author: "Colmaris"
description: "Jour 006/100 du défi 100DaysToOffLoad."
categories:
  - cat1
toc: true
---