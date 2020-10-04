---
layout: post
title: "Google Drive System"
category: system-design-questions
author: lotus
short-description: Design file storage system
---

-----
# Objectives
Google Drive and Dropbox is file storage system used for storing data. File storage systems become more popular these days because of its usefulness and convenience. People can store and exchange the file through **multi-devices** and **not affected by the location**

# Requirements
- User can upload/download files
- User can offline upload files (auto-synchronize data when the user comes online)
- User can create folders
- Data can be accessed multi-devices anytime
- The uploaded data is never lost
- Unlimited storage

# Assumptions
- We have 500M total users, and 100M daily active users (DAU).
- On average each user connects from three different devices.
- On average if a user has 200 files/photos, we will have 100 billion total files.
- Upload/download ratio is 1:1.
- The average file size is 100KB.
- Data just in type: file and photo.
- 1M active connections per minute.

# Design Proposal

