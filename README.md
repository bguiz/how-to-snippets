# How to snippets

A collection of snippets on how to do various computer-y things.

## Snippets

### Git

#### Pushed 2 branches, the 1st one is merged, and the 2nd one has the same changes as the first branch, plus some additional changes

- To avoid this scenario to begin with:
  You need to pull/ fetch the master branch,
  before adding commits to your 2nd branch.
- If you have already done this:
  Fix by doing a pull/ fetch from the master branch,
  and then merging that into your new branch:
  ```shell
  git checkout my-2nd-branch
  git fetch origin master:master
  git merge --no-ff master
  # now my-2nd-branch contains the commits from my-1st-branch
  # that have already been merged to master
  git push origin my-2nd-branch
  ```

#### List all files that have changed since branching from master

```shell
# show each of the files, names only
git diff --name-only $( git merge-base origin/master HEAD ) HEAD

# show each of the files with a status (M: modified, A: Added, etc)
git diff --name-status $( git merge-base origin/master HEAD ) HEAD

# show each of the files with number of lines added/ removed
git diff --stat $( git merge-base origin/master HEAD ) HEAD
```

### Patch a subset of changes from one branch on another

The following gets the latest 3 commits on the current branch from
2 particular files or folders into another branch,
using `patch` as an in-between.

```shell
# create the patch file
git diff HEAD~3 HEAD -- file-or-folder-1 file-or-folder-2 > patch-$( date "+%Y%m%d" ).diff

# switch to the target branch
git checkout another-branch

# perform a dry run to identify any potential merge conflicts
# no files will be overwritten by this
patch -p1 --dry-run < patch-$( date "+%Y%m%d" ).md

# overwrite the files
patch -p1  < patch-$( date "+%Y%m%d" ).md

# selectively add parts of the patch, using git add interactively
git add -p file-or-folder-1 file-or-folder-2

# discard the parts that you did not select
git checkout -- file-or-folder-1 file-or-folder-2

```

### ImageMagick

#### Create a sprite sheet from a folder of images

```bash
# Rename original files to have a `frame-` prefix,
# followed by a 3 digit sequential number
I=0 ; for F in Original*.png ; do printf -v PADDEDI "%03d" ${I} ; cp "${F}" "frame-${PADDEDI}.png" ; (( I++ )) ; done

# Use `convert -crop` from ImageMagick v6 to get just part of the picture
# WxH+X+Y : ${width}x${height}+${xOffset}+${yOffset}
# You have to work out these values manually
for F in frame-*.png ; do convert "${F}" -crop 1280x468+60+312 "foobar-${F%.png}-cropped.png" ; done

# Use `montage` to concatenate all images into a single file
# in a horizontal row
# `-resize` is to scale the picture to be smaller
# `-tile` used in this way is what tells it to be horizontal:
# ${numColumns}x${numRows}
# `-geometry` used this way makes it not leave any gaps
montage -resize 70% foobar-frame-*.png -tile $( ls foobar-frame-*.png | wc -l )x1 -geometry +0+0 foobar-sprite.png
```

### CSS

#### Create a frame based animation using a sprite sheet image

```html
<div class="sprite-transform-animation-wrapper foobar">
  <div class="sprite-transform-animation foobar"></div>
</div>
```

```css
.sprite-transform-animation-wrapper {
  margin: auto;
  overflow: hidden;
  position: relative;
  width: 100%;
  min-height: 386px;
}

.sprite-transform-animation {
  height: 100%;
  background-size: 100%, 100%;
  background-repeat: no-repeat;
  position: absolute;
  left: 0;
  top: 0;
}

```

- Do some math:
  - `width` is number of frames multiplied by 100%
  - `animation-timing-function` is number of frames minus 1

```css
.sprite-transform-animation.foobar {
  width: 1000%;
  background-image: url('foobar-sprite.png');
  animation-name: sprite-transform-animation-keyframes-foobar;
  animation-duration: 10s;
  animation-timing-function: steps(9);
  animation-iteration-count: infinite;
}

```

- Do some more math:
  - the `to` `transform` is (number of frames minus 1) divided by (number of frames) multiplied by 100%

