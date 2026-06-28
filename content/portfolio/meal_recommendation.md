---
title: "Meal Recommendation System"
date: 2025-11-09
tags: ["Pandas", "sklearn", "Analysis"]
author: "Tarek Mustafa"
draft: false
hidemeta: false
comments: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
  image: "/images/MealRecommendationBanner.webp"
  alt: "Image showing a cartoonish image about a computer recommending a meal"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# How Meal Recommendations Work

Imagine you're running a food marketplace with thousands of users and hundreds of meal options. How do you figure out what someone might want to eat?

Two people who look identical on paper might have completely different taste in food. One person loves spicy Thai dishes, the other can't stand them.

So if demographics aren't the answer, what is?

## Find People Like You

Collaborative filtering works on one simple idea:

> **If you and I order similar things, and I loved something you haven't tried yet, there's a good chance you'll love it too.**

That is it. No guessing based on who you are, just learning from patterns in what people actually order.

It's like real life: if your friend shares your taste in movies, you'll probably trust their movie recommendations. Collaborative filtering does the same thing, but automatically, across thousands of people.

---

## How It Works

### 1. Building the Picture

Every time someone orders and rates a meal, a piece is added to the puzzle. Over time, the system builds a picture of everyone's preferences.

You can think of it like a giant spreadsheet:

![Example of users and meal ratings](/images/MealUsers.webp)

To find meal preferences from this data, it needs to be reshaped into a **user-item interaction matrix**.

- Each row = one user
- Each column = one meal
- Each cell = rating (or 0/blank if not rated)

![Example dataset of users and meal ratings](/images/MealMatrix.webp)

An example using a pandas dataframe.

![Example of pandas dataset](/images/MealPivotDF.webp)

### 2. Finding Similar Tastes

Next, the system compares users to see who has similar tastes. It looks at everyone's ratings and measures how closely their preferences line up.

To do this, it uses a math formula called [**cosine similarity**](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html#sklearn.metrics.pairwise.cosine_similarity), which measures how "aligned" two users' ratings are:

`similarity(u, v) = dot(u, v) / (||u|| * ||v||)`

- If two users rated meals in a similar way, they have high similarity.
- If their tastes are very different, they have low similarity.

The result is a **user-user similarity matrix** that shows who's most alike.

> For example, if User 1 is most similar to User 2 (similarity = 0.82), the system will trust User 2's ratings more when suggesting new meals.

![Example dataset of users and meal ratings](/images/MealScore.webp)

Updated dataframe using the `cosine_similarity` function from [**sklearn**](https://scikit-learn.org/stable/modules/metrics.html).

### 3. Turning Similarity into Recommendations

Once the system knows which users have similar taste, it can recommend meals that similar users liked but the current user has not tried yet.

For example, if User 1 and User 2 are similar, and User 2 gave a high rating to a meal User 1 has never ordered, that meal becomes a good recommendation candidate. In a real application, the system would rank several candidates and show the strongest ones first.

![Example dataset of users and meal ratings](/images/MealSimilarityDF.webp)

---

## Why Businesses Care

This kind of system matters for more than just convenience:

- It increases orders. Showing people food they actually want likely leads to more purchases.
- It helps people discover new things. Many users get stuck ordering the same meals. Good suggestions help them try new meals.
- When the recommendations keep hitting the mark, people trust them and come back more often.
- It improves over time. Every order adds more data, which can make future suggestions more accurate.

---

## The Bottom Line

Collaborative filtering is not new, but it works because it is built on something intuitive: people with similar taste tend to like similar things.

So next time a meal recommendation hits just right, it is not magic. It is math learning from people who eat like you do.
