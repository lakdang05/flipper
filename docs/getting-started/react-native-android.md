---
id: react-native-android
title: Manually set up your React Native Android App
sidebar_label: React Native for Android
---

These instructions are aimed at people manually adding Flipper to a React Native 0.62+ app.
This should only be necessary if you have an existing app that cannot be upgraded with the
[React Native Upgrade tool](https://reactnative.dev/docs/upgrading).

## Dependencies

Flipper is distributed via JCenter. Add the dependencies to your `build.gradle` file.
You should also explicitly depend on [`soloader`](https://github.com/facebook/soloader)
instead of relying on transitive dependency resolution which is getting deprecated
with Gradle 5.

```groovy
repositories {
  jcenter()
}

dependencies {
  debugImplementation('com.facebook.flipper:flipper:0.35.0') {
    exclude group:'com.facebook.fbjni'
  }

  debugImplementation('com.facebook.flipper:flipper-network-plugin:0.35.0') {
    exclude group:'com.facebook.flipper'
  }
}
```

These exclusions are currently necessary to avoid some clashes with FBJNI
shared libraries.

## Application Setup

For maximum flexibility, we recommend you move the Flipper initialization to a separate
class that lives in a `debug/` folder, so that Flipper code never gets referenced in a
release build.

```java
import android.content.Context;
import com.facebook.flipper.android.AndroidFlipperClient;
import com.facebook.flipper.android.utils.FlipperUtils;
import com.facebook.flipper.core.FlipperClient;
import com.facebook.flipper.plugins.inspector.DescriptorMapping;
import com.facebook.flipper.plugins.inspector.InspectorFlipperPlugin;
import com.facebook.react.ReactInstanceManager;
import okhttp3.OkHttpClient;

public class ReactNativeFlipper {
  public static void initializeFlipper(Context context, ReactInstanceManager reactInstanceManager) {
    if (FlipperUtils.shouldEnableFlipper(context)) {
      final FlipperClient client = AndroidFlipperClient.getInstance(context);

      client.addPlugin(new InspectorFlipperPlugin(context, DescriptorMapping.withDefaults()));
    }
  }
}
```

Note that this only enables the Layout Inspector plugin. Check out [the React Native template](https://github.com/facebook/react-native/blob/6f627f684bb6506a677c9632b2710e4a541690a9/template/android/app/src/debug/java/com/helloworld/ReactNativeFlipper.java) for more plugins.

In your `Application` implementation, we then call the static method using
reflection. This gives us a lot of flexibility, but can be quite noisy.
Alternatively, recreate an empty `ReactNativeFlipper` class in a `release/` folder,
so you can call into the method directly.

```java
public class MainApplication extends Application implements ReactApplication {
  // ...

  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, /* native exopackage */ false);
    initializeFlipper(this, getReactNativeHost().getReactInstanceManager());
  }

  /**
   * Loads Flipper in React Native templates. Call this in the onCreate method with something like
   * initializeFlipper(this, getReactNativeHost().getReactInstanceManager());
   *
   * @param context
   * @param reactInstanceManager
   */
  private static void initializeFlipper(
      Context context, ReactInstanceManager reactInstanceManager) {
    if (BuildConfig.DEBUG) {
      try {
        /*
         We use reflection here to pick up the class that initializes
         Flipper, since Flipper library is not available in release mode
        */
        Class<?> aClass = Class.forName("com.example.ReactNativeFlipper");
        aClass
            .getMethod("initializeFlipper", Context.class, ReactInstanceManager.class)
            .invoke(null, context, reactInstanceManager);
      } catch (ClassNotFoundException e) {
        e.printStackTrace();
      } catch (NoSuchMethodException e) {
        e.printStackTrace();
      } catch (IllegalAccessException e) {
        e.printStackTrace();
      } catch (InvocationTargetException e) {
        e.printStackTrace();
      }
    }
  }
}
```

## Further Steps

To create your own plugins and integrate with Flipper using JavaScript, check out our [writing plugins for React Native](tutorial/react-native) tutorial!