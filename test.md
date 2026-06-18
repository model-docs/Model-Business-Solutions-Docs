# Anadil III Server – Faz 1

Bu sunucu uygulaması, ERP ürünümüzün çeşitli JSON API hizmetleri sağlayabilmesi için gerçekleştirilmiştir.

Anadil III çalışmaları kapsamında gerçekleştirilen bu uygulamada, MS SQL Server temel alınmıştır. Veritabanı bağımsızlığı, ORM kütüphanesi gibi özellikler şimdilik kullanılmamıştır.

## İçindekiler

- [Dağıtım Dizini](#dağıtım-dizini)
  - [Kapsam](#kapsam)
  - [Conf](#conf)
  - [Temp](#temp)
  - [Bin](#bin)
- [Modül](#modül)
    - [Api.xml ve Anadil Dosyaları](#apixml-ve-anadil-dosyaları)
      - [Group ve Include](#group-ve-include)
    - [Struct Tanımları](#struct-tanımları)
    - [Özel Veritabanı](#özel-veritabanı)
- [XML Tanım Dosyaları](#xml-tanım-dosyaları)
  - [Kapsam\app.xml (Güvenlik Tanımları)](#kapsamappxml-güvenlik-tanımları)
  - [Özel DLL Yükleme](#özel-dll-yükleme)
  - [Kapsam\api.xml](#kapsamapixml)
  - [Conf\site.xml](#confsitexml)
    - [Veritabanı](#veritabanı)
    - [Gizlilik](#gizlilik)
    - [Http](#http)
    - [Özel Ayarlar](#özel-ayarlar)
- [JWT Token](#jwt-token)

## Dağıtım Dizini

Anadil.Server.exe ile çalıştırılan sunucu, parametre olarak bir dağıtım (deployment) dizini almaktadır. Alt dizinler şunlardır:

### Kapsam

Bu dizin uygulamayı oluşturan modüller alt dizinler şeklinde içerir. Modüllerden oluşan iş uygulaması bütününü ilgilendiren global API’ler **api.xml**, güvenlik ve benzeri ayarlar da **app.xml** dosyalarında tanımlanırlar.

### Conf

Uygulamanın dağıtıma özel ayarları bu alt dizindeki **site.xml** dosyasında bulunmaktadır.

### Temp

Uygulamanın çalışması sırasında üretilen log dosyaları ve derlenmiş Anadil dosyaları bu dizinde bulunmaktadır.

### Bin

Sunucu uygulamasının EXE ve DLL dosyaları bu dizinde bulunmaktadır. Anadil.Server.exe çalıştırıldığında, eğer başka parametre verilmediyse bir alt dizinini temel alarak çalışacaktır.

## Modül

Bir iş uygulaması modüllerin kompozisyonundan oluşur. Modüller mümkün olduğunca kendi içlerinde bağımsız olacak şekilde tasarlanmalıdır. Faz 1 çalışmasında varolan ERP kullanılacağından bu ideali gerçekleştirmek çok beklenmemelidir.

#### Api.xml ve Anadil Dosyaları

Her modül API arabirimi sunar. Bir örnekle anlatalım:

Bu dosyada modülün dışarıya sunduğu API tanımları bulunmaktadır.

```xml
<api>
  <endpoint path="hesap/ekle" service="hesap_plani" function="Ekle" />
```

Eğer uygulamaya hesap/ekle URL’I kullanılarak bir istek gelirse, bu istek **hesap_plani.anadil** dosyasında bulunan **Ekle()** fonksiyonuna ulaşacaktır. Bu fonksiyon aşağıdaki şekilde tanımlanır.

```anadil
Export Ekle

Function Ekle(input as Hesap\Ekle) as Hesap\EkleRet
```

##### Group ve Include

Endpoint tanımlamalarını hem kolaylaştırmak, hem de daha yapısal hale getirmek için iki tag vardır. Örneğin:

```xml
<group path=”hesap” service=”HesapServisi”>
  <endpoint path=”ekle” function=”Ekle” />
  <endpoint path=”sil” function=”Sil” />
</group>
```

Bu, aşağıdaki tanım ile aynıdır:

```xml
  <endpoint path=”hesap/ekle” service=”HesapServisi” function=”Ekle” />
  <endpoint path=”hesap/sil” service=”HesapServisi” function=”Sil” />
```

Group tanımında path de, service de opsiyoneldir.

Group’lar içiçe tanımlanabilir. İçiçe tanımlarda her içte path tanımları eklene eklene gider.

Include tanımları da group ile aynı mantığa sahiptir. Tek farkı içinde başka dosyanın okunmasıdır. Aynı şekilde path ve service tanımları kullanılabilir. İçiçe tanımlara izin verilir.

```xml
<include file=”hesaplar.xml” path=”hesap” service=”HesapServisi” />
```

#### Struct Tanımları

Fonksiyonun parametresi ve dönüş değerleri Anadil **struct** yapısında olup, modül dizinindeki **hesap.struct.json** dosyasında aşağıdaki gibi tanımlanmıştır:

```
Ekle
{
  ad: string,
  kod: string,
  ust_hesap_kodu: string,
  ust_hesap_rsayac: number
}

EkleRet
{
  r_sayac: number,
  mesaj: string
}
```

Dışarıdan bu API’a ulaşacak istemciler bu JSON yapısında bir veriyi doldurararak gönderir, cevabı da yine bu yapıda alırlar.

#### Özel Veritabanı

Eğer modül, sitede tanımlı olan varsayılan veritabanından başka bir veritabanı kullanacaksa aşağıdaki tanımlanabilir.

```xml
<api database=”personel”>
```

Bu veritabanı  site.xml’deki **database** kısmında **name** attribute ile belirtilerek tanımlanmalıdır.

## XML Tanım Dosyaları

İş uygulaması ve dağıtıma özel tanımlar aşağıdaki XML dosyalarında yapılmaktadır:

### Kapsam\app.xml (Güvenlik Tanımları)

Örnekle anlatalım:

```xml
<app>
  <security module="security" validator="validateToken"
            sessionStruct="security\Session" />
</app>
```

<security> tanımında güvenliğin hangi modül tarafından sağlandığı belirtildikten sonra, JWT token’ın periyodik olarak hangi URL ile belirtilen API tarafından doğrulanacağı **validator** ile belirtilir.

Uygulamaya özel, JWT token içerisinde saklanan **session** bilgisinin struct’ının adı da **sessionStruct** ile belirtilir.

```xml
<endpoint path="validateToken" service="security" function="ValidateToken" />
```

```anadil
Function ValidateToken() as bool
  var session = GetSession()
  If not CheckLogin(session.user, session.password) then
    return false
  EndIf
  return true
EndFunction
```

### Özel DLL Yükleme

App.xml dosyasında uygulamanın Anadil kodları tarafından kullanılacak özel DLL’ler belirtilebilir:

```xml
<app>
  <security module="security" validator="validateToken" sessionStruct="security\Session" />
  <lib>
    <dll>Ornek.dll</dll>
    <dll>Ornekal.dll</dll>
  </lib>
</app>
```

Bu DLL’ler **bin** dizinine koyulmalıdır. Ayrıntılı bilgi için Anadil III Programlama Dili dökümantasyonuna bakınız.

### Kapsam\api.xml

Her modülün kendi api.xml dosyası olduğundan bahsetmiştik. Kapsam altındaki genel api.xml’de ise, modül adından bağımsız olarak URL’ler tanımlanabilir.

```xml
<api>
  <endpoint path="login" module="security" redirect="login" anonymous="true" />
  <endpoint path="logoff" module="security" redirect="logoff" />
  <endpoint path="validateToken" module="security" redirect="validateToken" />
```

Burada URL’in hangi modülde hangi alt URL’e yönlendirileceği **redirect** ile belirlenir.

Bir örnekle açıklığa kavuşturalım: Security modülü altındaki bir API’a ulaşmak için şöyle bir adres girilmelidir:

http://xyz:8080/security/login

Ama kapsam altında tanımlı API’a daha kısa olarak, veya özel bir URL ile ulaşılabilir:

http://xyz:8080/ login

### Conf\site.xml

Bu dosyada dağıtıma/kuruluma özel ayarlar tutulur.

```xml
<site>
  <database dbms="sqlserver" server="modelsavas\savasql"
            name="testdb" user="sa" password="123" />
  <secret key="87C33C1A-340E-49E8-AF11-745C346C8866" />
  <http port=”1905” />
</site>
```

#### Veritabanı

Veritabanında bir SQL Server bağlantısının bilgileri bulunur.

```xml
<database dbms="sqlserver" server="modelsavas\savasql"
            name="testdb" user="sa" password="123" />
```

#### Gizlilik

Secret ile, JWT token’ların şifrelenmesinde kullanılacak, kuruluma özel bir bilgi vardır.

Http ile web sunucusu ayarları belirlenir.

Bu dosya ilk kullanıldıktan şifreli olarak site.dat diye bir dosya oluşturulur. Bundan sonra sunucu verileri o dosyadan okuyacaktır. Bu durumda orijinal site.xml dosyası daha güvenli bir yere alınabilir.

Yine de conf dizininin güvenliği, sadece yetkili kullanıcıların ulaşacağı şekilde ayarlanmalıdır.

#### Http

Sunucuyu koşturan Kestrel web sunucusunun ayarları aşağıdaki gibi tanımlanabilir:

```xml
  <http port="1905">
    <limits MaxRequestBufferSize="0" MaxResponseBufferSize="0" KeepAliveTimeout="0">
    <cors>
      <origins>
        <allow origin="chrome-extension://gmmkjpcadciiokjpikmkkmapphbmdjok" />
      </origins>
    </cors>
  </http>
```

Eğer tüm originlere izin verilmesi isteniyorsa aşağıdaki gibi tanım yapılabilir:

```xml
<origins allow="any" />
```

#### Özel Ayarlar

Site.xml dosyasında iş uygulamasına özel ayarlar tanımlanabilir. Örneğin:

```xml
<site>
  <database dbms="sqlserver" server="modelsavas\savasql"
            name="testdb" user="sa" password="123" />
  <secret key="87C33C1A-340E-49E8-AF11-745C346C8866" />
  <http port=”1905” />
  <vade>6</vade>
</site>
```

Bu ayara GetSiteValue() fonksiyonu ile ulaşılabilir:

```anadil
var vade := GetSiteValue(“vade”, “12”)
```

İkinci parametre, eğer ayar tanımlanmamışsa okunacak default değerdir.

## JWT Token

Güvenlik, API istekleri arasında **http header** olarak taşınan **authorization bearer** ile sağlanır.

Anadil kodundan Token nesnesine herhangi bir anda **GetToken()** fonksiyonu ile ulaşılabilir. **Authenticated** ve **Expired** adlı iki bool özelliği mevcuttur.

Token’ın içinde ayrıca app.xml’deki tanımda **securityStruct** ile örneğini gösterdiğimiz, uygulamaya özel yapıda bir veri de saklanır. Bu veri yapısı uygulamaya özel veriler içerebilir.

**GetSession()** fonksiyonu ile bu özel veriye ulaşılabilir.

Bir örnekle biraz daha anlatalım. Önce API’ımızın login URL’ini inceleyelim:

```xml
<endpoint path="login" service="security" function="DoLogin" anonymous="true" />
```

Burada login’i herkesi çağırabilmesi için **Anonymous=true** belirtildiğini görüyorsunuz. Varsayılan durumda ise, bir API’ı çağırabilmek için bağlananın authenticated olması gerekir. Bunu sağlayan DoLogin fonksiyonuna bakalım:

```anadil
Function DoLogin(input as security\login) as bool
  var token = GetToken()
  If token.Authenticated then
    outm("Already authenticated")
    return false
  EndIf

  If not CheckLogin(input.user, input.password) then
    return false
  EndIf

  token.Authenticated = true
  var session = GetSession()
  session.user = input.user
  session.password = input.password
  return true
EndFunction
```

Burada login isteği doğrulandıktan sonra **token.Authenticated = true** ile artık bundan sonra token sahibinin tanındığı belirtiliyor.

Öte yandan Session nesnesi içine girişte kullanılan kullanıcı ve şifre bilgileri sonradan validation’da kullanılmak üzere saklanıyor:

```anadil
Function ValidateToken() as bool
  var session = GetSession()
  If not CheckLogin(session.user, session.password) then
    return false
  EndIf
  return true
EndFunction
```
