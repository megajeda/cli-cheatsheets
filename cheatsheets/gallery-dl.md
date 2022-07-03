# Gallery-DL



##  Options:

  General options             |                             |
  ----------------------------|----------------------------
  `-h, --help`                | Print help message and exit
  `--version`                 | Print program version and exit
  `-i, --input-file FILE`     | Download URLs found in FILE ('-' for stdin). More than one --input-file can be specified
  `-d, --destination PATH`    | Target location for file downloads
  `-D, --directory PATH`      | Exact location for file downloads
  `-f, --filename FORMAT`     | Filename format string for downloaded files ('/O' for "original" filenames)
  **Output options**          |
  `-q, --quiet`               | Activate quiet mode
  `-v, --verbose`             | Print various debugging information
  `-s, --simulate`            | Simulate data extraction; do not download anything
  `-E, --extractor-info`      | Print extractor defaults and settings
  `-K, --list-keywords`       | Print a list of available keywords and example values for the given URLs
 **Downloader options**       |
  `--filesize-min SIZE`       | Do not download files smaller than SIZE (e.g. 500k or 2.5M)
  `--filesize-max SIZE`       | Do not download files larger than SIZE (e.g. 500k or 2.5M)
  `--no-skip`                 | Do not skip downloads; overwrite existing files
  `--no-download`             | Do not download any files
 **Configuration options**    |
  `-c, --config FILE`         | Use a specific/additional configuration file
  `-o, --option OPT`          | Additional `'<key>=<value>'` option values for extractor run
  `--ignore-config`           | Do not read the default configuration files
 **Selection options**        |
  `-A, --abort N`             | Stop current extractor run after N consecutive file downloads were skipped
  `-T, --terminate N`         | Stop current and parent extractor runs after N consecutive file downloads were skipped
  `--range RANGE`             | Index-range(s) specifying which images to download. For example '5-10' or '1,3-5,10-'
  `--chapter-range RANGE`     | Like '--range', but applies to manga-chapters and other delegated URLs
  `--filter EXPR`             | Python expression controlling which images to download.
  `--chapter-filter EXPR`     | Like `'--filter'`, but applies to manga-chapters and other delegated URLs


### Specifying files to download

Specifying files is done with a filter and/or a Python expression

**Filter examples**                                               |                                               |
------------------------------------------------------------------|----------------------------------------------
**Use --range to specify limits on number of results**            |
`--range '1-500'`                                                 | Download the first 500 results
`--range '5-10'`                                                  | Skip the first 4 results, download results 5-10
`--range '1,3-5,10-'`                                             | Download first, third to fifth, and all results starting from 10
**Use --filters to specify criteria based on extractor key**      |
`--filter "extension == 'png'"`                                   | Download files based on filename extension.
`--filter "extension == 'jpg' or extension == 'png'"`             | Conditions can be combined with 'and' and 'or'
`--filter "extension not in ('gif', 'mp4', 'swf')"`               | Or you use 'in' or 'not in'
`--filter "'query' in title"`                                     | Search in titles for 'query'
`--filter "'query' in title or 'query' in tags"`                  | Search in titles and tags for 'query'
`--filter "'TAGNAME' in tags"`                                    | Download a file if TAGNAME is found in the file's tags.
`--filter "any('ball' in tag for tag in tags)"`                   | Test whether a word is in the tags eg 'ball' in 'little ball', 'blue doll', 'red ball'
`--filter "any('ball' == T.lower() for T in tags)"`               | Convert tags to lowercase with str.lower() before checking them to avoid case issues.
`--filter "num < 3"`                                              | Specify maximum number of images per post to download
`--filter "datetime(2015, 1, 1) <= date < datetime(2016, 1, 1)"`  | Extract all available files, silently ignore any file whose date is not between values
`--filter "date >= datetime(2021) or terminate()"`                | Exits at first file before 2021, requires extractor results to be correctly sorted
`--filter "'Apollo' in title.split()  and  '11' in title.split()"`| Split title into word list, test if "Apollo" and "11" are in the resulting lists
`--filter "re.search(r'\bApollo\b.*\b11\b', title)"`              | Alternative method using a python regular expression
`--filter "image_width >= 1000 and rating in ('s', 'q')"`         | Example:
**Use an image filter in extractor config**                       |
`"image-filter": "not contains(tags, ('sports', 'ball'))"`        | Syntax is "not contains(tags, ('tag1', 'tag2', 'tag3', 'tag4'))"
`"image-filter": "width >= 1200 and width/height > 1.2"`          |
**Twitter specific**                                              |
https://twitter.com/search?q=(from:HANDLE+filter:media+until:YYYY-MM-DD) |
https://twitter.com/search?q=%23MY_HASHTAG                        | Requires twitter login for hashtag search to fully work.


