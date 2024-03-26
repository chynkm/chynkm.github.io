---
author: "Karthik M"
title: "Cache busting using PHP asset fingerprinting"
date: "2019-07-01"
canonicalUrl: https://litebreeze.com/software-development/php-asset-fingerprinting
tags:
- asset fingerprinting
- cache busting
- php
- cdn
---
In this fast moving technological age, users expect swift information at their fingertips. Most users have a low
tolerance for slow loading websites. How can we increase user retainment and ensure better customer satisfaction?

The below article focuses on one aspect of improving web application and it’s development.

## What are Assets?
Assets are files like CSS, Image, JavaScript etc. that forms the frontend of your website.

## How to speed up Asset delivery?
There are many ways to make the website load faster to its users. Below are 4 easy to use methods which have been tried
and tested by myself:

(1) Minify assets – Removing redundant and unwanted code without affecting the resource is called minification. You can
find more information [here](https://developers.google.com/speed/docs/insights/MinifyResources).

(2) Cache assets – (Asset caching can be implemented at CDN, ISP, networking equipment, server or browsers. In this post,
we will focus on browser caching). According to wikipedia, cache is a hardware or software component that stores data so
future requests for that data can be served faster. When cache is applied to your browser, the browser "remembers" the
assets which were loaded.

When you request a particular web page, the browser runs multiple requests to the server to display the page and load
its assets. If asset caching is enabled on the website, the assets get cached on the first request. They will not be
re-fetched on subsequent requests. This is the reason that the initial page load time is greater than the successive
page loads.

By utilising browser caching, asset files get stored in the browser cache. This in turn decreases the page load time for
all pages which share the assets.

(3) GZIP compression – Say a CSS file has a size of 200 KB. When you [GZIP](https://en.wikipedia.org/wiki/Gzip) the
file, the size becomes 40 KB. The time of delivery from the server to the user’s browser will be less for a 40 KB file
than a 200 KB file. All modern browsers handle GZIP compression. Compressing your files will decrease the data transit
time, in turn decreasing the page load time.

Please check these links for configuration info about GZIP compression depending on your webserver
[Apache GZIP](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html),
[Apache DEFLATE](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html),
[NGINX GZIP](https://nginx.org/en/docs/http/ngx_http_gzip_module.html).

(4) CDN – A Content Delivery Network (CDN) refers to a geographically distributed group of servers which work together
to provide fast delivery of Internet content. Simply put – it is a bunch of servers all around the world that host your
files. As the browser requests your web page, it downloads the (asset) files from the nearest server (thus decreasing
the page load time). eg: [CloudFront](https://aws.amazon.com/cloudfront/)

However, caching assets on your browser creates another problem. For example, let’s suppose your stylesheet is called
`style.css` and a browser access your web page. Due to the asset cache functionality, it loads the file `style.css` from
the browser cache.

If the file content is later modified on the server, the browser will not know about it. All subsequent requests to the
webpage will load the cached file, as it is cached in the browser. Until the cache is cleared or the cache expires,
the file from the server will not be loaded in the browser.

In order to mitigate this problem, we would need to force the browser to request for the modified file. The solution to
this problem is the topic of our post.

## What is asset fingerprinting?
Asset fingerprinting is a technique that forces a relation between the filename and its content. Simply said: the name of a file depends on its content. When the content of a file is altered, asset fingerprinting modifies the corresponding filename. This ensures that the remote clients request a fresh copy of the file.

The act of forcing your browser (CDN, ISP, networking equipment, server) to load the modified file is accomplished by asset fingerprinting. This is generally known as cache busting.

Below you can find the extract of a PHP code used to achieve the same in CodeIgniter framework. The logic can be used across all frameworks/languages.

`common_helper.php`
```
function asset_url($filename = '')
{
    if(ENVIRONMENT == 'development') {
        return site_url('assets/'.$filename).'?ver='.md5_file('assets/'.$filename);
    }

    $explodedFileName = explode('.', $filename);
    $fileExtension = array_pop($explodedFileName);
    $filename = implode('.', $explodedFileName).'-'.md5_file('assets/'.$filename).'.'.$fileExtension;
    return site_url('prod/'.$filename);
}
```

`Asset.php`
```
<?php
class Asset {
    public static function fingerprint_assets() {
        $source_directory = 'assets';
        $destination_directory = 'prod';
        exec('rm -rf '.$destination_directory);
        self::recursive_copy($source_directory, $destination_directory);
    }
    private static function recursive_copy($source_directory, $destination_directory) {
        $dir = opendir($source_directory);
        mkdir($destination_directory);
        while(false !== ( $file = readdir($dir)) ) {
            if (( $file != '.' ) && ( $file != '..' )) {
                if ( is_dir($source_directory.'/'.$file) ) {
                    self::recursive_copy($source_directory.'/'.$file, $destination_directory .'/'. $file);
                } else {
                    $explodedFileName = explode('.', $file);
                    $fileExtension = array_pop($explodedFileName);
                    $destinationFileName = implode('.', $explodedFileName).'-'.md5_file($source_directory.'/'.$file).'.'.$fileExtension;
                    copy($source_directory . '/' . $file,$destination_directory . '/' . $destinationFileName);
                }
            }
        }
        closedir($dir);
    }
}
Asset::fingerprint_assets();
```
Let us consider that all your assets reside in the asset folder (located in CodeIgniter applications base path). Execute
the file Asset.php from the base path of your CodeIgniter application. This generates the fingerprinted files for
production use(prod folder). When you use the following code in your layout:

```
<link rel="stylesheet" href="<?php echo asset_url('css/style.css'); ?>">
<script src="<?php echo asset_url('js/zepto.js'); ?>"></script>
```

The helper function produces the following output:-
```
<link rel="stylesheet" href="https://example.com/prod/css/style-8b6aee697a2a565794c5b38a5f80b02a.css">
<script src="https://example.com/prod/js/zepto-50a4556b0089cfa1cb61e88ea23bbcce.js"></script>
```

The assets filename is appended with the `md5_sum` value of the file. When the asset file is modified, the `md5_sum` of
the file changes. This generates a new filename. As your browser does not have the new file, the browser requests a
fresh copy from the server. This in turn loads the new asset to the user.

The above code can be improved. Rather than executing the method `asset_url()` for each request of your page load -
another data type can be referenced. Create a HASH (key => value array) every time the production assets are
fingerprinted. Reference this HASH in the function `asset_url()`. I leave this to you, the developer to figure out.

If you use this procedure, all asset files on your production will be appended with a fingerprint. This should further
speed up your website and avoid caching problems.

Cheers and Happy Coding!

_Note: As this blog post is meant to provide a very high-level overview, explaining each component of the implementation
is beyond the scope of this article. I may follow this up with a series of posts explaining the specifics as time permits._
