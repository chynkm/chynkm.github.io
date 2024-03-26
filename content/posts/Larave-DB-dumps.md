---
author: "Karthik M"
title: "Simple Laravel DB backups for MySQL/MariaDB"
date: "2019-08-19"
tags:
- laravel
- database
- php
- mysql
- mariadb
---

Databases(DB) persists all critical information related to a web application. Hence DB backups are a business necessity.
I will post a simple code snippet suitable for Laravel applications to backup your DB to
[AWS S3](https://aws.amazon.com/s3/). I assume that you are using EC2 machine for hosting the Laravel application.

Since configuring and setting up a S3 bucket is out of scope for the article, I will outline the steps for reference

1. Start by creating a S3 bucket(eg: `db-backups`) which is private.
2. Enable versioning for the bucket, so that overwrite will still preserve the backups.
3. Complete creating the bucket and note down ARN(eg: `arn:aws:s3:::db-backups/*`).
4. Create an IAM Role having the necessary permissions to the S3 bucket.
5. Attach this role to the EC2 instance.

Coming back to the main topic, DB backups in Laravel. If you are using RDS as the DB store, EC2 machine won't have the
`mysqldump` command. Hence, please install them using (Ubuntu distros)
```
$ sudo apt-get install mysql-client
```
Once installed, execute the following commands in shell and note the path
```
$ which mysqldump
Output: /usr/bin/mysqldump
$ which gzip
Output: /bin/gzip
```

Update the `.env` file variables of the Laravel application

```
AWS_DEFAULT_REGION=eu-central-1
DB_AWS_BUCKET=db-backups

MYSQLDUMP_EXE=/usr/bin/mysqldump
GZIP_EXE=/bin/gzip
```

Read through the [Laravel documentation](https://laravel.com/docs/5.8/filesystem) and install packages for S3 access.
Create a new disk in `config/filesystems.php` file, eg:
```
        's3db' => [
            'driver' => 's3',
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('DB_AWS_BUCKET'),
        ],
```

To utilise Laravel configuration caching, I have created a file called `config/env.php`, where custom environment
variables are configured. Its similar to `config/app.php`, except it holds variables which are newly introduced in the
application. A sample follows:
```
<?php

return [
    'mysqldump_exe' => env('MYSQLDUMP_EXE'),
    'gzip_exe' => env('GZIP_EXE'),
];
```

I prefer to create custom classes, under the `app/Classes` directory. Feel free to use yours, but change the namespace of
the following file accordingly. Create a file `app/Classes/DbBackup.php`. The code is self explanatory.
```
<?php

namespace App\Classes;

use Illuminate\Support\Facades\Storage;

class DbBackup
{
    private $fileName, $tmpFile, $gzipFile;

    /**
     * Initialise the variables
     *
     * @return void
     */
    public function __construct()
    {
        $this->fileName = config('database.connections.mysql.database').'-'.date('Y-m-d').'.sql';
        $this->tmpFile = '/tmp/'.$this->fileName;
        $this->gzipFile = $this->tmpFile.'.gz';
    }

    /**
     * Create the backup
     *
     * @return object this
     */
    protected function createBackup()
    {
        exec(config('env.mysqldump_exe').
            ' --user='.config('database.connections.mysql.username').
            ' --host='.config('database.connections.mysql.host').
            ' --password='.config('database.connections.mysql.password').
            ' --databases '.config('database.connections.mysql.database').
            ' > '.$this->tmpFile);

        exec(config('env.gzip_exe').' '.$this->tmpFile);

        return $this;
    }

    /**
     * Upload to S3
     *
     * @return object this
     */
    protected function uploadToS3()
    {
        Storage::disk('s3db')
            ->put($this->fileName.'.gz', fopen($this->gzipFile, 'r'));

        return $this;
    }

    /**
     * Delete the temp file
     *
     * @return object this
     */
    protected function removeBackup()
    {
        sleep(2);
        unlink($this->gzipFile);
    }

    /**
     * Method which calls all sub functions
     *
     * @return void
     */
    public function init()
    {
        $this->createBackup()
            ->uploadToS3()
            ->removeBackup();
    }

}
```

Now, we require the DB backups to be run daily, only in Production systems. Hence update the method in
`app/Console/Kernel.php`.
```
    protected function schedule(Schedule $schedule)
    {
        // this code will create the automatic DB backups.
        $schedule->call('App\Classes\DbBackup@init')
            ->daily()
            ->environments(['production']);
    }
```

The above code will only run if the [Laravel scheduler](https://laravel.com/docs/5.8/scheduling#introduction) is
configured.

That's it, folks! DB backups will be created and saved to S3 daily at midnight.
