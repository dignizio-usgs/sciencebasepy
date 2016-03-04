Python ScienceBase Utilities
============================
This Python module provides some basic services for interacting with ScienceBase.  It requires the "requests"
module to be installed, which can be found at
[http://docs.python-requests.org/en/latest/](http://docs.python-requests.org/en/latest/)
If you get security errors also install requests[security]

Quick Start
-----------
Install the pysb libraries into your python installation by running `python setup.py install`.  Example usage is contained in `demo.py`.

Module Contents
---------------
The SbSession class provides the following methods:

### Login
* `login(username, password)`
Log into ScienceBase using the username and password.  This causes a cookie to be added to the session
to be used by subsequent calls.

* `loginc(username)`
Log into ScienceBase using the given username, and prompt for the password.  The password is not
echoed to the console.  Provided as a convenience for interactive scripts.

* `is_logged_in()`
Return whether the SbSession is logged in and active in ScienceBase

* `get_session_info()`
Return ScienceBase Josso session info

* `ping()`
Ping ScienceBase.  A very low-cost operation to determine whether ScienceBase is available.

* `logout()`
Log out of ScienceBase

### Create
* `create_item(sb_json)`
Create a new ScienceBase item.  Documentation on the sb_json format can be found at
https://my.usgs.gov/confluence/display/sciencebase/ScienceBase+Item+Core+Model

### Read
* `get_item(id)`
Get the JSON for the ScienceBase item with the given ID.

* `get_my_items_id()`
Get the ID of the logged in user's "My Items"

* `get_child_ids(parentid)`
Get the IDs of all children of the ScienceBase item with the given ID.

* `get(url)`
Get the text response of the given URL.

* `get_json(url)`
Get the JSON response of the given URL.

* `get_item_files_zip(sb_json, destination)`
Get a zip of all files attached to the ScienceBase item and place it in the destination
folder.  Destination defaults to the current directory.  If specified, the destination folder
must exist.  This creates the zip file server-side and then streams it to the client.

* `get_item_files(sb_json, destination)`
Download all files attached to the ScienceBase item and place them in the destination folder.
Destination defaults to the current directory.  If specified, the destination folder must
exist.  The files are streamed individually.

* `get_item_file_info(sb_json)`
Get information about all files attached to a ScienceBase item.  Returns a list of
dictionaries containing url, name and size of each file.

* `download_file(url, local_filename, destination)`
Download an individual file.  Destination defaults to the current directory.  If specified,
the destination folder must exist.

### Update
* `update_sb_item(sb_json)`
Updates an existing ScienceBase item.

* `upload_file_to_item(sb_json, filename)`
Upload a file to an existing ScienceBase item.

* `upload_file_and_create_item(parentid, filename)`
Upload a file and create a ScienceBase item.

* `upload_files_age_create_item(parentid, [filename,...])`
Upload a set of files and create a ScienceBase item.

* `upload_files_and_update_item(item, [filename,...])`
Upload a set of files and update an existing ScienceBase item.

* `upload_files_and_upsert_item(item, [filename,...])`
Upload multiple files and create or update a ScienceBase item.

* `replace_file(filename, item)`
Replace a file on a ScienceBase Item.  This method will replace all files named the same as the new file,
whether they are in the files list or in a facet.

* `upload_file(filename, mimetype)`
(Advanced usage) Upload a file to ScienceBase.  The file will be staged in a temporary area.  In order
to attach it to an Item, the pathOnDisk must be added to an Item files entry, or one of a facet's file entries.

### Delete
* `delete_item(sb_json)`
Delete an existing ScienceBase item.

* `delete_items(itemIds)`
Delete multiple ScienceBase Items.  This is much more efficient than using delete_item() for multiple deletions, as it
performs the action server-side in one call to ScienceBase.

* `undelete_item(itemid)`
Undelete a ScienceBase item.

### Move
* `move_item(itemid, parentid)`
Move the ScienceBase Item with the given itemid under the ScienceBase Item with the given parentid.

* `move_items(itemids, parentid)`
Move all of the ScienceBase Items with the given itemids under the ScienceBase Item with the given parentid.

### Search
* `find_items_by_any_text(text)`
Find items containing the given text somewhere in the item.

* `find_items_by_title(text)`
Find items containing the given text in the title of the item.

* `find_items(params)`
Find items meeting the criteria in the specified search parameters.  These are the same parameters that you pass
to ScienceBase in an advanced search.

* `next(items)`
Get the next page of results, where *items* is the current search results.

* `previous(items)`
Get the previous page of results, where *items* is the current search results.

### Shortcuts
* `get_shortcut_ids(itemid)`
Get the IDs of all ScienceBase Items shortcutted from a given item.
    
* `create_shortcut(itemid, parentid)`
Create a shortcut on the ScienceBase Item with the id parentid to another Item with id itemid.
        
* `remove_shortcut(itemid, parentid)`
Remove a shortcut from the ScienceBase Item with the id parentid to another Item with id itemid.

Example Usage
-------------

````python
    import pysb
    import os

    sb = pysb.SbSession()

    # Get a public item.  No need to log in.
    item_json = sb.get_item('505bc673e4b08c986b32bf81')
    print "Public Item: " + str(item_json)

    # Get a private item.  Need to log in first.
    username = raw_input("Username:  ")
    sb.loginc(str(username))
    # Need to wait a bit after the login or errors can occur
    time.sleep(5)
    item_json = sb.get_item(sb.get_my_items_id())
    print "My Items: " + str(item_json)

    # Create a new item.  The minimum required is a title for the new item, and the parent ID
    new_item = {'title': 'This is a new test item',
        'parentId': sb.get_my_items_id(),
        'provenance': {'annotation': 'Python ScienceBase REST test script'}}
    new_item = sb.create_item(new_item)
    print "NEW ITEM: " + str(new_item)

    # Upload a file to the newly created item
    new_item = sb.upload_file_to_item(new_item, 'pysb.py')
    print "FILE UPDATE: " + str(new_item)

    # List file info from the newly created item
    ret = sb.get_item_file_info(new_item)
    for fileinfo in ret:
        print "File " + fileinfo["name"] + ", " + str(fileinfo["size"]) + "bytes, download URL " + fileinfo["url"]

    # Download zip of files from the newly created item
    ret = sb.get_item_files_zip(new_item)
    print "Downloaded zip file " + str(ret)

    # Download all files attached to the item as individual files, and place them in the
    # tmp directory under the current directory.
    path = "./tmp"
    if not os.path.exists(path):
        os.makedirs(path)
    ret = sb.get_item_files(new_item, path)
    print "Downloaded files " + str(ret)

    # Delete the newly created item
    ret = sb.delete_item(new_item)
    print "DELETE: " + str(ret)

    # Upload multiple files to create a new item
    ret = sb.upload_files_age_create_item(sb.get_my_items_id(), ['pysb.py','readme.md'])
    print str(ret)

    # Search
    items = sb.find_items_by_any_text(username)
    while items and 'items' in items:
        for item in items['items']:
            print item['title']
        items = sb.next(items)

    # Logout
    sb.logout()
````
Copyright and License
---------------------
This USGS product is considered to be in the U.S. public domain, and is licensed under
[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).

Although this software program has been used by the U.S. Geological Survey (USGS), no warranty, expressed or implied,
is made by the USGS or the U.S. Government as to the accuracy and functioning of the program and related program
material nor shall the fact of distribution constitute any such warranty, and no responsibility is assumed by the
USGS in connection therewith.
