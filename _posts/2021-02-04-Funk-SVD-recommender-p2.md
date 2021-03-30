---
layout: post
title:  "Simon Funk’s Netflix SVD Method in Tensorflow, Part 2"
date:   2021-02-04 10:00:00 +0000
categories: Tensorflow Python SVD
---

This is part 2 of my implementation of Simon Funk’s SVD method for Netflix challenge. If you want part 1, click [here](https://temugebatpurev.wordpress.com/2021/02/04/simon-funks-netflix-svd-method-in-tensorflow-part-1/). In this post, I apply the method on real data. For the rest of the post, I use The Movies Dateset from Kaggle. More specifically, I use the ratings_small.csv, which can be downloaded here: [link](https://www.kaggle.com/rounakbanik/the-movies-dataset). Please download the data and place it in your working folder.

The ratings_small.csv data set contains 100,000 ratings from 671 users on 9,066 movies. If the matrix was fully filled, then it would contain about 6M ratings. Unfortunately, the matrix is only 1.7% filled. This is quite more sparse than the toy problem in Part 1. The data is stored as user_id, movie_id, rating and timestamp. We don’t use the timestamp variable in this post.


<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/image-14.png?raw=true">
</p>
<p align="center">
<i>ratings_small.csv</i>
</p>
