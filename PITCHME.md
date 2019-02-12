---?image=images/logo_flutter_1080px_clr.png&size=auto 70%
# Flutter basics #2

---
## Localizations

+++
## Intl
- reczne generowanie plików .arb na podstawie własnej impementacji pliku bazowego lokalizacji |
- nanoszenie zmian w plikach .arb |
- generowanie kodu dart na podstawie przetłumaczonych plików .arb |

+++
### Przykład implementacji delegata lokalizacji
```dart
static Future<BundledAppLocalizations> load(Locale locale) {
    final String name = _isCountryCodeEmpty(locale) ? locale.languageCode : locale.toString();
    final String localeName = Intl.canonicalizedLocale(name);
    return initializeMessages(localeName).then((_) {
      Intl.defaultLocale = localeName;
      return BundledAppLocalizations();
    });
  }

  static bool _isCountryCodeEmpty(Locale locale) =>
      locale.countryCode == null || locale.countryCode.isEmpty;

  static BundledAppLocalizations of(BuildContext context) {
    return Localizations.of<BundledAppLocalizations>(context, BundledAppLocalizations);
  }

  String get helloWorld {
    return Intl.message(
      'Hello world',
      name: "helloWorld",
    );
  }

  String get homeButton {
    return Intl.message(
      'Home button',
      name: "homeButton",
    );
  }

  String get sectionName {
    return Intl.message(
      'Last section',
      name: "sectionName"
    );
  }
```
@[1-8](Future do zaladowania lokalizacji)
@[17-22](Wiadomość do exportu)

+++
```
flutter packages pub run intl_translation:extract_to_arb --output-dir=lib/locale/intl lib/locale/BundledAppLocalizations.dart

flutter packages pub run intl_translation:generate_from_arb \
    --output-dir=lib/locale/intl --no-use-deferred-loading \
    lib/locale/BundledAppLocalizations.dart lib/locale/intl/intl_en.arb lib/locale/intl/intl_pl.arb
```
@[1](Generowanie głównego pliku .arb)
@[3-5](Generowanie kodu dart na podstawie podanych plików .arb)
