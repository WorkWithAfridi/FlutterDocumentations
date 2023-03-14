# ****Localizations with BLoC & Shared pref****

---

# 1. Set-up localization

Open the **`pubspec.yaml`** file and add the following packages:

```dart
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations: # Add this line
    sdk: flutter # Add this line
  intl: ^0.17.0 # Add this line
```

And 

```dart
flutter:
  generate: true # Add this line
```

The next step is to create a new folder inside **`lib`** where our localization files are stored. This folder is usually called **`l10n`** (which stands for localizations) or **`l18n`** (which stands for internationalization). That’s the commonly used name, but you can still use your own folder name.

Inside that folder, create a file with the **`.arb`** extension. In my case, I want to provide two languages to my app, English and Bahasa Indonesia. Then I create two files, **`app_en.arb`** and **`app_id.arb`**. Inside the **`.arb`** file, there are ***key-value pairs*** for the localized texts.

```dart
{
  "onboarding": "The Easy Way\nTo Manage Your\nDaily Expense ✨",
  "getStarted": "Get Started",
  "haveAccount": "I Have Account"
  ...
}
//For EN
```

and

```dart
{
  "onboarding": "Cara Mudah\nAtur Pengeluaran\nHarian ✨",
  "getStarted": "Mulai",
  "haveAccount": "Saya Punya Akun"
  ...
}
//For other lan
```

Next, in the root project directory create a new **`.yaml`**file called **`l10n.yaml`**
 and add the following:

```dart
arb-dir: lib/l10n # 1
template-arb-file: app_en.arb # 2
output-localization-file: app_localizations.dart # 3
```

Here are what the above code does:

1. Specified the localization location in the project. This will point to the folder that was created early, **`lib/l10n`**.
2. Specified the default language for the app when the user installs it for the first time. In this case, the default language is English (**`app_en.arb`**).
3. Specified the name of generated localization file. The commonly used name for that is **`app_localizations.dart`**, but you can use your own name for more convenience.

To start generating the localizations, build and run the app or run the command below in the terminal:

```dart
flutter gen-l10n
```

This will generate the localizations files like so:

```dart
.dart_tool
├── flutter_gen
│   ├── gen_l10n
│   │   ├── app_localizations.dart <--
│   │   ├── app_localizations_en.dart <--
│   │   └── app_localizations_id.dart <--
│   └── pubspec.yaml
...
```

After that, open **`lib/main.dart`** and set the**`supportedLocales`** and **`localizationsDelegates`**
 properties of **`MaterialApp`** with the generated localizations code.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      ...
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      ...
    );
  }
}
```

Finally, the localized texts are ready to use. To access the localized texts, use the following syntax:

```
AppLocalizations.of(context)!.<YOUR_TEXT_KEY>
```

```dart
import 'package:flutter/material.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import '../../../gen/assets.gen.dart';
import '../../components/components.dart';

