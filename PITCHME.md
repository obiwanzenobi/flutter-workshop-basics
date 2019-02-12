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

```dart
return MaterialApp(
      title: 'Flutter Demo',
      localizationsDelegates: [
        BundledAppLocalizationsDelegate(),
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
      ],
      supportedLocales: [const Locale("en"), const Locale("pl")],
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: ListPage(BlocBinder.of(context).listBloc),
    );
 ```
@[4](Delegat lokalizacji w widgecie MaterialApp)
@[8](Dodanie wspieranych języków)

+++
```
flutter packages pub run intl_translation:extract_to_arb --output-dir=lib/locale/intl lib/locale/BundledAppLocalizations.dart

flutter packages pub run intl_translation:generate_from_arb \
    --output-dir=lib/locale/intl --no-use-deferred-loading \
    lib/locale/BundledAppLocalizations.dart lib/locale/intl/intl_en.arb lib/locale/intl/intl_pl.arb
```
@[1](Generowanie głównego pliku .arb)
@[3-5](Generowanie kodu dart na podstawie podanych plików .arb)

+++
## i18n IDE Plugin
- Automatyczna generacja kodu dart z plików .arb znajdujących sie w /res/values/ |
- Generowanie listy wspieranych języków na podstawie dostępnych zasobów |
- Wymaga jedynie zarejstrowania wygenerowanego delegata w MaterialApp |
- Tylko dla InteliJ IDEA oraz Android Studio |
- https://github.com/long1eu/flutter_i18n |
---
## Pierwsze użycie build runnera - built_values
- Generuje kod value modeli |
- Gwarantuje immutability |
- Tworzy za nas funkcje equals(==) oraz hashcode |
- https://github.com/google/built_value.dart |

+++
## LiveTemplate dla built_values

```
import 'package:built_value/built_value.dart';

part '$FILE_NAME$.g.dart';

abstract class $CLASS_NAME$ implements Built<$CLASS_NAME$, $CLASS_NAME$Builder> {
  $CLASS_NAME$._();
  factory $CLASS_NAME$([updates($CLASS_NAME$Builder b)]) = _$$$CLASS_NAME$;
}
```
+++
## Komendy do uruchomienia build runnera
```
flutter packages pub run build_runner build

flutter packages pub run build_runner watch
```
@[1](Pojedyńcze uruchomienie build runnera)
@[3](Watch nasłuchujący na zmiany (zapis) w plikach oznaczonych do generowania)

---
### Zapytania REST oraz (de)serializacja danych

+++
## Wygodny stack do obsługi zapytań sieciowych: Jaguar Retrofit
W jego skład wchodzi:
- Jaguar serializer - narzedzie build runnera do generowania serializatorów (nie współdziała z build_value) |
- Jaguar resty - biblioteka znacznie upraszczająca komunikacji http, dodaje obsługę interceptorów oraz autoryzacji |
- Jaguar retrofit - generowanie funkcji zapytań sieciowych na konkretne endpointy na podstawie adnotacji w klasie baozwej |
- https://github.com/Jaguar-dart/client |

+++
### Przykład użycia - serializer
```dart
import 'package:jaguar_serializer/jaguar_serializer.dart';

part 'user_response.jser.dart';

class UserResponse {
  final String login;

  UserResponse(this.login);
}

@GenSerializer()
class UserResponseSerializer extends Serializer<UserResponse> with _$UserResponseSerializer {}
```
Deklarujemy model oraz serializer oznaczając go odpowiednią adnotacją
+++
### Jaguar serializer live template - jser
```
import 'package:jaguar_serializer/jaguar_serializer.dart';

part '$FILE_NAME$.jser.dart';

class $CLASS_NAME$ {
}

@GenSerializer()
class $CLASS_NAME$Serializer extends Serializer<$CLASS_NAME$> with _$$$CLASS_NAME$Serializer {}
```
+++
### Resty
```dart
typedef FutureOr<dynamic> After(StringResponse response);

typedef FutureOr<void> Before(RouteBase route);

Route(apiUrl)
      .after(injector.getDependency<ResponseLoggingInterceptor>())
      .after(injector.getDependency<Authenticator>())
      .before(injector.getDependency<RequestLoggingInterceptor>())
      .before(injector.getDependency<AuthInterceptor>())
      .withClient(injector.getDependency<BaseClient>()))     
```
@[1](Definicja interceptorów after)
@[3](Definicja interceptorów before)
@[5-10](Budowanie base route)
+++
### Definicja interfejsu api
```dart
import 'package:flutter_bloc_test/infrastructure/users/responses/user_response.dart';
import 'package:jaguar_retrofit/jaguar_retrofit.dart';

part 'users_api.jretro.dart';

@GenApiClient(path: "users")
class UsersApi extends ApiClient with _$UsersApiClient {
  final Route base;

  UsersApi(this.base);

  @GetReq()
  Future<List<UserResponse>> users();
}
```
