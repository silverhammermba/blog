---
layout: post
title: "A hint of Javascript"
date: 2015-06-02
categories: etc
---

I’ve found this surprisingly useful:

    class String
      def to_proc
        Proc.new { |x| x[self] }
      end
    end

    [{'a' => 1, 'b' => 2}].map(&'a')
    # [1]

It makes Hashes a bit more like Objects.
