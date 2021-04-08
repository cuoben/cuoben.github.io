---
layout: post
title: "关于表引擎VersionedCollapsingMergeTree和CollapsingMergeTree的差异"
description: "关于表引擎VersionedCollapsingMergeTree和CollapsingMergeTree的差异"
categories: [clickhouse]
tags: [clickhouse,VersionedCollapsingMergeTree,CollapsingMergeTree]
redirect_from:
  - /2021/02/22/
---



# 关于表引擎VersionedCollapsingMergeTree和CollapsingMergeTree的差异

1. VersionedCollapsingMergeTree在折叠数据时会优先折叠主键排序的首位，而CollapsingMergeTree与之相反，会折叠主键排序的末位。
2. 在插入主键重复数据时CollapsingMergeTree会和ReplacingMergeTree一样会替换旧数据，而VersionedCollapsingMergeTree不会去重，数据会追加。