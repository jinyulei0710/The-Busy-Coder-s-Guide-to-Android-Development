###经由‘Flavor’的依赖

相似的，如果你定义了多个`product flavor`，你就可以有雨某个特定‘flavor’绑定的依赖。

例如，假定你为多个可穿戴设备编写了一个应用，并且你设置了三个`product flavor`:

productFlavors{
  standard{
    applicationId ="com.commonsware.android.wearable.qr"
  }
  imwatch{
    applicationId "com.commonsware.android.wearable.qr.imwatch"
  }
  sony{
    applicationId "com.commonsware.android.wearable.qr.sony"
  }
}

包含以上productFlavors配置的android 闭包之后的 dependencies闭包可以有
一个有每种口味的混合以及口味特有的依赖：例如:

dependencies{
  compile 'com.android.support:support-v4:19.0.1'
  compile 'com.google.code.gson:gson:2.2.4'
  compile 'com.squareup.okhttp:okhttp:1.3.0'
  compile 'com.squareup.retorfit:retrofit:1.4.0'
  sonyCompile 'com.sonyericsson.extras.liveware.aef:SmartExtansionUntils:2.1.0'
}

这里，最后一个依赖使用的是sonyCompile,而不是compile,只是这是一个专为sony产品口味使用的依赖。

注意这里为sonyCompile列出了的artifact并不真实存在，至少现在不存在。在到某个时间点SONY开始把
它们放在Maven Central或它们自己的artifact仓库之前，转换SONY的代码成本地的artifact，
然后通过mavenLocal()进行引用是有可能的。
