---
title: "Shopware 6: Images and their locations"
description: "How are the images stored and how is the path for the images generated when you upload images in Shopware 6 ? Lets try to find out."
date: 2023-01-01T16:59:58+01:00
draft: false
toc: false
images:
tags:
  - shopware6
---
First we need to be aware that there are different ways to store images in shopware. You can store them on your local file system, or you can store them in Amazon S3 or Google Cloud Platform buckets.

For this article we'll look into how the files are stored when they are save on the local filesystem.

## Path strategies
In Shopware6 there are 4 different path strategies that can be chosen in the core.

The path strategy needs to be chosen when you install the store, and it is rather hard to change the strategy on an existing installation. If you change the strategy you will need to rename all the files on the disk to new paths.

The default in Shopware6 is the `physical_filename` strategy.
This can be changed by setting the env variable `SHOPWARE_CDN_STRATEGY_DEFAULT`.

### The strategies

So what are the different strategies that can be used, and how do they work ?

Common for all strategies is that the physical path for the media enteties are not stored in the database. The path are calculated when you upload a media entity and then the path is re-calculated each time you need to get the URL path for a media entity.

The urls for media is calculated like this:
```php
        return $this->toPathString([
            'media',
            $this->pathnameStrategy->generatePathHash($media),
            $this->pathnameStrategy->generatePathCacheBuster($media),
            $this->pathnameStrategy->generatePhysicalFilename($media),
        ]);
```

`generatePathCashBuster` - This part is the upload time in unix timestamp. For a file that was uploaded on 4th of May 2022 at 13:37:00 the timestamp would be `1651664220`

`generatePhysicalFilename` -  The media entity has the filename and the extension and this function returns the full name of the file.

The differences between the built-in path-strategies are the `generatePathHash` function.


#### physical_filename
Generate a md5-hash based on the uploaded timestamp and the name of the file. It then uses the 6 first characters and add a slash for every two character. 

Example: You have a media where the timestamp is `1651664220` and the filename is `superproduct`. This strategy will do `md5('1651664220/superproduct')` and get the result `1af5f762a63c7f48ced23376a3dc5b8b` where it will use `1af5f7` and the result for the `generatePathHash` function will be `1a/f5/f7`
You will then end up with the full path `media/1a/f5/f7/1651664220/superproduct.jpg`


#### filename
Generate a hash based only on the filename

Example: You have a media where the filename is `superproduct`. This strategy will do `md5('superproduct')` and will get the result `3fe87fc4ca7c159664484ee79631ec8e` where it will use `3fe87f` and the result for the `generatePathHash` function will be `3f/e8/7f`
You will then end up with the full path `media/3f/e8/7f/1651664220/superproduct.jpg`

#### plain
Generate no hash

Example: You have a media where the filename is `superproduct`. 
You will then end up with the full path `media/1651664220/superproduct.jpg`

#### id
Generate a hash based on the ID of the media entity

Example: You have a media where the filename is `superproduct` and the id of the media entity is `897cd16e55b7450787192d100cc8320d`. This strategy will do `md5('897cd16e55b7450787192d100cc8320d')` and will get the result `400daaacfe5308e03f3dea7e4816e29e` where it will use `400daa` and the result for the `generatePathHash` function will be `40/0d/aa`
You will then end up with the full path `media/40/0d/aa/1651664220/superproduct.jpg`
