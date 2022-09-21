---
title: Media player qui ne fonctionne pas dans ownCloud 5
author: VaLouille
type: post
date: 2013-03-15T15:32:50+00:00
url: /2013/03/media-player-qui-ne-fonctionne-pas-dans-owncloud-5/
categories:
  - linux

---
Sur la toute nouvelle version d&rsquo;ownCloud (5.0.0) le &lsquo;Dropbox like libre&rsquo;, le media player ne fonctionne pas. Il est impossible de lire de la musique. Dans firebug, j&rsquo;ai trouvé l&rsquo;erreur suivante :

```
{"data":{"message":"Token expired. Please reload page."},"status":"error"}
```

En fait, il faut modifier le fichier apps/media/ajax/api.php tel que ci-dessous :

```
 \OCP\JSON::checkAppEnabled('media');
 \OCP\JSON::checkLoggedIn();
-\OCP\JSON::callCheck();
 
 error_reporting(E_ALL); //no script error reporting because of getID3
 
@@ -43,6 +42,7 @@
 if ($arguments['action']) {
   switch ($arguments['action']) {
     case 'delete':
+      \OCP\JSON::callCheck();
       $path = $arguments['path'];
       $collection->deleteSongByPath($path);
       $paths = explode(PATH_SEPARATOR, \OCP\Config::getUserValue(\OCP\USER::getUser(), 'media', 'paths', ''));
```
