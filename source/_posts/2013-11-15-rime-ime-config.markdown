---
layout: post
title: "鼠须管输入法的几项配置"
date: 2013-11-15 21:42
comments: true
categories: Mac IME
---

<!--more-->

### 默认输入英文

如果输入法默认处于英文状态，就可以删除英文输入法，直接使用Shift切换中英文输入。

配置文件：`luna_pinyin_simp.schema.yaml`

配置项：`switches` -> `ascii mode` -> `reset`

值：`1`

{% codeblock %}
switches:
  - name: ascii_mode
    reset: 1
    states: [ 中文, 西文 ]
{% endcodeblock %}

### Shift直接输入英文

默认情况下，在输入中文时按下Shift键会临时切换到英文状态，回车输入英文后恢复中文输入状态，按如下方法修改为直接输入英文并切换到英文输入状态。

配置文件：`default.yaml`

配置项：`ascii_composer` -> `switch_key` -> `Shift_L` / `Shift_R`

值：`commit_code`

{% codeblock %}
ascii_composer:
  good_old_caps_lock: true
  switch_key:
    Caps_Lock: commit_code
    Control_L: noop
    Control_R: noop
    Shift_L: commit_code
    Shift_R: commit_code
{% endcodeblock %}