`gallery-dl -j --no-download --filter 'ups < -100' --chapter-filter 'ups < -100' 'https://www.reddit.com/r/EarthPorn/new/' > out.json
`jq '.[][1] | {title, ups}' out.json`

OR

`jq '.[][1] | select(.ups > 100) | {title, ups}`


## Conditional filenames/directory

You can use conditions to specify alternative filenames/directory within extractor block:

```json
    "sitename": {
        "filename": {
            "extension == 'mp4'": "video filename",
            "len(title) < 16"   : "some other filename",
            "": "default filename"
        }
        "directory": {
            "score >= 20"        : ["high score"],
            "score >= 5"         : ["mid score"],
            "'nature' in content": ["Nature Pictures"],
            "retweet_id != 0"    : ["{category}", "{user[name]}", "Retweets"],
            ""                   : ["{category}", "default"]
        }
    }
```

add the artist's nickname from the tags to the directory name using python expression

    "directory": ["\fE ', '.join([t[7:] for t in tags if t.startswith('artist:')])"]

chain a couple of replace operations to transform mp4 into vid and everything else into img:

      {extension:Rmp4/vid/Rjpg/img/Rpng/img/Rgif/img/}




    `"locals().get('is_gallery')" : "{title[:220]!t:R /_/}_{id}{num:?_//>02}.{extension}",`

`is_gallery` if that field is always defined and you want to check whether it's true

`locals().get('is_gallery')` if that field is not guaranteed to be defined and you want to check whether it's true

`'is_gallery' in locals()` if you don't care about that field's value and only want to check whether it is defined or not.

### Format strings

#### Alternatives

Specify alternatives in format strings with '|': `{title_jpn|title}`

'|' is only recognized in the data-selection part before the ':' `{date1|date2|date3:%Y%m%d}` for eg `{title_jpn[:220]|title[:220]}`

#### Slices

`{title[:220]!t:R /_/}`

takes the value of title, trims it to 220 characters `[:220]`, removes whitespace from both ends `!t`, and replaces spaces with underscores `:R     /_/`.


`[220]` accesses the 221st element in that list or string (zero-indexed),
`[:220]`, short for [0:220], returns the slice from element 0 to 219 inclusive(or less if the string is shorter).
`title[0:10:2]` returns every second character from the first ten
`title[::-1]` reverses a string.

Both fields must support any eventual format specifiers or is throws an exception. For example `{edited|date:%Y%m%d}` will not work since the string value in {edited} can't be formatted with %Y%m%d.


Using a regular expression:

  `"re.search(r'foo(bar)+', description)"`

Using multiple conditions

  `"date >= datetime(2021, 1, 1) and extension == 'mp4' and favorite_count > 20000"`

Language conditions

    `"lang == 'en'"`
    `"language == 'French' and 10 <= chapter < 20"

Python expression to convert title to lower case, and replace spaces with underscores

`{title.lower().replace(' ', '_')}`

```json

    "postprocessors": [
      {
        "name": "metadata",
        "whitelist": ["deviantart"],
        "mode": "custom",
        "content-format": "{description}",
        "extension-format": "{extension}.description.htm"
      },
      {
        "name": "exec",
        "final": true,
        "command": "mkdir -p {}/metadata && mv -t {}/metadata -- *.json"
        "alternative-command": "(if not exist {}metadata mkdir {}metadata) & (if exist {}*.json move /y {}*.json {}metadata)",
      },
      {
        "name": "exec",
        "note-description": "deletes empty description files",
        "command": "for %F in ({}.description.htm) do if %~zF==0 del \"%F\"",
        "final": false
      }
    ]
  },

```



