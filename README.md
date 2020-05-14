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

## Licence

GPL-3.0

## Author

[Brendan Graetz](http://bguiz.com/)
