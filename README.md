
# react-native-draglist

[![npm package][npm-img]][npm-url]
[![Downloads][downloads-img]][downloads-url]
[![Issues][issues-img]][issues-url]

FlatList that can be reordered by dragging its items

![show_me_reordering](https://user-images.githubusercontent.com/39933441/203197020-eb409b97-e108-4d9b-8ee4-684ae238b65b.gif)

## Why Does This Exist At All?

> Given [react-native-draggable-flatlist](https://github.com/computerjazz/react-native-draggable-flatlist/),
> why is there also this package?

Great question. `react-native-draggable-flatlist` has silky-smooth animations, contains dozens of code files, and even manipulates internal data structures in [react-native-reanimated](https://github.com/software-mansion/react-native-reanimated) to make the animations work. You should absolutely use, and prefer, `react-native-draggable-flatlist`, if it works for you.

`react-native-draglist` exists because `react-native-reanimated`, which `react-native-draggable-flatlist` depends on, randomly hangs and crashes apps through a variety of [issues](https://github.com/software-mansion/react-native-reanimated/issues), several of which have not been fixed despite several major "stable" releases. Furthermore, the hangs and crashes are both frequent and hard to reliably reproduce, making their timely resolution unlikely.

## What Is `react-native-draglist`
This package is a basic version of `react-native-draggable-flatlist` without dependencies on anything except `react` and `react-native`. Specifically, it is deliberately built to avoid `react-native-reanimated` and its hanging/crashing issues.

It is limited in that it does not animate as smoothly (though it does `useNativeDriver`). It supports both vertical and horizontal lists.

# Installation
With no dependencies outside of `react-native` and `react`, this package installs easily:
```
npm install react-native-draglist
```
or
```
yarn add react-native-draglist
```

# Use

## Typical Flow
1. Set up `DragList` much like you do any `FlatList`, except with a `renderItem` that calls `onDragStart` at the appropriate time and `onDragEnd` in `onPressOut`.
2. When `onReordered` gets called, update the ordering of `data`.

That's basically it.

## Show Me The Code

```TSX
import React, {useState} from 'react';
import {StyleSheet, Text, TouchableOpacity, View} from 'react-native';
import DragList, {DragListRenderItemInfo} from 'react-native-draglist';

const SOUND_OF_SILENCE = ['hello', 'darkness', 'my', 'old', 'friend'];

export default function DraggableLyrics() {
  const [data, setData] = useState(SOUND_OF_SILENCE);

  function keyExtractor(str: string, _index: number) {
    return str;
  }

  function renderItem(info: DragListRenderItemInfo<string>) {
    const {item, onDragStart, onDragEnd, isActive} = info;

    return (
      <TouchableOpacity
        key={item}
        onPressIn={onDragStart}
        onPressOut={onDragEnd}>
        <Text>{item}</Text>
      </TouchableOpacity>
    );
  }

  async function onReordered(fromIndex: number, toIndex: number) {
    const copy = [...data]; // Don't modify react data in-place
    const removed = copy.splice(fromIndex, 1);

    copy.splice(toIndex, 0, removed[0]); // Now insert at the new pos
    setData(copy);
  }

  return (
      <DragList
        data={data}
        keyExtractor={keyExtractor}
        onReordered={onReordered}
        renderItem={renderItem}
      />
  );
}
```

## API

All `FlatList` properties are supported, with the following extensions/modifications:
- `renderItem` is now passed a `DragListRenderItemInfo`, which extends `ListRenderItemInfo` with these additional fields:

|Field|Type|Note|
|--|--|--|
|`onDragStart`|`() => void`|Your item should call this function when you detect a drag starting (i.e. when the user wants to begin reordering the list). A common implementation is to have a drag handle on your item whose `onPressIn` calls `onDragStart`. Alternatively, you could have an `onLongPress` call this, or use any other mechanism that makes most sense for your UI. *DragList* will not start rendering items as being dragged until you call this.
|`onDragEnd`|`() => void`|Your item should call this function when you detect a tap or drag ending. A common implementation is to have a drag handle whose `onPressOut` calls `onDragEnd`. If you don't call this during `onPressOut`, *DragList* will not realize your item is no longer active if the user taps but doesn't drag (because you will have called `onDragStart`, and yet *DragList* couldn't capture the pan responder from you because the user hasn't moved, thus it doesn't know when the user releases).
|`isActive`|`boolean`|This is `true` iff the current item is actively being dragged by the user. This can be used to render the item differently while it's being dragged (e.g. less opacity, different background color, borders, etc).

- `async onReordered(fromIndex: number, toIndex: number)` is called once the user drops a dragged item in its new position. This is *not called* if the user drops the item back in the spot it started. `DragList` will await this function and not reset its UI until it completes, so you can make modifications to the underlying data before the list resets its state.
  - `fromIndex` will be between `[0, data.length)` (that is, any valid index from the items you gave it).
  - `toIndex` reflects the position to which the item should be moved in the pre-modified `data`. It will never equal `fromIndex`. So, for instance, if `toIndex` is `0`, you should make `data[fromIndex]` the first element of `data`. **Note**: if the user drags the item to the very end of the list, `toIndex` will equal `data.length` (i.e. it will reference an index that is one beyond the end of the list -- the range of values is `[0, data.length]`).
- `onHoverChanged(index: number)` (optional): called whenever an item being dragged changes its index in the list. Note this is only called when the item hasn't been dropped into its final (potentially new) index yet — it's called as the item hovers around various indices it could be dropped at.
- `ref: React.RefObject<FlatList<T>>` (optional): You can optionally pass a ref, which DragList will tunnel through to the underlying FlatList (via `forwardRef`). This is useful, for instance, if you want to `scrollToIndex` yourself on the underlying list.
- `CustomFlatList: typeof FlatList` (optional): You can pass any component that implements the same interface as `FlatList`. Note: the component needs to support all sorts of `FlatList` things (e.g. `ref`, `scrollToPos`, etc) — i.e. it needs to implement the whole `FlatList` interface, not be just a `React.ComponentType<FlatListProps<T>>`. 

## Example Included
To play with the list, you can run the example within `example/`:

```console
npm install
cd example
npm install
npm run android   # or `npm run ios`, which takes longer to build
```

# FAQs

## How can I contribute?
Thanks for being willing! Please see
[CONTRIBUTING.md](https://github.com/fivecar/react-native-draglist/blob/main/CONTRIBUTING.md). I'd
love your help.

## What about lists with multiple columns?
This package makes no attempt to handle multi-column lists. I'm happy to look at PRs that attempt
such things, but I suspect most attempts will be fraught with issues because the UX for dragging in
a multi-column list isn't immediately obvious, especially when the underlying `FlatList`
implementation can't be controlled from the outside.

## Do you have caveats?
This package is implemented with probably 1/10th the files, and 1/20th the advanced concepts, as `react-native-draggable-flatlist`. The latter even directly modifies unpublished internal data structures of `react-native-reanimated`, so it's all sorts of advanced in ways that this package will never be. You should prefer, and default to, using `react-native-draggable-flatlist` unless its random hangs and crashes bother you.

If you have suggestions, or better yet, PRs for how this package can be improved, [please connect via GitHub](https://github.com/fivecar/react-native-draglist/)!

[downloads-img]:https://img.shields.io/npm/dt/react-native-draglist
[downloads-url]:https://www.npmtrends.com/react-native-draglist
[npm-img]:https://img.shields.io/npm/v/react-native-draglist
[npm-url]:https://www.npmjs.com/package/react-native-draglist
[issues-img]:https://img.shields.io/github/issues/fivecar/react-native-draglist
[issues-url]:https://github.com/fivecar/