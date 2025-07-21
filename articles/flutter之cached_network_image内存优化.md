# flutter之cached_network_image内存优化

[tag]:flutter|性能|cached_network_image
[create]:2025-07-21

在flutter中，为了优化图片加载，一般会使用一些缓存策略。而最省事的，不过直接使用第三方库。而`cached_network_image`就是这么一个库: [cached_network_image](https://pub.dev/packages/cached_network_image)

## 使用

```dart
CachedNetworkImage(
  imageUrl: 'https://example.com/image.jpg',
  width: 50,
  height: 50
)
```

直接使用组件，则这张图片被下载后，则会被缓存到本地，而后续加载该图片则直接从本地缓存中读取，速度快了很多。

但是当遇到列表项中，每一项都包含有图片时，则很容易就会出现应用内存占用太高，甚至爆了的问题。

但是`cached_network_image`也给出了优化方法，它提供了四个参数用以控制内存缓存图片的大小和磁盘缓存图片的大小，如:

```dart
CachedNetworkImage(
  imageUrl: 'https://example.com/image.jpg',
  width: 50,
  height: 50,
  memCacheWidth: 150,
  memCacheHeight: 150,
  maxWidthDiskCache: 150,
  maxHeightDiskCache: 150,
)
```

其中的参数`memCacheWidth, memCacheHeight, maxWidthDiskCache, maxHeightDiskCache`表示图片被缓存到内存与磁盘时的宽高，如果图片的宽高小于这些值，则会被压缩，从而减少内存占用。

但是这个使用会产生一个问题（当前`cached_network_image`的版本为`3.3.1`），即当列表项组件被销毁时，内存不会被释放。

## 优化

产生这个问题的原因就在于我们指定了`memCacheWidth,memCacheHeight`，而它的内存释放，默认是释放原图大小的缓存。

所以为了释放被占用的内存，需要手动去释放指定大小的图片的内存缓存。

```dart
// 列表项组件内
@override
void dispose() {
  ResizeImage cacheKey = ResizeImage(CachedNetworkImageProvider(
        widget.data.imageUrl,
        scale: 1,
        maxWidth: 150,
        maxHeight: 150
      ),
      height: 150,
      width: 150
    );
    cacheKey.obtainKey(ImageConfiguration.empty).then((value) {
      ImageCache imageCache = PaintingBinding.instance.imageCache;
      imageCache.evict(value);
    });
    super.dispose();
}
```

这样，当列表项组件被销毁时，图片的内存缓存也会被释放，从而减少内存占用。

