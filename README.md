# PHPScanner
PHPScanner is a small and simple script which scans uploaded content for known scripts like c99 before your PHP script handles the request

## How to Install
Via SSH/Terminal (requires root or prepend each command with `sudo`)

1. `cd /etc/php5` - Move to the php5 directory
2. `mkdir extensions` - Creates the extensions directory if it does not exist
3. `cd extensions` - Move into the extensions directory
4. `wget -O PHPScanner.zip https://github.com/ChubbyNinja/PHPScanner/archive/master.zip` - Download the PHPScanner
5. `unzip PHPScanner.zip` - Unzip the PHPScanner (You may need to `apt-get install unzip`)
6. `mv PHPScanner-master PHPScanner` - Rename the extracted folder
7. `rm PHPScanner.zip` - Remove the archive
8. `pico /etc/php5/apache2/php.ini` - Open up the php.ini file
9. Find `auto_prepend_file` under Data Handling and replace with `auto_prepend_file =/etc/php5/extensions/PHPScanner/PHPScanner.php`
10. `service apache2 restart` - restart apache2 to load PHPScanner.


If you did not want to load PHPScanner through php.ini, you can simply download the PHPScanner and do the following:
```
<?php
require( 'path/to/PHPScanner/PHPScanner.php' );
```

## php.ini vs require
There are multiple benefits to including this script at php.ini level.

1. You don't always have control over the upload scripts on your website, this is often the case with Content Management Systems that support 3rd party plugins, like Wordpress, Joomla, Drupal. PHPScanner will run before any other scripts get access to the files being uploaded. This ensures PHPScanner can take action **before** the malicious code is stored or executed.
2. You don't have to remember to include PHPScanner when you create multiple websites (VirtualHosts)
3. Probably more...


## Example Output
There is no need to change how your uploads are handled (not even for WordPress, Joomla or other Content Management Systems).


### Example 1 - Clean file
Here's an example output for `var_dump( $_FILES );`

```
array(1) {
  ["my_file"]=>
  array(7) {
    ["name"]=>
    string(11) "Capture.JPG"
    ["type"]=>
    string(10) "image/jpeg"
    ["tmp_name"]=>
    string(14) "/tmp/phpJ1kFGr"
    ["error"]=>
    int(0)
    ["size"]=>
    int(127218)
    ["scan_results"]=>
    string(2) "OK"
    ["scan_details"]=>
    array(0) {
    }
  }
}
```

As you can see, there are two additional keys `scan_results` and `scan_details`.

1. `scan_results` is a string, with two values available

   **OK** - Scan completed and the file was considered clean
   
   **PUP** - Scan completed and the file was considered a Potentially Unwanted Program
   
2. `scan_details` is an array, if the file was considered a PUP, this array is filled with the strings matched
   
   It is then down to the Developer to programmatically decide if the file is indeed dangerous.
   

### Example 2 - RootKit file
Here's an example output for `var_dump( $_FILES );`

```
array(1) {
  ["my_file"]=>
  array(7) {
    ["name"]=>
    string(8) "test.txt"
    ["type"]=>
    string(10) "text/plain"
    ["tmp_name"]=>
    string(26) "/tmp/phpDrv7Jh_VIRUS_FOUND"
    ["error"]=>
    int(8)
    ["size"]=>
    int(165477)
    ["scan_results"]=>
    string(3) "PUP"
    ["scan_details"]=>
    array(7) {
      [0]=>
      array(2) {
        ["vun_id"]=>
        int(0)
        ["vun_string"]=>
        string(23) "-type f -name .htpasswd"
      }
      [1]=>
      array(2) {
        ["vun_id"]=>
        int(2)
        ["vun_string"]=>
        string(7) "netstat"
      }
      [2]=>
      array(2) {
        ["vun_id"]=>
        int(4)
        ["vun_string"]=>
        string(6) "find /"
      }
      [3]=>
      array(2) {
        ["vun_id"]=>
        int(5)
        ["vun_string"]=>
        string(9) "ccteam.ru"
      }
      [4]=>
      array(2) {
        ["vun_id"]=>
        int(10)
        ["vun_string"]=>
        string(6) "psyBNC"
      }
      [5]=>
      array(2) {
        ["vun_id"]=>
        int(13)
        ["vun_string"]=>
        string(11) "/etc/passwd"
      }
      [6]=>
      array(2) {
        ["vun_id"]=>
        int(14)
        ["vun_string"]=>
        string(23) "packetstormsecurity.org"
      }
    }
  }
}
```

