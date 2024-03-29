## Overview

[Glide](https://github.com/bumptech/glide) is an Image Loader Library for Android developed by bumptech and is a library that is recommended by Google. It has been used in many Google open source projects including Google I/O 2014 official application.  It provides animated GIF support and handles image loading/caching.  

**NOTE: Glide v4 has breaking API changes.  See this [link](Displaying-Images-with-the-Glide-v3-Library) for the older v3 version.**

### Setup

Add to your `app/build.gradle` file:

```gradle
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.8.0'
  // Glide v4 uses this new annotation processor -- see https://bumptech.github.io/glide/doc/generatedapi.html
  annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
}
```

Make sure to create a `MyAppGlideModule` that simply extends from `AppGlideModule` and has the `@GlideModule` annotation.  For now, the class is empty but later we will show how it can be used to set the default image resolution.  If you upgrading from Glide v3, make sure you follow this step too:

```java
import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

// new since Glide v4
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {
  // leave empty for now
}
```

Make sure to sync your project to Gradle before continuing, since Glide needs to generate the necessary code to invoke `GlideApp.with()` in Android Studio.

### Basic Usage

If you are migrating from Glide v3, make sure to review [this guide](https://bumptech.github.io/glide/doc/migrating.html).  Instead of `Glide.with()`, you will need to use `GlideApp.with()`:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .into(ivImg);
```

### Advanced Usage

Resizing images with:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .override(300, 200)
    .into(ivImg);
```

Placeholder and error images:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.imagenotfound)
    .into(ivImg);
```

Cropping images with:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .centerCrop()
    .into(ivImg);
```

### Configuration

Modify your `MyAppGlideModule` to override applyOptions:

```java
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    // Glide default Bitmap Format is set to RGB_565 since it 
    // consumed just 50% memory footprint compared to ARGB_8888.
    // Increase memory usage for quality with:

    builder.setDefaultRequestOptions(new RequestOptions().format(DecodeFormat.PREFER_ARGB_8888));
  }
}
```

### Resizing

Ideally, an image's dimensions would match exactly those of the `ImageView` in which it is being displayed, but as this is often not the case, care must be taken to resize and/or scale the image appropriately. Android's native support for this isn't robust, especially when displaying very large images (such as bitmaps returned from the camera) in smaller image views, which can often lead to errors (see Troubleshooting).

Glide automatically limits the size of the image it holds in memory to the `ImageView` dimensions. Picasso has the same ability, but requires a call to `fit()`. With Glide, if you _don't want_ the image to be automatically fitted to the `ImageView`, you can call `override(horizontalSize, verticalSize)`. This will resize the image before displaying it in the `ImageView` but _without_ respect to the image's aspect ratio:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .override(100, 200) // resizes the image to 100x200 pixels but does not respect aspect ratio
    .into(ivImg);
```

Resizing images in this way without respect to the original aspect ratio will often make the image appear skewed or distorted. In most cases, this should be avoided, and Glide offers two standard scaling transformation options to prevent this: `centerCrop` and `fitCenter`.

If you only want to resize one dimension, use `Target.SIZE_ORIGINAL` as a placeholder for the other dimension:

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .override(100, Target.SIZE_ORIGINAL) // resizes width to 100, preserves original height, does not respect aspect ratio
    .into(ivImg);
```

#### `centerCrop()`

Calling `centerCrop()` scales the image so that it fills the requested bounds of the `ImageView` and then crops the extra. The `ImageView` will be filled completely, but the entire image might not be displayed.

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .override(100, 200) 
    .centerCrop() // scale to fill the ImageView and crop any extra
    .into(ivImg);
```

#### `fitCenter()`

Calling `fitCenter()` scales the image so that both dimensions are equal to or less than the requested bounds of the `ImageView`. The image will be displayed completely, but might not fill the entire `ImageView`.

```java
GlideApp.with(context)
    .load("http://via.placeholder.com/300.png")
    .override(100, 200) 
    .fitCenter() // scale to fit entire image within ImageView
    .into(ivImg);
```

## Troubleshooting

### OutOfMemoryError Loading Errors

If an image or set of images aren't loading, make sure to check the Android monitor log in Android Studio. There's a good chance you might see an `java.lang.OutOfMemoryError "Failed to allocate a [...] byte allocation with [...] free bytes"` or a `Out of memory on a 51121168-byte allocation.`. This is quite common and means that **you are loading one or more large images** that have not been properly resized.

First, you have to find which image(s) being loaded are likely causing this error. For any given `Glide` call, we can fix this by **one or more of the following approaches**:

- Add an explicit width or height to the `ImageView` by setting `layout_width=500dp` in the layout file.
- Call `.override(width, height)` during the Glide load and explicitly set a width or height for the image such as: `GlideApp.with(...).load(imageUri).override(500, 500).into(...)`. 
- Try removing `android:adjustViewBounds="true"` from your `ImageView` if present and if you not calling `.override()`
- Open up your static placeholder or error images and make sure their dimensions are relatively small (< 500px width). If not, resize those static images and save them back to your project.

Applying these tips to all of your Glide image loads should resolve any out of memory issues. As a fallback, you might want to open up your `AndroidManifest.xml` and then add `android:largeHeap` to your manifest:

```xml
<application
        android:name=".MyApplication"
        ...
        android:largeHeap="true"
        ...
```

Note that this is not generally a good idea, but can be used temporarily to trigger fewer out of memory errors.

### Loading Errors

If you experience errors loading images, you can create a `RequestListener<Drawable>` and pass it in via `Glide.listener()` to intercept errors:

```java
GlideApp.with(context)
        .load("http://via.placeholder.com/300.png")
        .placeholder(R.drawable.placeholder)
        .error(R.drawable.imagenotfound)
        .listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                // log exception
                Log.e("TAG", "Error loading image", e);
                return false; // important to return false so the error placeholder can be placed
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                return false;
            }
        })
        .into(ivImg);
