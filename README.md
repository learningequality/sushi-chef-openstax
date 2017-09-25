# OpenStax Chef

Chef for OpenStax


## Running

Run souschef

    python souschef.py

Move intermediary content into place

    mkdir content
    mv Example\ Open\ Stax.zip content/
    cd content
    unzip Example\ Open\ Stax.zip
    rm Example\ Open\ Stax.zip
    cd ..

Run sushichef

    mkdir -p chefdata/trees/
    ./sushichef.py  -v --reset --channeldir='content/Example Open Stax/Example Open Stax/' --token=<yourtokenhere>



## Installation

* [Install pip](https://pypi.python.org/pypi/pip) if you don't have it already.

* Run `pip install -r requirements.txt`

## Description
A sous chef is responsible for scraping content from a source and putting it into a folder
and csv structure (see example `sous-chef-openstax/examples/Sample Channel.zip`)



__\*\*\* A sous chef has been started for you under sous-chef-openstax/souschef.py \*\*\*__



## Using the DataWriter

The DataWriter (utils.data_writer.DataWriter) is a tool for creating channel .zip files in a
standardized format. This includes creating folders, files, and csvs that will be used to
generate a channel.



### Step 1: Open a DataWriter

The DataWriter class is meant to be used in a context. To open, add the following to your code:

```
from utils.data_writer import DataWriter
with data_writer.DataWriter() as writer:
    # Add your code here
```

You can also set a `write_to_path` to determine where the DataWriter will generate a zip file.



### Step 2: Create a Channel

Next, you will need to create a channel. Channels need the following arguments:

  - __title__ (str): Name of channel
  - __source_id__ (str): Channel's unique id
  - __domain__ (str): Who is providing the content
  - __language__ (str): Language of channel
  - __description__ (str): Description of the channel (optional)
  - __thumbnail__ (str): Path in zipfile to find thumbnail (optional)

To create a channel, call the `add_channel` method from DataWriter

```
from utils.data_writer import DataWriter

CHANNEL_NAME = "Channel name shown in UI"
CHANNEL_SOURCE_ID = "<some unique identifier>"
CHANNEL_DOMAIN = <yourdomain.org>"
CHANNEL_LANGUAGE = "en"
CHANNEL_DESCRIPTION = "What is this channel about?"

with data_writer.DataWriter() as writer:
    writer.add_channel(CHANNEL_NAME, CHANNEL_SOURCE_ID, CHANNEL_DOMAIN, CHANNEL_LANGUAGE, description=CHANNEL_DESCRIPTION)
```

To add a channel thumbnail, you must write the file to the zip folder
```
thumbnail = writer.add_file(CHANNEL_NAME, "Channel Thumbnail", CHANNEL_THUMBNAIL, write_data=False)
writer.add_channel(CHANNEL_NAME, CHANNEL_SOURCE_ID, CHANNEL_DOMAIN, CHANNEL_LANGUAGE, description=CHANNEL_DESCRIPTION, thumbnail=thumbnail)
```

The DataWriter's `add_file` method returns a filepath to the downloaded thumbnail. This method will
be covered more in-depth in Step 4.



### Step 3: Add a Folder

In order to add subdirectories, you will need to use the `add_folder` method
from the DataWriter class. `add_folder` accepts the following arguments:

  - __path__ (str): Path in zip file to find folder
  - __title__ (str): Content's title
  - __source_id__ (str): Content's original ID (optional)
  - __language__ (str): Language of content (optional)
  - __description__ (str): Description of the content (optional)
  - __thumbnail__ (str): Path in zipfile to find thumbnail (optional)

Here is an example of how to add a folder:

```
# Assume writer is a DataWriter object
TOPIC_NAME = "topic"
writer.add_folder(CHANNEL_NAME + / + TOPIC_NAME, TOPIC_NAME)
```



### Step 4: Add a File

Finally, you will need to add files to the channel as learning resources.
This can be accomplished using the `add_file` method, which accepts these
arguments:

  - __path__ (str): Path in zip file to find folder
  - __title__ (str): Content's title
  - __download_url__ (str): Url or local path of file to download
  - __license__ (str): Content's license (use le_utils.constants.licenses)
  - __license_description__ (str): Description for content's license
  - __copyright_holder__ (str): Who owns the license to this content?
  - __source_id__ (str): Content's original ID (optional)
  - __description__ (str): Description of the content (optional)
  - __author__ (str): Author of content
  - __language__ (str): Language of content (optional)
  - __thumbnail__ (str): Path in zipfile to find thumbnail (optional)
  - __write_data__ (boolean): Indicate whether to make a node (optional)

For instance:

```
from le_utils.constants import licenses

# Assume writer is a DataWriter object
PATH = CHANNEL_NAME + "/" + TOPIC_NAME + "/filename.pdf"
writer.add_file(PATH, "Example PDF", "url/or/link/to/file.pdf", license=licenses.CC_BY, copyright_holder="Somebody")
```


The `write_data` argument determines whether or not to make the file a node.
This is espcially helpful for adding supplementary files such as thumbnails
without making them separate resources. For example, adding a thumbnail to a
folder might look like the following:

```
# Assume writer is a DataWriter object
TOPIC_PATH = CHANNEL_NAME + "/" + TOPIC_NAME
PATH = TOPIC_PATH + "/thumbnail.png"
thumbnail = writer.add_file(PATH, "Thumbnail", "url/or/link/to/thumbnail.png", write_data=False)
writer.add_folder(TOPIC_PATH, TOPIC_NAME, thumbnail=thumbnail)
```



## Extra Tools

### PathBuilder

The PathBuilder is a tool for tracking folder and file paths to write to the zip file.

To initialize a PathBuilder object, you will need to specify a channel name:

```
from utils.path_builder import PathBuilder

CHANNEL_NAME = "Channel"
PATH = PathBuilder(channel_name=CHANNEL_NAME)
```

You can now build this path using `open_folder`, which will append another item to the path:

```
...
PATH.open_folder('Topic')         # str(PATH): 'Channel/Topic'
```

You can also set a path from the root directory:
```
...
PATH.open_folder('Topic')         # str(PATH): 'Channel/Topic'
PATH.set('Topic 2', 'Topic 3')    # str(PATH): 'Channel/Topic 2/Topic 3'
```


If you'd like to go back one step back in the path:
```
...
PATH.set('Topic 1', 'Topic 2')    # str(PATH): 'Channel/Topic 1/Topic 2'
PATH.go_to_parent_folder()        # str(PATH): 'Channel/Topic 1'
PATH.go_to_parent_folder()        # str(PATH): 'Channel'
PATH.go_to_parent_folder()        # str(PATH): 'Channel' (Can't go past root level)
```

To clear the path:
```
...
PATH.set('Topic 1', 'Topic 2')    # str(PATH): 'Channel/Topic 1/Topic 2'
PATH.reset()                      # str(PATH): 'Channel'
```



### Downloader (utils.downloader.py)

`downloader.py` has a `read` function that can read from both urls and file paths.
To use:

```
from utils.downloader import read

local_file_content = read('/path/to/local/file.pdf')            # Load local file
web_content = read('https://example.com/page')                  # Load web page contents
js_content = read('https://example.com/loadpage', loadjs=True)  # Load js before getting contents

```

 The `loadjs` option will load any scripts before reading the contents of the page,
 which can be useful for web scraping.



_For more examples, see `examples/openstax_souschef.py` (json) and `examples/wikipedia_souschef.py` (html)_



