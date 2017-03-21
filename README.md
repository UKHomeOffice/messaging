# Messaging API Dev Scripts

## Usage
Messages and translations can be added by means of the various scripts in the `messaging` directory.
  * `messaging/push`
  * `messaging/pull`
  * `messaging/delete`
  * `messaging/generate`
  
For your convenience, each of the scripts can be called with a `-h` or `--help` option to display a help message reminding you of the script's usage options. 

### Working locally
While working locally, you can override the messages pulled from the messaging API by creating a JSON file in `messages_dir`. This file should take the format `[story number]_[language code].json` - for example, `DJ-1234_en.json`.

The file should contain the new messages you wish to use (either overwriting the values of existing terms or adding new ones) in JSON format (the `generate` script will create this for you, see below):
```json
[
  {
    "term": "ref.alternateLanguages.en",
    "definition": "English-New"
  },
  {
    "term": "ref.alternateLanguages.zh",
    "definition": "Chinese-New"
  }
]
```

When the application is loaded, the 'local' terms will be loaded in preference of any existing terms, allowing you to see the changes before pushing them.

#### Generate a local working file
##### `messaging/generate`
To generate a file which fits the above criteria, you can use the `messaging/generate` script. By default, the script will prompt for a story number and generate a file for each default language containing a single term-definition pair. The following options can also be used:
  * `-n` to set the number of entries (term-definition pairs) to include in the file
  * `-s` to set the story number (no prompt will be shown in this case)
  * `-l` to generate a file only for one specific language
  * `-t` to add an additional 'tag' which can be used in the translation tool to identify certain messages. To add more than one tag, put the entire set of tags in a single set of quotes and comma-separate them (for example `-t "new_terms, release_55"`)

There is an entry in `.gitignore` for these files which prevents them being added in stash/git     

The following are valid usages:
  * `messaging/generate` - will prompt for the story number and create one file for each language, each containing one entry
  * `messaging/generate -n 3` - will prompt for the story number and create a file for each language, each containing three entries
  * `messaging/generate -l en` - will prompt for the story number and create one file for the language 'en' containing one entry
  * `messaging/generate -n 6 -s DJ-1234` - will create a file for each language with the story number DJ-1234, each containing six entries
  * `messaging/generate -s DJ-1234 -n 5 -l zh` - will create one file for language 'zh' with story number DJ-1234, containing five entries
  * `messaging/generate -t release_55` - will prompt for the story number and create a file for each language with one entry each, with an additional tag of 'release_55'
  * `messaging/generate -t "release_55, new_terms"` - will prompt for the story number and create a file for each language with one entry each, with two additional tags, "release_55" and "new_terms"

### Pulling messages
##### `messaging/pull`
Run `messaging/pull` to pull the up-to-date messages from the API and insert them into the local messages files (which take the format `messages_[languade code].json` e.g. `messages_en.json`, `messages_zh.json`).

Running this script will overwrite the contents of the file.

Since you will typically want to avoid pulling all translations, the `messaging/pull` provides a `-l` option which enables you to pull only the English messages as follows:
    * `messaging/pull -l en`

### Pushing messages
##### `messaging/push`
Once you are happy with your changes to the messages in your local working file, you can use the `messaging/push` script to push your local messages, which should be in a file in the format described above. By default, the script will upload all files beginning `DJ-` from the `messages_dir` directory.

If you have only one active story in progress (so have only one local working file) then running the script with no parameters will do exactly what you need.

But if you have two or more active stories, each having their own local working file, then when you complete the first story you will probably not want to upload the messages for any other stories than the one you have completed. Therefore the `messaging/push` script allows you to specify a `-s` parameter to name just one file to be pushed as follows:
    * `messaging/push -s DJ-1234`
 
Whether you push one or all local working files, you will receive a message back from the API indicating whether or not the push was successful, and giving a summary of the changes you have made (for example, the number of new terms you have added).

### Deleting keys
#### `messaging/delete`
Deletion of keys can be performed using the `messaging/delete` script.

**Note:** this operation will delete the key and **all** translations and therefore should not be used to simply remove a translation which is not required.

The script can take two options:
  * `-f` to specify a plain text file which contains the keys to be deleted. The keys should each be on a new line with no separator
  * `-s` to specify the story number that the deletion relates to
  * the script can also be called by passing a comma-separated list of keys to delete directly (optionally with the `-s` option)
  * if the `-s` option is not specified, the user will be prompted to enter the story number
  * if both `-s` and `-f` options are specified, any additional arguments specified will be ignored

The following are all valid usages:
  * `messaging/delete -s DJ-1234 -f terms.txt` - will delete the keys in terms.txt
  * `messaging/delete -f terms.txt` - will prompt for the story number and delete the keys in terms.txt
  * `messaging/delete -s DJ-1234 ref.alternateLanguages.en, ref.countries.ISL - will delete the keys shown
  * `messaging/delete ref.completeCountriesListing.IDN, calendarPicker.wednesday` - will prompt for the story number and delete the keys shown
  
### Developer Workflow
  1. [Generate a local working file](#Generate-a-local-working-file).
  2. [Work locally, by adding/updating messages in the local working file](#Working-locally).
  3. Before pushing your code changes to master, [push your new/updated messages to the server](#Pushing-messages).
  4. [Pull the updated messages](#Pulling-messages). You will most likely only need to pull the English messages. Since we are not adding messages to the master messages file, you should never get a conflict when you pull the messages or pull the customer project.
  5. At this stage you may wish to verify that your new messages are present in the pulled messages.
  6. Delete your local working message file(s) from your project. This is a simple file delete which can be done from IntelliJ or the command prompt.
  7. Prepare your commit, which must include the pulled message file(s), and push to master in the usual way.