```


## Transformations

Transformations are supported by an additional third-party library, [glide-transformations](https://github.com/wasabeef/glide-transformations). First, add the dependencies:

```gradle
dependencies {
    implementation 'jp.wasabeef:glide-transformations:3.3.0'
    // If you want to use the GPU Filters
    implementation 'jp.co.cyberagent.android.gpuimage:gpuimage-library:1.4.1'
}
```
### Rounded Corners

```java
int radius = 30; // corner radius, higher value = more rounded
int margin = 10; // crop margin, set to 0 for corners with no crop
GlideApp.with(this)
        .load("http://via.placeholder.com/300.png")
        .transform(new RoundedCornersTransformation(radius, margin))
        .into(ivImg);
```

### Crop

Circle crop:

```java
GlideApp.with(this)
        .load("http://via.placeholder.com/300.png")
        .transform(new CircleCrop())
        .into(ivImg);
```
### Effects

Blur:

```java
GlideApp.with(this)
        .load("http://via.placeholder.com/300.png")
        .transform(new BlurTransformation())
        .into(ivImg);
```

Multiple transforms:

```java
GlideApp.with(this)
        .load("http://via.placeholder.com/300.png")
        .transform(new MultiTransformation<Bitmap>(new BlurTransformation(25), new CircleCrop()))
        .into(ivImg);
```

- [List of available transformations](https://github.com/wasabeef/glide-transformations/blob/master/README.md#transformations-1)

## Advanced Usages

### Showing a `ProgressBar`

Add a `ProgressBar` or otherwise handle callbacks for an image that is loading:

```java
progressBar.setVisibility(View.VISIBLE);

GlideApp.with(this)
        .load("http://via.placeholder.com/300.png")
        .listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(Exception e, Object model, Target<Drawable> target, boolean isFirstResource) {
                progressBar.setVisibility(View.GONE);
                return false; // important to return false so the error placeholder can be placed
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                progressBar.setVisibility(View.GONE);
                return false;
            }
        })
        .into(ivImg);
