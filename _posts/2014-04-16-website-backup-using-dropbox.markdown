---
title: Website backup using dropbox
layout: post
---

# Upload server backup directly to dropbox

Hi friends,

Drop-box provides an elegant way to upload, download and view files.  It is a good choice for storing personal files with the advantage of sharing public links. Today, lets take a peep into dropbox PHP api and see how it can be used automate the task of creating online backups in no time.

- To get started, you need to create a Dropbox app by logging into your Dropbox account at https://www.dropbox.com/developers/apps
- Once done with app creation, download the SDK. I am using PHP SDK available at https://www.dropbox.com/developers/core/sdks/php

As mentioned in the above guide, create a .json file containing the app credentials. After first authentication, your app will get access to all the dropbox files. The json file looks something like this:
{% highlight javascript %}
{
  "key": "g89bzk2tlkiefca",
  "secret": "4ggyf1pq0xhelbf"
}
{% endhighlight %}
- To list all files, use the below PHP CLi command:
	[root@ankita examples]# php get-metadata.php auth-file-output /
In this command, first argument is file name of PHP script, second argument is the token obtained by running the ‘authorize.php’ and the third argument is the path of Dropbox . Here it is ‘/’. This will output summary of folders along with list all the files in ‘/’ directory.
- To download a file, use below PHP CLi command:
	[root@ankita examples]# php download-file.php auth-file-output /Ghost.exe Ghost.exe
In this command, first argument is file name of PHP script, second argument is the token obtained by running the ‘authorize.php’, third argument is the file name and fourth argument is the destination file name prefixed by directory name. This will output operation summary.

Now that we are comfortable with uploading/ listing / downloading of files, let us make a small script that will automate the backup operation. Below is the contents of *backup.sh*:

	[root@ankita examples]#tar -czf files.tar.gz files/ && php upload-file.php auth-file-output files.tar.gz /Kayako/files.tar.gz

And below is the output of operation:

	Array
(
    [revision] => 129
    [rev] => 8100ff003d
    [thumb_exists] => 
    [bytes] => 29023959
    [modified] => Sun, 16 Mar 2014 21:51:49 +0000
    [client_mtime] => Sun, 16 Mar 2014 21:51:49 +0000
    [path] => /Kayako/files.tar.gz
    [is_dir] => 
    [icon] => page_white_compressed
    [root] => dropbox
    [mime_type] => application/octet-stream
    [size] => 27.7 MB
)

Automation can be achieved by running this script at regular intervals via cron.
