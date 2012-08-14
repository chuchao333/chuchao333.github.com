---
layout: post
title: "Octopress and Github"
date: 2012-07-25 23:21
comments: true
categories: Octopress
tags: octopress github
---

This post documents how I set up this blog and the problems encounted. Octopress
is quite popular recently and you can just find a huge number of bloggers who
have written about how to set this up online. My main references are
`Setting up a blog on Octopress and github <http://www.gerardcondon.com/blog/2012/03/04/setting-up-octopress-and-github/>`_
and `Blog = Github + Octopress <http://mrzhang.me/blog/blog-equals-github-plus-octopress.html>`_ (In Chinese).

.. more

Ruby setup
----------

Since I am new to ruby, I just follow others' instructions to install ruby by rvm.
My first attempt was:


.. code-block:: console

   $ sudo apt-get install ruby-rvm

This turned out to cause some troubles in the following steps to install ruby 1.9.2.
I don't know the exact reason because I am not familiar with ruby. This `link <http://octopress.org/docs/setup/rvm/>`_
contains detailed information to install rvm. Once it's installed, installing octopress
following `the offical guide <http://octopress.org/docs/setup/>`_ should be straightforward.

Host on Github
--------------

This is very simple in Octopress.
I mainly followed `this <http://code.dblock.org/octopress-setting-up-a-blog-and-contributing-to-an-existing-one>`_, one
thing worth mentioning here is during the step to enter the url of your repository after typing

.. code-block:: console

   $ rake setup_github_pages

Here, the repo must use the username/username.github.com naming scheme,
see `here <https://help.github.com/articles/user-organization-and-project-pages>`_

Writing posts with reStructuredText
-----------------------------------
As you may already know, I am a python fan. So I prefer rst to markdown.
`jekyll-rst <https://github.com/xdissent/jekyll-rst>`_
is the plugin to add rst support to Octopress. After enabling the plugin, I changed
default post ext in _confg.yml to rst. Code block in rst is quite handy to type.

.. code-block:: python

   def test():
       """Testing octopress"""

       print "this is so cool"
