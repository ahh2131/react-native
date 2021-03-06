## Static Resources

Over the course of a project, it is not uncommon to add and remove many images and accidentally end up shipping images you are no longer using in the app. In order to fight this, we need to find a way to know statically which images are being used in the app. To do that, we introduced a marker on require. The only allowed way to refer to an image in the bundle is to literally write `require('image!name-of-the-asset')` in the source.

```javascript
// GOOD
<Image source={require('image!my-icon')} />

// BAD
var icon = this.props.active ? 'my-icon-active' : 'my-icon-inactive';
<Image source={require('image!' + icon)} />

// GOOD
var icon = this.props.active ? require('image!my-icon-active') : require('image!my-icon-inactive');
<Image source={icon} />
```

When your entire codebase respects this convention, you're able to do interesting things like automatically packaging the assets that are being used in your app. Note that in the current form, nothing is enforced, but it will be in the future.

### Adding Static Resources to your iOS App using Images.xcassets

![](/react-native/img/StaticImageAssets.png)

> **NOTE**: App build required for new resources
>
> Any time you add a new resource to `Images.xcassets` you will need to re-build your app through Xcode before you can use it - a reload from within the simulator is not enough.

*This process is currently being improved, a much better workflow will be available shortly.*

> **NOTE**: PNG images are required when loading with `require('image!my-icon')`
>
> At this time, only PNG images are supported in iOS. There is an [issue](https://github.com/facebook/react-native/issues/646) that is currently addressing this bug. In the meantime a quick fix is to rename your files to my-icon.png or to use the `uri` property like: `source={{ uri: 'my-icon' }}` instead of `require()`.

### Adding Static Resources to your Android app

Add your images as [bitmap drawables](http://developer.android.com/guide/topics/resources/drawable-resource.html#Bitmap) to the android project (`<yourapp>/android/app/src/main/res`). To provide different resolutions of your assets, check out [using configuration qualifiers](http://developer.android.com/guide/practices/screens_support.html#qualifiers). Normally, you will want to put your assets in the following directories (create them under `res` if they don't exist):

* `drawable-mdpi` (1x)
* `drawable-hdpi` (1.5x)
* `drawable-xhdpi` (2x)
* `drawable-xxhdpi` (3x)

If you're missing a resolution for your asset, Android will take the next best thing and resize it for you.

> **NOTE**: App build required for new resources
>
> Any time you add a new resource to your drawables you will need to re-build your app by running `react-native run-android` before you can use it - reloading the JS is not enough.

*This process is currently being improved, a much better workflow will be available shortly.*

## Network Resources

Many of the images you will display in your app will not be available at compile time, or you will want to load some dynamically to keep the binary size down. Unlike with static resources, *you will need to manually specify the dimensions of your image.*

```javascript
// GOOD
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}} />
```

## Local Filesystem Resources

See [CameraRoll](/react-native/docs/cameraroll.html) for an example of
using local resources that are outside of `Images.xcassets`.

### Best Camera Roll Image

iOS saves multiple sizes for the same image in your Camera Roll, it is very important to pick the one that's as close as possible for performance reasons. You wouldn't want to use the full quality 3264x2448 image as source when displaying a 200x200 thumbnail. If there's an exact match, React Native will pick it, otherwise it's going to use the first one that's at least 50% bigger in order to avoid blur when resizing from a close size. All of this is done by default so you don't have to worry about writing the tedious (and error prone) code to do it yourself.

## Why Not Automatically Size Everything?

*In the browser* if you don't give a size to an image, the browser is going to render a 0x0 element, download the image, and then render the image based with the correct size. The big issue with this behavior is that your UI is going to jump all around as images load, this makes for a very bad user experience.

*In React Native* this behavior is intentionally not implemented. It is more work for the developer to know the dimensions (or aspect ratio) of the remote image in advance, but we believe that it leads to a better user experience. Static images loaded from the app bundle via the `require('image!x')` syntax *can be automatically sized* because their dimensions are available immediately at the time of mounting.

For example, the result of `require('image!logo')` from the above screenshot:

```javascript
{"__packager_asset":true,"isStatic":true,"path":"/Users/react/HelloWorld/iOS/Images.xcassets/react.imageset/logo.png","uri":"logo","width":591,"height":573}
```

## Source as an object

In React Native, one interesting decision is that the `src` attribute is named `source` and doesn't take a string but an object with an `uri` attribute.

```javascript
<Image source={{uri: 'something.jpg'}} />
```

On the infrastructure side, the reason is that it allows us to attach metadata to this object. For example if you are using `require('image!icon')`, then we add an `isStatic` attribute to flag it as a local file (don't rely on this fact, it's likely to change in the future!). This is also future proofing, for example we may want to support sprites at some point, instead of outputting `{uri: ...}`, we can output `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}` and transparently support spriting on all the existing call sites.

On the user side, this lets you annotate the object with useful attributes such as the dimension of the image in order to compute the size it's going to be displayed in. Feel free to use it as your data structure to store more information about your image.

## Background Image via Nesting

A common feature request from developers familiar with the web is `background-image`. To handle this use case, simply create a normal `<Image>` component and add whatever children to it you would like to layer on top of it.

```javascript
return (
  <Image source={...}>
    <Text>Inside</Text>
  </Image>
);
```

## Off-thread Decoding

Image decoding can take more than a frame-worth of time. This is one of the major source of frame drops on the web because decoding is done in the main thread. In React Native, image decoding is done in a different thread. In practice, you already need to handle the case when the image is not downloaded yet, so displaying the placeholder for a few more frames while it is decoding does not require any code change.
