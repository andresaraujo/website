---
layout: tutorial
title: "Internationalizing Flutter Apps"

permalink: /tutorials/internationalization/
---

<div class="whats-the-point" markdown="1">

<b> <a id="whats-the-point" class="anchor" href="#whats-the-point" aria-hidden="true"><span class="octicon octicon-link"></span></a>What you'll learn:</b>

* How to track the device's locale (the user's preferred language).
* How to manage locale-specific app values.
* How define the locales an app supports.

</div>

## Internationalizing Flutter Applications

If your app might be deployed to users who speak another language then
you'll need to "internationalize" it. That means you'll need to write
the app in a way that makes it possible to "localize" values like text
and layouts for each language or "locale" that the app
supports. Flutter provides widgets and classes that help with
internationalization and the Flutter libraries themselves are
internationalized.

### Tracking the Locale: The `Locale` class and the `Localizations` Widget

The [`Locale`](https://docs.flutter.io/flutter/dart-ui/Locale-class.html)
class is used to identify the user's language. Mobile devices support
setting the locale for all applications, usually via a system settings
menu. Internationalized apps respond by displaying values that are
locale-specific. For example if the user switches the device's locale
from English to French then a Text widget that displayed "Hello World"
would be rebuilt with "Bonjour le monde".

The
[`Localizations`](https://docs.flutter.io/flutter/widgets/Localizations-class.html)
widget defines the locale for its child and the localized resources
that the child depends on. The
[`WidgetsApp`](https://docs.flutter.io/flutter/widgets/WidgetsApp-class.html)
widget creates a Localizations widget and rebuilds it if the system's locale changes.

You can always lookup an app's current locale with `Localizations.localeOf()`:
```
Locale myLocale = Localizations.localeOf(context);
```

### Loading and Retrieving Localized Values

The `Localizations` widget is used to load and lookup objects that
contain collections of localized values. Apps refer to these objects
with [`Localizations.of(context,type)`](https://docs.flutter.io/flutter/widgets/Localizations/of.html).
If the device's locale changes, the Localizations widget automatically
loads values for the new locale and then rebuilds widgets that used
it. This happens because Localizations works like an
[`InheritedWidget`](https://docs.flutter.io/flutter/widgets/InheritedWidget-class.html). When
a build function refers to an inherited widget an implicit dependency
on the inherited widget is created. When an inherited widget changes
(when the Localizations widget's locale changes), its dependent
contexts are rebuilt.

Localized values are loaded by the Localizations widget's list of
[`LocalizationsDelegate`](https://docs.flutter.io/flutter/widgets/LocalizationsDelegate-class.html)s. Each
delegate must define an async
[`load()`](https://docs.flutter.io/flutter/widgets/LocalizationsDelegate/load.html)
method that produces an object which encapsulates a collection of
localized values. Typically these objects define one method per localized value.

In a large app, different modules or packages might be bundled with
their own localizations. That's why the Localizations widget manages a
table of objects, one per LocalizationsDelegate. To retrieve the
object produced by one of the LocalizationsDelegate's `load` methods,
you specify a BuildContext and the object's type.

For example the localized strings for the Material widgets are defined
by the
[`MaterialLocalizations`](https://docs.flutter.io/flutter/material/MaterialLocalizations-class.html)
class. Instances of this class are created by a LocalizationDelegate
provided by the
[`MaterialApp`](https://docs.flutter.io/flutter/material/MaterialApp-class.html)
 class. They can be retrieved with `Localizations.of`:

```dart
Localizations.of<MaterialLocalizations>(context, MaterialLocalizations);
```

This particular `Localizations.of()` expression is used frequently, so the 
MaterialLocalizations class provides a convenient shortand:

```dart
static MaterialLocalizations of(BuildContext context) {
  return Localizations.of<MaterialLocalizations>(context, MaterialLocalizations);
}

/// References to the localized values defined by MaterialLocalizations
/// are typically written like this:

tooltip: MaterialLocalizations.of(context).backButtonTooltip,
```

### Defining a Class for the App's Localized Resources

Putting all of this together for an internationalized app usually
starts with the class that encapsulates the app's localized
values. The example that follows is typical of such classes. The
`MyLocalizations` class contains the app's strings (just one for the example)
translated into the locales that the app supports. It uses the `initializeMessages()` function
generated by the [Dart `intl` package](https://pub.dartlang.org/packages/intl)
to load the translated strings and
[`Intl.message()`](https://www.dartdocs.org/documentation/intl/0.15.1/intl/Intl/message.html)
to look them up.

```dart
class MyLocalizations {
  MyLocalizations(this.locale);

  final Locale locale;

  static Future<MyLocalizations> load(Locale locale) {
    return initializeMessages(locale.toString())
      .then((Null value) {
        return new MyLocalizations(locale);
      });
  }

  static MyLocalizations of(BuildContext context) {
    return Localizations.of<MyLocalizations>(context, MyLocalizations);
  }

  String get helloWorld => Intl.message('helloWorld', name: 'helloWorld', locale: locale.toString());
  // ... more Intl.message() methods like get helloWorld
}
```

A class based on the `intl` package imports a generated message catalog that provides
the `initializeMessages()` function and the per-locale backing store for `Intl.message()`.
The message catalog is produced by an `intl` tool that analyzes the source code for
classes that contain `Intl.message()` calls. In this case that would just be the
`MyLocalizations` class.

One could choose another approach for loading localized resources and looking them up
while still conforming to the structure of MyLocalizations. The An Alternative Class
for the App's Localized Resources below provides such an example.

An instance of `MyLocalizations` is created by a simple `LocalizationsDelegate`:

```dart
class MyLocalizationsDelegate extends LocalizationsDelegate<MyLocalizations> {
  @override
  Future<MyLocalizations> load(Locale locale) => MyLocalizations.load(locale);

  @override
  bool shouldReload(MyLocalizationsDelegate old) => false;
}
```

An instance of `MyLocalizationsDelegate` is provided to the application's
[`MaterialApp`](https://docs.flutter.io/flutter/material/MaterialApp-class.html)
(or its [`WidgetsApp`](https://docs.flutter.io/flutter/widgets/WidgetsApp-class.html)
for non-material applications):

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      localizationsDelegates: <LocalizationsDelegate>[
        new MyLocalizationsDelegate()
      ],
      supportedLocales: <Locale>[
        const Locale('en', 'US'), // US English
        const Locale('fr', 'CA'), // Candian French
      ],
      child: home: new Scaffold(
        appBar: new AppBar(
          title: new Text(MyLocalizations.of(context).helloWorld),
          ...
        ),
      ),
    );
  }
}
```

As you can see, the app's `AppBar` title comes from an instance of
`MyLocalizations` produced by `MyLocalizations.load()` via
`MyLocalizationsDelegate`. If the device's locale changes, then the
app will be rebuilt with a new `MyLocalizations` object, one
constructed for the new locale. Any reference to
`MyLocalizations.of()` anywhere in the app will causes its context to
be rebuilt if the locale changes.

### Specifying the App's `supportedLocales` Parameter

Although the Flutter's Material library includes support for about 16
languages, only English language translations are available by
default. It's up to applications to decide exactly which languages
they'll support, since it wouldn't make sense for the toolkit
libraries to support a different set of locales than the app does.

The `MaterialApp`
[`supportedLocales`](https://docs.flutter.io/flutter/material/MaterialApp-class.html)
parameter limits locale changes. When the user changes the locale
setting on their device, the app's `Localizations` widget will only
follow suit if the new locale is a member of the this list. If an
exact match for the device locale isn't found, then the first
supported locale with a matching
[`languageCode`](https://docs.flutter.io/flutter/dart-ui/Locale/languageCode.html)
is used. If that fails, then the first element of the
`supportedLocales` list is used.

In terms of the previous `MyApp` example, the app will only accept the
US English or French Canadian locales and it will substitute US
English (the first locale in the list) for anything else.

An app that wants to use a different "locale resolution" method, can
provide a
[`localeResolutionCallback`](https://docs.flutter.io/flutter/material/MaterialApp-class.html). 
For example to have your app unconditionally accept whatever locale the
user selects:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
       localeResolutionCallback(Locale locale, Iterable<Locale> supportedLocales) {
         return locale;
       }
       // ...
    );
  }
}

```


### An Alternative Class for the App's Localized Resources

The previous `MyApp` example was defined in terms of the Dart `intl`
package. Developers can choose their own approach for managing
localized values for the sake of simplicity or perhaps to integrate
with a different i18n framework.

The example that follows uses the same `MyLocalizationsDelegate`
definition as before. However in this case the class that contains the
app's localizations, `MyLocalizations`, is slightly different.

```dart
class MyLocalizations {
  MyLocalizations(Locale locale) {
    _nameToValue = _localeToNameToValue[locale.languageCode];
  }

  Map<String, String> _nameToValue;
  Map<String, Map<String, String>> _localeToNameToValue = {
    'en': <String, String>{               // English
      'helloWorld': 'Hello World',
     },
    'fr': <String, String>{               // French
      'helloWorld': 'Bonjour le monde',
     },
     // ...
  };

  static Future<MyLocalizations> load(Locale locale) {
    return SynchronousFuture(new MyLocalizations(locale));
  }

  static MyLocalizations of(BuildContext context) {
    return Localizations.of<MyLocalizations>(context, MyLocalizations);
  }

  String get helloWorld => _nameToValue['helloWorld'];
  // ... more methods like get helloWorld
}
```

In this case the app's translations are built into the MyLocalizations
class. The constructor initializes `_nameToValue` map based on the
locale's language code.

The static `MyLocalizations.load()` method returns a
[`SynchronousFuture`](https://docs.flutter.io/flutter/foundation/SynchronousFuture-class.html)
because no asynnchronous loading needs to take place.
