---
layout: post
title:  "Add openwrt package"
author: Willen
categories: [ openwrt ]
tags: [ openwrt]
comments: false
rating: false
---

# Add openwrt package

以sphinxbase-5prealpha.tar.gz为例。

1. 先把openwrt的rule.mk包进来：

   include $(TOPDIR)/rules.mk

2. 定义package的信息：

   1. PKG_NAME:=sphinxbase
      PKG_VERSION:=5prealpha
      PKG_RELEASE:=1

      PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
      PKG_SOURCE_URL:=https://github.com/willendzw/cmusphinx-depot/blob/master/cmusphinx-5prealpha/sphinxbase-5prealpha/
      PKG_HASH:=f72bdb59e50b558bed47cc2105777200d2b246a0f328e913de16a9b22f9a246f

      PKG_LICENSE:=BSD
      PKG_LICENSE_FILES:=

      PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

      PKG_BUILD_PARALLEL:=1

      PKG_INSTALL:=1

   以上package的变量含义，可参考https://openwrt.org/docs/guide-developer/packages。PKG_HASH没有的话，可以手动计算。如下：

   sha256sum sphinxbase-5prealpha.tar.gz

   将生成的哈希值填充到Makefile的PKG_MIRROR_HASH即可。

3. 编译目标

   
   
4. 

   