```

### Adjusting Image Size Dynamically

To readjust the `ImageView` size after the image has been retrieved, first define a `SimpleTarget<Bitmap>` object to intercept the `Bitmap` once it is loaded:

```java
private SimpleTarget target = new SimpleTarget<Bitmap>() {  
    @Override
    public void onResourceReady(Bitmap bitmap, @Nullable Transition<? super Bitmap> transition) {
        // do something with the bitmap
        // set it to an ImageView
        imageView.setImageBitmap(bitmap);
    }
};

```

Next, pass the `SimpleTarget` to Glide via `into()`:

```java
GlideApp.with(context)
        .load("http://via.placeholder.com/300.png")
        .asBitmap()
        .into(target);
```

**Note:** The `SimpleTarget` object must be **stored as a member field or method** and cannot be an anonymous class otherwise this won't work as expected.  The reason is that Glide accepts this parameter as a weak memory reference, and because anonymous classes are eligible for garbage collection when there are no more references, the network request to fetch the image may finish after this anonymous class has already been reclaimed.  See this [Stack Overflow](http://stackoverflow.com/questions/24180805/onbitmaploaded-of-target-object-not-called-on-first-load#answers) discussion for more details.

In other words, you cannot do this all inline `GlideApp.with(this).load("url").into(new SimpleTarget<Bitmap>() { ... })` as in other scenarios.

## Networking

By default, Glide uses [HttpUrlConnection](https://guides.codepath.com/android/sending-and-managing-network-requests#sending-an-http-request-the-hard-way) to handle network requests.  Integration libraries can be used to use Volley or OkHttp.  See [this guide](https://bumptech.github.io/glide/int/about.html) for more information.

### Using with OkHttp

There is a way to use Glide to use OkHttp instead, which may be useful if you need to do authenticated requests.  First, add the `okhttp3-integration` library as a dependency:

```gradle
dependencies {
  implementation 'com.squareup.okhttp3:okhttp:X.X.X'
  implementation "com.github.bumptech.glide:okhttp3-integration:4.7.1"
}
```

By adding the `okhttp3-integration` module, an OkHttp instance will be internally created and registered.  A `OkHttpLibraryGlideModule` will be used to register this component with Glide.

#### Injecting OkHttpClient

If you already have a shared instance of OkHttpClient, you can define it in your custom `MyAppGlideModule` and exclude the use of `OkHttpLibraryGlideModule` by using the `@Excludes` annotation:

```java
@GlideModule
@Excludes(OkHttpLibraryGlideModule.class) // initialize OkHttp manually
public final class MyAppGlideModule extends AppGlideModule {

    @Override
    public void registerComponents(@NonNull Context context, @NonNull Glide glide, @NonNull Registry registry) {
        super.registerComponents(context, glide, registry);
        OkHttpClient okHttpClient = new OkHttpClient();
        registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(okHttpClient));
    }
```

See this [GitHub issue](https://github.com/bumptech/glide/issues/2002) for more context about how to inject an OkHttpClient through Dagger 2.

### ProGuard

Review [this section](https://github.com/bumptech/glide#proguard) if are configuring Glide for use with ProGuard.

## References

* <https://github.com/bumptech/glide>
* <https://inthecheesefactory.com/blog/get-to-know-glide-recommended-by-google/en>
* <http://www.androidtutorialshub.com/glide-image-loading-library-for-android-tutorial/>
* <https://futurestud.io/blog/glide-getting-started>
* <https://github.com/wasabeef/glide-transformations>
* <http://google-opensource.blogspot.com/2014/09/glide-30-media-management-library-for.html>
* <http://tutorialwing.com/android-glide-library-tutorial-example/>
* <http://vardhan-justlikethat.blogspot.com/2014/09/android-image-loading-libraries-picasso.html>
* <http://www.appance.com/glide-image-loading-and-caching-library-for-android/>
* <http://www.androidhive.info/2016/04/android-glide-image-library-building-image-gallery-app/>
* <https://medium.com/@vlonjatgashi/using-glide-with-kotlin-5e345b557547/>
* <https://bumptech.github.io/glide/doc/configuration.html#applications/>