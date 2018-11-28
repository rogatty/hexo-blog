---
title: Using S3 to store private files
date: 2018-02-11 20:56:20
tags:
---
Planned to use https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html so sharing images is easy but it requires master key to create signed URLs. Not feasible for browser JS apps? I might go back to the topic when working on sharing entries. For now, I used S3 permissions instead.

https://serverless-stack.com/chapters/create-a-cognito-identity-pool.html
https://serverless-stack.com/chapters/upload-a-file-to-s3.html

https://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#RESTAuthenticationQueryStringAuth
https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#getSignedUrl-property

Permissions to DynamoDB:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html
https://github.com/apollographql/GitHunt-API/blob/master/api/schema.js

Data modeling:
http://serverlessarchitecture.com/2016/01/18/dynamodb-performance-and-scaling-with-partitions-indexes-and-readwrite-capacity-units/
https://highlyscalable.wordpress.com/2012/03/01/nosql-data-modeling-techniques/
https://www.slideshare.net/AmazonWebServices/design-patterns-using-amazon-dynamodb
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GuidelinesForLSI.html
https://stackoverflow.com/questions/21381744/difference-between-local-and-global-indexes-in-dynamodb

Global secondary index — an index with a hash and range key that can be different from those on the table. A global secondary index is considered "global" because queries on the index can span all of the data in a table, across all partitions.

Local secondary index — an index that has the same hash key as the table, but a different range key. A local secondary index is "local" in the sense that every partition of a local secondary index is scoped to a table partition that has the same hash key.

user, entity, file
user has many entities
entity has create_date
entity has many files
entity has many tags
get entities with files where user = 'x' order by create_date
get entities with files where user = 'x' and tag = 'y' order by create_date
get tags where user = 'x'

Entry {
	userId H
	createDate R
	content
	hasAttachments
}

Attachment {
	entryId H
	position R
	url
}