---
id: sandbox-plugin
title: Sandbox Setup
sidebar_label: Sandbox
---

To use the sandbox plugin, you need to add the plugin to your Flipper client instance. Currently the plugin is only supported on Android.

## Android

```java
import com.facebook.flipper.plugins.sandbox.SandboxFlipperPlugin;
import com.facebook.flipper.plugins.sandbox.SandboxFlipperPluginStrategy;

final SandboxFlipperPluginStrategy strategy = getStrategy(); // Your strategy goes here
client.addPlugin(new SandboxFlipperPlugin(strategy));
```
