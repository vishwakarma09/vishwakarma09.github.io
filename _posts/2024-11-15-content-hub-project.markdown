---
title: ContentHub filesharing
layout: post
---

To make file sharing easy between group of people. A user can upload files in his own ContentHub, and then the file/folder can be shared with specific people. Sharing with multiple people is very easy. Generate public links for files. Complete with registration, login, forget-password, reset password, logout, etc,. features.

Built on Laravel 11.x, Vue3, MySQL

### GitHub links:

[API](https://github.com/vishwakarma09/content_hub_api) written in Laravel 11.x, using laravel/sanctum, kalnoy/nestedset
[Infra](https://github.com/vishwakarma09/content_hub_infra) Docker compose
[FrontEnd](https://github.com/vishwakarma09/content_hub_frontend) Written in Vue 3 using Tailwind, DaiseyUI, PrimeVue and pinia for state management

### Demo:
[Video Link](https://drive.google.com/file/d/1WW8f302xrRK1iFyM8DrAcW6VWQ5IMyc2/view)

### Key Features
- Files/folders represented as nodes. For nodes which are of type file, there is extra metadata stored for uploaded file.
- If node type is folder, it can be browsed like a directory
- Ability to create public shareable link
- Ability to share a folder to multiple people
- Full access to shared folder; can add, delete files from shared folder

### Todo
- Ability to add tags
- Preview common fileTypes e.g. PDF, images, etc,.
- Social login