There's a couple of things to notice above

1. As you can see, the `tmp_name` has been appended with `_VIRUS_FOUND` - the decision to do this instead of remove the file completely is to give you (the developer) the flexibility of deciding if you should continue. - The default action will be to fail
2. The `error` number has changed to `8`.
3. The `scan_results` are now `PUP`
4. The `scan_details` now gives you a multidimensional array
    `vun_id` is the id in our definitions.php
    `vun_string` is the matched string
    
    

### Example 3 - multiple files attribute
You can upload multiple files through one input box - PHPScanner takes this into consideration too.
Here's an example output for `var_dump( $_FILES );`

```
array(2) {
  ["my_files"]=>
  array(7) {
    ["name"]=>
    array(2) {
      [0]=>
      string(8) "test.txt"
      [1]=>
      string(11) "Capture.JPG"
    }
    ["type"]=>
    array(2) {
      [0]=>
      string(10) "text/plain"
      [1]=>
      string(10) "image/jpeg"
    }
    ["tmp_name"]=>
    array(2) {
      [0]=>
      string(26) "/tmp/phpPPAuIS_VIRUS_FOUND"
      [1]=>
      string(14) "/tmp/phpr0GhA7"
    }
    ["error"]=>
    array(2) {
      [0]=>
      int(8)
      [1]=>
      int(0)
    }
    ["size"]=>
    array(2) {
      [0]=>
      int(165477)
      [1]=>
      int(127218)
    }
    ["scan_results"]=>
    array(2) {
      [0]=>
      string(3) "PUP"
      [1]=>
      string(2) "OK"
    }
    ["scan_details"]=>
    array(2) {
      [0]=>
      array(7) {
        [0]=>
        array(2) {
          ["vun_id"]=>
          int(0)
          ["vun_string"]=>
          string(23) "-type f -name .htpasswd"
        }
        [1]=>
        array(2) {
          ["vun_id"]=>
          int(2)
          ["vun_string"]=>
          string(7) "netstat"
        }
        [2]=>
        array(2) {
          ["vun_id"]=>
          int(4)
          ["vun_string"]=>
          string(6) "find /"
        }
        [3]=>
        array(2) {
          ["vun_id"]=>
          int(5)
          ["vun_string"]=>
          string(9) "ccteam.ru"
        }
        [4]=>
        array(2) {
          ["vun_id"]=>
          int(10)
          ["vun_string"]=>
          string(6) "psyBNC"
        }
        [5]=>
        array(2) {
          ["vun_id"]=>
          int(13)
          ["vun_string"]=>
          string(11) "/etc/passwd"
        }
        [6]=>
        array(2) {
          ["vun_id"]=>
          int(14)
          ["vun_string"]=>
          string(23) "packetstormsecurity.org"
        }
      }
      [1]=>
      array(0) {
      }
    }
  }
  ["test2"]=>
  array(5) {
    ["name"]=>
    string(0) ""
    ["type"]=>
    string(0) ""
    ["tmp_name"]=>
    string(0) ""
    ["error"]=>
    int(4)
    ["size"]=>
    int(0)
  }
}
```

## Example Triggers

You can trigger a manual scan for either files or strings

```
var_dump( $PHPScanner->manual_scan_string('cat /etc/passwd') );
var_dump( $PHPScanner->manual_scan_string('Some safe string with nothing bad on it') );
var_dump( $PHPScanner->manual_scan_file('/var/www/uploads/file.php') );
```

Outputs
```
array(3) {
  ["msg"]=>
  string(9) "PUP Found"
  ["found"]=>
  array(1) {
    [0]=>
    array(2) {
      ["vun_id"]=>
      int(13)
      ["vun_string"]=>
      string(11) "/etc/passwd"
    }
  }
  ["status"]=>
  string(3) "PUP"
}
array(2) {
  ["msg"]=>
  string(10) "File clean"
  ["status"]=>
  string(2) "OK"
}
array(3) {
  ["msg"]=>
  string(9) "PUP Found"
  ["found"]=>
  array(1) {
    [0]=>
    array(2) {
      ["vun_id"]=>
      int(0)
      ["vun_string"]=>
      string(11) "-type f -name .htpasswd"
    }
  }
  ["status"]=>
  string(3) "PUP"
}
```
    
#### TODO

1. Email webmaster with diagnostic and client information when PUP has been found
2. Auto-Update definitions list
