---
layout: post
title:  "Simon Funk’s Netflix SVD Method in Tensorflow, Part 1"
date:   2021-02-04 10:00:00 +0000
categories: Tensorflow Python SVD
---

Back in 2006, Netflix issued a challenge to predict user scores of movies given the user’s past ratings and ratings of other users. In other words, Netflix provided 100M ratings of 17K movies by 500K users and asked to fill in certain cells, as shown in the image below. The difficulty in their provided data was that only 100M ratings were present out of 8.5B possible entries. Simon Funk used a singular value decomposition (SVD) approach that got him 3rd place in the challenge. In this post, we explore the method and math of his approach and then implement it on a toy problem using Tensorflow. In the next post, I will apply the method to real data. If you’re interested in the original post by Simon Funk, check his blog here: [link](https://sifter.org/~simon/journal/20061211.html).

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/matrix_example.png?raw=true">
</p>
<p align="center">
Ratings are sparsely populated in 1 to 5 range. Question marks need to be filled.
</p>

Filling out this matrix, or certain cells, is useful for recommendation systems. If we can figure out a good method to fill the matrix, then we simply look up a rating for any movie for any new user. I decided to make this post because I found the method quite interesting to study. As we will see soon, the method is based on gradient descent, which means we can use Tensorflow to implement it. If you’re interested in Funk’s method or you’re not familiar with how to write custom training loops in Tensorflow, you might learn something. Or not.

The post will go as follows:

1. Introduction to method and math
2. Writing a basic implementation in Tensorflow
3. Applying the method to a toy problem
4. Applying the method to real data. – This one is cover in the next post.

**Introduction**

The method applies an SVD decomposition to the given giant matrix. The SVD decomposition of any matrix can be written as: A = U S VT, where U and V are ortho-normal matrices, shown below. In fact, if you’re familiar with PCA, the columns of V are the principal axes of PCA. S is a diagonal matrix.


<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/1024px-Singular_value_decomposition_visualisation.svg?raw=true">
</p>
<p align="center">
SVD decomposition. Image from Wikipedia. By Cmglee – Own work, CC BY-SA 4.0, https://commons.wikimedia.org/w/index.php?curid=67853297
</p>