```css
@keyframes sprite-transform-animation-keyframes-foobar {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-90%);
  }
}

```

### Javascript

#### Left pad

```javascript
String(5).padStart(5, '0');
// '00005'
```

- Ref: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/padStart

#### Capitalise first letter

```javascript
str.charAt(0).toUpperCase() + str.slice(1);
```

### Browser JS

#### Replace a DOM element with a new one

```javascript
var originalElement = document.querySelector('.original');
var newElement = document.createElement('div');
// populate `newElement` with attributes/ contents
originalElement.replaceWith(newElement);
```

Check [compatibility](https://caniuse.com/#search=replaceWith),
otherwise fall back on:

```javascript
var originalElement = document.querySelector('.original');
var newElement = document.createElement('div');
// populate `newElement` with attributes/ contents
originalElement.parentNode.replaceChild(newElement, originalElement);
```

### Shell

#### Find biggest files within a folder

```bash
ls -1Rhs $( find ${HOME} -type f -printf "%s - %p\n" | sort -n | tail -n 20 | awk '/^.* - /{print $NF}' )
```

- Replace `${HOME}` with your target directory.
- Replace `20` with your desired file count
- How it works:
  - `find` iterates over all files recursively, and here we're telling it to print the size followed by the file name
  - `sort` to arrange them in order of smallest to largest
  - `tail` to only grab the last few, that is the largest
  - `awk` to remove the file size, only have the file names now
  - `ls` wraps around the output to put the file sizes back in, but in a human readable form, e.g. `2604202398` --> `2.5G`

#### Extract subdirectory within tar gz archive into subdirectory on disk

```shell
mkdir -p ./foo/bar/baz/
tar \
  -C ./foo/bar/baz/ \
  -xvf foo-1.2.3.tar.gz foo-1.2.3/quux \
  --strip-components=2
```

Notes:

- you must `mkdir` first, `tar` will not create a directory for you
- the `-C` option tells tar where to extract to
  - this is the subdirectory on disk
  - note that this *must* come before the `-xvf` option,
    the order is significant
- `-xvf` is for *extract*, *verbose*, *file name*
  - the first thing that comes after this is the archive file name
  - the second thing that comes after this, which is optional,
    is the subdirectory within the archive file
- `--strip-components` should be set to the number of levels in the
  subdirectory within the archive
  - if not set, the subdirectory on disk will contain further
    subdirectories within it that match those within the archive,
    which *probably* is not what you want.
- use `tar -tvf foo-1.2.3.tar.gz` to see a full file listing,
  in order to determine the values needed above

#### Obtain DNS records

```shell
dig ${DOMAIN} +nostats +nocmd +nocomments
```

- the default output of dig typically contains a bunch of ifno that you probably are not interested in, so these options help you to narrow it down

#### List all files (recursively)

While `tree` is a handy tool for visual inspection of a directory,
for batch processing purposes,
you may need to list the full path of the files.

```shell
find . -type f -print
```

If you wish to exclude certain paths, use `-not -path`.
The following excludes `.git` and `node_modules` sub directories:

```shell
find . -type f \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  -print
```

#### List all file extensions (recursively)

Use the commands to list all files recursively,
and add `sed` and `sort`, like so:

```shell
find . -type f \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  -print \
  | sed 's|.*\.||' \
  | sort -u
```

Here, `sed` is used to extract the file extensions (if present),
and `sort` is used to remove duplicates from the list (`-u` for unique).

### FFmpeg

#### Record your screen and microphone

Assuming that your screen dimensions are `2560x1080` pixels,
and you want to trim off the `30` pixels from the top of the screen.

```shell
ffmpeg \
  -video_size 2560x1050 \
  -framerate 20 \
  -f x11grab \
  -i :0.0+0,30 \
  -f pulse \
  -i default \
  -ac 2 \
  rec-01.mp4
```

- To select the appropriate part of the screen,
  `-video_size 2560x1050` is used in conjunction with `-i :0.0+0,30`.
- The audio input is simply whatever is the default for your machine,
  that is selected in system settings.

## Licence

GPL-3.0

## Author

[Brendan Graetz](http://bguiz.com/)
