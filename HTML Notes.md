# Alignment

## Z-Index

### position:

#### 1
For `z-index` to work additionally the background must have some positioning (usually `position: relative`)

#### (relatively) absolute positioning

> [Advanced Positioning Tutorial | HTML & CSS Is Hard (internetingishard.com)](https://www.internetingishard.com/html-and-css/advanced-positioning/)

Coordinates for absolute elements are always relative to the closest container that is a positioned element. It only falls back to being relative to the browser when none of its ancestors are positioned. So, if we change `.item-absolute`’s parent element to be relatively positioned, it should appear in the top-left corner of _that_ element instead of the browser window.

```css
.item-absolute {
  position: absolute;
}
.absolute {
  position: relative;
}
```

The `.absolute` div is laid out with the normal flow of the page, and we can manually move around our `.item-absolute` wherever we need to. This is great, because if we want to alter the normal flow of the container, say, for a mobile layout, any absolutely positioned elements will automatically move with it.

# Design

## Color

### Defaults

It's important to know that divisions often appear white because the default color of the body is also white. It's important to remember that they're transparent unless you specify the `background` property.
This came up when I believed that `z-index` wasn't working properly and I assumed that the index didn't apply to background colors.

## Also see

[[Regex Magic]]