class OnboardingScreen extends StatelessWidget {
  const OnboardingScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    return Scaffold(
      backgroundColor: Colors.white,
      body: SafeArea(
        child: Column(
          children: [
            Expanded(child: Assets.onboarding.svg()),
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    l10n.onboarding,
                    style: Theme.of(context).textTheme.headline1,
                  ),
                  const SizedBox(height: 32.0),
                  Button.filled(
                    onPressed: () {},
                    label: l10n.getStarted,
                  ),
                  const SizedBox(height: 8.0),
                  Button.outlined(
                    onPressed: () {},
                    label: l10n.haveAccount,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

# 2. Setup - BLoC

To make things simple, we can use an ***enhanced enum***

```dart
import 'dart:ui';

import '../../../../gen/assets.gen.dart';

enum Language {
  english(
    Locale('en', 'US'),
  ),
  indonesia(
    Locale('id', 'ID'),
  );
}
```

Next, create a new **bloc** to handle the app language change.

> BLoC State
> 

```dart
part of 'language_bloc.dart';

class LanguageState extends Equatable {
  const LanguageState({
    Language? selectedLanguage,
  }) : selectedLanguage = selectedLanguage ?? Language.english;

  final Language selectedLanguage;

  @override
  List<Object> get props => [selectedLanguage];

  LanguageState copyWith({Language? selectedLanguage}) {
    return LanguageState(
      selectedLanguage: selectedLanguage ?? this.selectedLanguage,
    );
  }
}
```

> BLoC Event
> 

```dart
part of 'language_bloc.dart';

abstract class LanguageEvent extends Equatable {
  const LanguageEvent();

  @override
  List<Object> get props => [];
}

class ChangeLanguage extends LanguageEvent {
  const ChangeLanguage({required this.selectedLanguage});
  final Language selectedLanguage;

  @override
  List<Object> get props => [selectedLanguage];
}
```

> BLoC
> 

```dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';

import '../models/language.dart';

part 'language_event.dart';
part 'language_state.dart';

class LanguageBloc extends Bloc<LanguageEvent, LanguageState> {
  LanguageBloc() : super(const LanguageState()) {
    on<ChangeLanguage>(onChangeLanguage);
  }

  onChangeLanguage(ChangeLanguage event, Emitter<LanguageState> emit) {
    emit(state.copyWith(selectedLanguage: event.selectedLanguage));
  }
}
```

Now, you can use that bloc in your app.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => LanguageBloc(),
      child: BlocBuilder<LanguageBloc, LanguageState>(
        builder: (context, state) {
          return MaterialApp.router(
            ...
            locale: state.selectedLanguage.value, // Change language based on the state
            supportedLocales: AppLocalizations.supportedLocales,
            localizationsDelegates: AppLocalizations.localizationsDelegates,
            ...
          );
        },
      ),
    );
  }
}
```

Here, we wrap the **`MaterialApp`**with **`BlocBuilder`**to rebuild the widget tree when the state changes. Also, don’t forget to register the bloc with **`BlocProvider`**. Then, set the **`locale`**
 property with the state value.

Change states using the following syntax

```dart
context.read<LanguageBloc>().add(ChangeLanguage(selectedLanguage: Language.values[index]));
```

---

# 2. Setup - Shared Pref.

Open the **`pubspec.yaml`**file and add the package.

```dart
shared_preferences: ^2.0.17
```

Next, we need to modify our **bloc**. First, add a new **bloc event** to retrieve the cached language preference:

> BLoC Event
> 

```dart
part of 'language_bloc.dart';

abstract class LanguageEvent extends Equatable {
  const LanguageEvent();

  @override
  List<Object> get props => [];
}

class ChangeLanguage extends LanguageEvent {
  const ChangeLanguage({required this.selectedLanguage});
  final Language selectedLanguage;

  @override
  List<Object> get props => [selectedLanguage];
}

class GetLanguage extends LanguageEvent {} // Add this
```

After that, update the LanguageBloc with the following code:

```dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../models/language.dart';

part 'language_event.dart';
part 'language_state.dart';

const languagePrefsKey = 'languagePrefs';

class LanguageBloc extends Bloc<LanguageEvent, LanguageState> {
  LanguageBloc() : super(const LanguageState()) {
    on<ChangeLanguage>(onChangeLanguage);
    on<GetLanguage>(onGetLanguage);
  }

  onChangeLanguage(ChangeLanguage event, Emitter<LanguageState> emit) async {
    // # 1
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(
      languagePrefsKey,
      event.selectedLanguage.value.languageCode,
    );
    emit(state.copyWith(selectedLanguage: event.selectedLanguage));
  }
  
  // # 2
  onGetLanguage(GetLanguage event, Emitter<LanguageState> emit) async {
    final prefs = await SharedPreferences.getInstance();
    final selectedLanguage = prefs.getString(languagePrefsKey);
    emit(state.copyWith(
      selectedLanguage: selectedLanguage != null
          ? Language.values
              .where((item) => item.value.languageCode == selectedLanguage)
              .first
          : Language.english,
    ));
  }
}
```

Here: 

1. When the user changes their preferred language, store it in the shared preference, then emit the state changes.
2. When the user opens the app, loads the preferred language from the shared preference, then emits the state changes. If the user opens the app for the first time, which means the shared preferences are currently ***null***, then provide the default language.

Lastly, update the **`BlocProvider`**inside **`main.dart`** with the following: