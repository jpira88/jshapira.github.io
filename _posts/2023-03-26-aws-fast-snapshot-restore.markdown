---
layout: post
title:  "Reducing I/O latency on EBS restored from snapshots with AWS FSR" 
date:  2023-03-26 11:31:29 +0200
categories: jekyll update
author: j.shapira
tags: ["DataEngineering", "BE", "Performance", "Cloud", "Architecture"]
---

When creating an EBS snapshot the data is saved to S3.  
Snapshots are incremental, meaning that only the blocks that have been modified later than the most recent snapshot - are saved.  
For obvious reasons this makes the snapshot faster to save and cheaper to store.

When initiating an EBS volume from a snapshot, the EBS is almost immediately ready for use.  
This is a very neat feature which is available due to the asynchronous nature of how storage blocks are loaded from S3 to EBS.  

The data is loaded in 2 ways:
1. Ongoing process in the background
2. In case an access attempt is made to a block which isn't loaded yet, it immediately downloaded from S3.

The caveat in this approach is that if we'll attempt to access a block which is not downloaded yet,
there will be an I/O latency while the data is being downloaded from S3.

To solve this problem, AWS announced a new feature called Fast Snapshot Restore (FSR).  
Enabling FSR for a snapshot increases the amount of resources allocated for the download process.

According to the documentation, enabling FSR on a snapshot can speed up the data loading to ~1TB/Hour,
with current cost of 0.75$ per hour.
So for example, initiating EBS from an FSR enabled snapshot of 10TB will take 10 hours and 7.5$.

Some limitations:  
You can enable up to 5 FSR enabled snapshots per region and only to snapshots sized 16TiB or less.

A full documentation of FSR can be found <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-fast-snapshot-restore.html" target="_blank">here</a>.

