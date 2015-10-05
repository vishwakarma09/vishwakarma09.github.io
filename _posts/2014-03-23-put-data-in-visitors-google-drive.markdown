---
title: Put data in visitor's google drive
layout: post
---

# Put data in visitor's google drive

Hi friends,

In continuation of working with cloud storage, let us take a peep into the Google drive system. Google has provided with extensive API support to all its products including drive. I am using PHP sdk at

	https://github.com/google/google-api-php-client

Earlier the code was on code.google.com, but due to recent decisions (in which download section was omitted due to abusive use), all open source projects are turning to Github.

To get started, extract the sdk in your document root. Next thing is to get the API information. For this web project you need to first create a project at :

	https://console.developers.google.com/project

After creating project:

- Go to APIs and Auth > Turn on Drive API and Drive SDK
- Go to credentials > create new client ID > client ID for web application. Here enter details viz:
- Authorized JavaScript origins : eg. http://candyartist.tk
- Authorized redirect URI eg. http://candyartist.tk/
	NOTE: Note the trailing '/' is important.

After this you can create a 'HTML' document in document root and populate it with your API credentials. The complete file is at:
  
When this file will be accessed by clients, they will first need to 'authenticate your app'. After authentication, this interface can be used by them to upload files to their google drive. A working sample available at:

	http://candyartist.tk/index.html

Your comments and suggestions are always welcome.