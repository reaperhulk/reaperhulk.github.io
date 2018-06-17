---
author: admin
comments: true
date: 2009-05-20 03:39:52+00:00
layout: post
slug: basic-webdav-bridge-for-cloud-files
title: Basic WebDAV Bridge For Cloud Files
wordpress\_id: 507
tags:
- cloud files
- cloudfiles
- hack
- php
- webdav
---

Awhile back I signed up for Skitch, a service that lets you create quick screenshots, edit them, and upload them to a variety of services.  Unfortunately, despite the abundance of choices, Cloud Files support is not an option.  This was irritating, and since I've never been one to let phrases like "ridiculous kludge" or "bereft of all sanity" deter me an idea was born. **Why not write a (very) rudimentary WebDAV interface in PHP and use that to bridge to Cloud Files?**
To accomplish this feat it turns out we really only need 3 distinct pieces of functionality.   The actual plumbing that communicates to Cloud Files will be provided by the PHP Cloud Files API.




  1. HTTP Basic Auth to capture the necessary Cloud Files credentials


  2. PUT method for WebDAV


  3. DELETE method for WebDAV


The code below represents the minimum subset of WebDAV functionality required to support Skitch without issues (uploading and deleting).  It is nowhere near a complete WebDAV implementation.  Additionally, some code (mime\_types function and some logging and error handling) have been omitted.  You can [download](/assets/media/2009/05/cf_webdav_bridge_01.zip) the full package for use on your own server.


```php
require('cloudfiles.php');

if (isset($_SERVER['HTTP_AUTHORIZATION'])) {
	list($_SERVER['PHP_AUTH_USER'], $_SERVER['PHP_AUTH_PW']) = explode(':',base64_decode(substr($_SERVER['HTTP_AUTHORIZATION'],6)));
}
if (!isset($_SERVER['PHP_AUTH_USER'])) {
	header('WWW-Authenticate: Basic realm="CloudFiles"');
	header('HTTP/1.1 401 Unauthorized');
	echo 'You must be authorized!';
	exit;
}

$auth = new CF_Authentication($_SERVER['PHP_AUTH_USER'],$_SERVER['PHP_AUTH_PW']);
try {
	$auth->authenticate();
}
catch(AuthenticationException $e) {
	echo 'CF Authentication failure';
	exit;
}
$conn = new CF_Connection($auth);
$skitch_container = $conn->create_container('skitch');
$public_uri = $skitch_container->make_public();

switch ($_SERVER['REQUEST_METHOD']){
	case 'DELETE':
		$file = substr($_SERVER['REQUEST_URI'] , ( strrpos($_SERVER['REQUEST_URI'], '/') + 1) ) ;
		try {
			$skitch_container->delete_object($file);
		}
		catch(NoSuchObjectException $e) {
			echo 'no such object to delete';
			exit;
		}
		catch(InvalidResponseException $e) {
			echo 'invalid response to delete request';
			exit;
		}
	break;

	case 'PUT':
		if($datain = fopen('php://input','r')){
			while(!@feof($datain)){
				$data .= fgets($datain,4096);
			}
			@fclose($datain);
		}
		if(isset($data)) {
			if (isset($_SERVER['PATH_INFO'])) {
				$file_name = substr($_SERVER['PATH_INFO'],1);
			}
			$mime = mime_types($file_name);
			$object = $skitch_container->create_object($file_name);
			$object->content_type = $mime;
			try {
				$object->write($data);
			}
			catch(InvalidResponseException $e) {
				echo 'Invalid response error from CF';
				exit;
			}
			catch(SyntaxException $e) {
				echo 'Syntax exception error from CF';
				exit;
			}
		}
	break;
}

```

To wire this up now, we have just a few steps to perform.




  1. WebDAV can't return the public base URL for the container, so use another program to create a container named "skitch", mark it public, then copy the URL.


  2. Go to the Webpost tab in Skitch and create a new account using WebDAV.


  3. Set server to the server you uploaded the file to, user to your CF username, password to your API key, directory to the name of your script (potentially /path/to/script.php if it isn't at the root of your server's httpdocs), and finally paste the base URL (without a trailing /).  A screenshot of a sample config can be seen below.


[![skitch](/assets/media/2009/04/skitch-300x204.png)](/assets/media/2009/04/skitch.png)

Success!  I've been using this several weeks and haven't had an issue, but this hack does make me wonder how usable a full blown WebDAV translation layer would be...
