---
layout: post
title:  "Simon Funk’s Netflix SVD Method in Tensorflow, Part 1"
date:   2021-02-04 10:00:00 +0000
categories: Tensorflow Python SVD
---

Back in 2006, Netflix issued a challenge to predict user scores of movies given the user’s past ratings and ratings of other users. In other words, Netflix provided 100M ratings of 17K movies by 500K users and asked to fill in certain cells, as shown in the image below. The difficulty in their provided data was that only 100M ratings were present out of 8.5B possible entries. Simon Funk used a singular value decomposition (SVD) approach that got him 3rd place in the challenge. In this post, we explore the method and math of his approach and then implement it on a toy problem using Tensorflow. In the next post, I will apply the method to real data. If you’re interested in the original post by Simon Funk, check his blog here: [link](https://sifter.org/~simon/journal/20061211.html).

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/matrix_example?raw=true">
</p>
<p align="center">
Ratings are sparsely populated in 1 to 5 range. Question marks need to be filled.
</p>
