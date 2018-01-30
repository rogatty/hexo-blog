---
title: Journal - a place to keep my memories
description: I want to have a place to keep my memories. It includes notes, photos, videos and possibly other types of content. It should be available at all times, wherever I go, which means both desktop and mobile support.
date: 2018-01-29 09:06:56
---
I want to have a place to keep my memories. It includes notes, photos, videos and possibly other types of content. It should be available at all times, wherever I go, which means both desktop and mobile support. I don't need full access when I'm offline but it should at least allow me to add new content and synchronize when there is a connection. I want to be able to tag the content and search it easily. It should be secure but I also want to share some of the content with family or friends. I want to allow them to upload images in photo albums I invited them to. Would be nice if the content could be commented.
<!-- more --> 

## Existing solutions
I checked multiple free apps and played with a few trial versions. Some were close but none of them met all my requirements. Google Photos has almost everything I need but unfortunately, it doesn't allow text content. Also, I'm wary of giving Google *even more* information.

## Something of my own
I decided to build my own application that will do all the things I described above. I like being able to implement any feature I may need in the future instead of migrating between applications or sending feature requests which may never be satisfied. It will probably cost me a lot of time but in the process, I'm going to learn new concepts and technologies. If I succeed it will be something to be proud of and might serve me for years to come. If it ends up really good I might even find a way to monetize it.

## Backend
My plan is to use AWS services as the backend. I have no experience with AWS but from my research, it seems that:
1. It's popular and there is a lot of learning resources available.
1. Storage is cheaper than other providers I checked.
1. I feel confident in letting them store confident data, I believe they're as stable as Google.
1. They offer the whole package. I'm going to use:
    - S3 for file storage
    - DynamoDB for metadata and text storage
    - Cognito for authentication
    - IAM for authorization
    - Lambda for thumbnail processing and GraphQL API (probably more as I make progress)
    - CloudFront
        - As a cache, to keep S3 costs lower
        - As a security layer, to prevent accessing S3 directly
    - And more when I get to things like monitoring, logging, etc.

I thought briefly about keeping the data in local storage but quickly changed my mind after checking HDD prices and estimating electric bills. It doesn't make sense, nowadays cloud solutions are much cheaper and more reliable.

## Frontend
I picked React as the frontend framework because I wanted to learn it. Before, I had only a brief contact with it, working on [a fork of Ooyala HTML5 skin](https://github.com/Wikia/html5-skin/commits?author=rogatty). It's the most popular JS framework and I'd like to know what's all the fuss about. I'm going to keep an eye on keeping the code as universal as I can to not be tied to React too much. If I get stuck or decide I don't want to learn it anymore, I'll rewrite the frontend to Ember which I work with for over 3 years.

## API
As a bridge between AWS and React, I picked GraphQL. It's another trendy technology that I don't know and I like what it brings to the table. Schemas should prevent me from making stupid typo mistakes and allow easier refactoring. Using single API request to get all the needed data should speed up the application, especially on 3G connection. I don't see the need for the REST's caching as I'm going to store as much as possible in the app for offline support anyway. Also, there won't be many users sending requests. Even if I scale at some point, the responses won't be shared as it's all private data so it couldn't be cached well. Versioning or information hiding that REST provides doesn't seem necessary as I'll be the only one working on the app.

## Offline support
I spent a couple of days researching options for managing data store in React. I got hyped after reading [Introducing Redux Offline: Offline-First Architecture for Progressive Web Applications and React Native](https://hackernoon.com/introducing-redux-offline-offline-first-architecture-for-progressive-web-applications-and-react-68c5167ecfe0), then found [apollo-offline](https://github.com/Malpaux/apollo-offline) which is built on top of redux-offline and I thought that it'll be a smooth ride. Unfortunately, [Apollo moved away from Redux in v2.0](https://www.apollographql.com/docs/react/2.0-migration.html#redux) and replaced it with a custom store. Theoretically, I could use the old version but it's not supported anymore and I don't think it's a good idea to do that in a new project. There is [some movement](https://dev-blog.apollodata.com/announcing-apollo-cache-persist-cb05aec16325) in Apollo project but it's still in progress. That's why I decided to keep the offline support out of the picture for now and get back to it when I figure out the more important issues. I take into consideration the possibility to give up GraphQL and use redux-offline + REST instead.

## Current state
I started work in November and got some elements done already. [Serverless Stack](https://serverless-stack.com/) is a great resource that helped me a lot. I used some of the code there but also mixed in elements from [Full-stack React + GraphQL Tutorial](https://dev-blog.apollodata.com/full-stack-react-graphql-tutorial-582ac8d24e3b) and some other tutorial I forgot. To make it less ugly I used [React wrapper for Material Design (Web) Components](https://github.com/jamesmfriedman/rmwc) but I don't have a particular plan for the visuals yet. When it works as expected, then will be the time to make it pretty.

What works:
- Adding, editing and removing entries
- Entries are synchronized with DB
- I have a dev environment without need to call AWS
- Login page is in progress

{% asset_img "screenshot.png" "Journal as of 29.01.2018" %}

Repository link: [https://github.com/rogatty/journal](https://github.com/rogatty/journal)

## Timeline
So far, the progress was slow but I want to change it. On 22.01.2018 I started my first [12 Week Year](https://www.goodreads.com/book/show/10009377-the-12-week-year) and one of three goals is to create a Journal MVP and write about it on this blog. I track the progress in a 12WY spreadsheet and [in a Github project](https://github.com/rogatty/journal/projects/1). If everything goes as planned, I should have the MVP ready before 16.04.2018.