# SHITP — Super Hybrid Information Transfer Protocol

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB)
![Version](https://img.shields.io/badge/version-0.1.1-orange)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

> **Simple structure. Known sizes. Easy parsing.**
> A lightweight data transfer protocol designed from the ground up.

## What is SHITP?

SHITP (Super Hybrid Information Transfer Protocol) is a lightweight data
transfer protocol designed from the ground up. Its goal is to carry data in a
format that is predictable in structure, fast to parse, and easy to implement.

This repository contains the core building blocks of the protocol:

- Core classes for **packet serialization and parsing** (`base.py`, `packet.py`)
- Web-style **request and response builders** (`web/`)
- The protocol's `method`, `mime`, and `status` enums

## What makes it different?

SHITP builds on the experience of established text-based protocols, but sets
its own rules. The standout design decisions:

- **Sizes are always known.** The first 8 bytes of every packet carry the
  header and body sizes explicitly. A parser never has to scan for or guess
  where the payload ends — read the first 8 bytes and you know the entire
  packet.
- **Simple header structure.** Every header is a single-line `name=value`
  pair. No escape sequences, no continuation lines, no multi-stage parsing
  rules.
- **Deterministic packets.** The same input always produces the same output;
  a parser holds no state.
- **Native binary support.** A body given as `str` is encoded as UTF-8;
  given as `bytes`, it is carried as-is.

## Project layout

```
shitp/
├── base.py      # ShitProtocol — create() and decode()
├── packet.py    # ShitpPacket — headers, body, and encode()
├── methods.py   # ShitpMethod enum
├── mimes.py     # MimeType enum
├── status.py    # ShitpStatus enum
└── web/         # Request and response builders
```

## Wire format

Every SHITP packet consists of four parts:

```
+------------------+------------------+------------------+-------------+
|  Header Size     |   Body Size      |     Headers      |    Body     |
|    4 bytes       |    4 bytes       |   variable       |  variable   |
+------------------+------------------+------------------+-------------+
```

- **Header Size** and **Body Size** are unsigned 32-bit integers encoded in
  network byte order (big-endian).
- **Headers** consist of the `name=value` pairs described below.
- **Body** is the payload of the request or response.

## Header format

Every header is a single-line `name=value` pair, UTF-8 encoded and terminated
by `\n`:

```
version=0.1.1
method=FETCH
mime_type=shitp/markdown
host=example.com
custom_header=value
```

The first `=` character separates the name from the value; values may contain
additional `=` characters. The builders set the standard headers
automatically; `set_header()` lets you add as many custom headers as you
like.

## Installation

The package is not published on PyPI yet. Clone the repository and add the
`shitp/` directory to your project — there are no external dependencies.

Requirements:

- Python 3.10+ (type hints use the `str | bytes` syntax)
- Standard library only (`struct`, `enum`)

## Usage

Building a client request and a server response:

```python
from shitp.web.request import ShitpWebRequest
from shitp.web.response import ShitpWebResponse
from shitp.methods import ShitpMethod
from shitp.mimes import MimeType
from shitp.status import ShitpStatus

request = ShitpWebRequest(
    ShitpMethod.FETCH,
    "example.com",
    "# Hello, SHITP!",
).create("0.1.1", MimeType.MARKDOWN)

response = ShitpWebResponse(
    ShitpStatus.OK,
    "# Hello!",
).create("0.1.1", MimeType.MARKDOWN)
```

Both return a `ShitpPacket`. Add custom headers with `set_header()` and
serialize with `encode()`:

```python
request.set_header("user_agent", "shitp-client/0.1.1")

raw_request = request.encode()  # bytes, ready for the wire
```

Decoding a packet — `ShitProtocol.decode()` handles the 8-byte size prefix
automatically:

```python
from shitp.base import ShitProtocol

protocol = ShitProtocol()
packet = protocol.decode(raw_request)

print(packet.get_header("method"))  # 'FETCH'
print(packet.get_header("host"))    # 'example.com'
print(packet.get_body())            # b'# Hello, SHITP!'
```

Building a packet directly, without the web builders:

```python
from shitp.packet import ShitpPacket

packet = ShitpPacket()
packet.set_header("version", "0.1.1")
packet.set_header("mime_type", "shitp/markdown")
packet.set_body("# Hello, SHITP!")

raw = packet.encode()
```

## What's in v0.1.1?

### Methods — `shitp.methods`

| Method  | Description                            |
|---------|----------------------------------------|
| `FETCH` | Retrieves a resource from the server   |
| `PUSH`  | Sends a resource to the server         |

### MIME types — `shitp.mimes`

| MIME type        | Description                       |
|------------------|-----------------------------------|
| `shitp/markdown` | Markdown content (the only type so far) |

### Statuses — `shitp.status`

| Status                  | Description       |
|-------------------------|-------------------|
| `OK`                    | Request succeeded |
| `NOT_FOUND`             | Resource not found|
| `INTERNAL_SERVER_ERROR` | Server-side error |
| `TIMEOUT`               | Timeout           |
| `RESPONSE_ERROR`        | Response error    |

### Classes — `shitp.base` and `shitp.packet`

| Member                                      | Purpose                                                  |
|---------------------------------------------|----------------------------------------------------------|
| `ShitProtocol.create(version, mime_type)`   | Builds a packet with the standard headers                |
| `ShitProtocol.decode(raw_packet)`           | Parses wire bytes (size prefix included) into a packet   |
| `ShitpPacket.set_header(name, value)`       | Adds or replaces a header                                |
| `ShitpPacket.get_header(name)`              | Returns a header value, or `None` if absent              |
| `ShitpPacket.set_body(str \| bytes)`        | Sets the packet body (`str` is UTF-8 encoded)            |
| `ShitpPacket.get_body()`                    | Returns the raw body bytes                               |
| `ShitpPacket.encode()`                      | Serializes the packet into wire bytes                    |

### Builders — `shitp.web`

| Class                                      | Purpose                                     |
|--------------------------------------------|---------------------------------------------|
| `ShitpWebRequest(method, host, body=None)` | Client request with `method`/`host` headers |
| `ShitpWebResponse(status, body=None)`      | Server response with `status` header        |

## Roadmap

- [x] Responsibilities should be separated. The protocol should be redesigned for general-purpose use.
- [x] Refactor for object-oriented programming
- [ ] More methods, MIME types, and status codes
- [ ] Typed data structures for headers
- [ ] Official protocol specification document

The transport layer (TCP sockets) and the server frameworks built on top of
it are out of scope for this repository; they will live in separate repos.

## Contributing

PRs and issues are always welcome. For larger changes, open an issue to
discuss first; for small fixes, feel free to send a PR directly.

---

# SHITP — Super Hybrid Information Transfer Protocol

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB)
![Version](https://img.shields.io/badge/version-0.1.1-orange)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

> **Basit yapı. Bilinen boyutlar. Kolay ayrıştırma.**
> Sıfırdan tasarlanmış, hafif bir veri aktarım protokolü.

## SHITP nedir?

SHITP (Super Hybrid Information Transfer Protocol), sıfırdan tasarlanmış hafif
bir veri aktarım protokolüdür. Amacı, veriyi yapısı öngörülebilir, ayrıştırması
hızlı ve uygulaması kolay bir formatta taşımaktır.

Bu depo, protokolün temel yapı taşlarını içerir:

- **Paket serileştirme ve ayrıştırma** için çekirdek sınıflar (`base.py`, `packet.py`)
- Web tarzı **istek ve yanıt oluşturucuları** (`web/`)
- Protokolün `method`, `mime` ve `status` enum'ları

## Onu farklı kılan nedir?

SHITP, köklü metin tabanlı protokollerin deneyimi üzerine inşa edilir, ancak
kendi kurallarını koyar. Öne çıkan tasarım kararları:

- **Boyutlar her zaman bilinir.** Her paketin ilk 8 baytı, başlık ve gövde
  boyutlarını açıkça taşır. Bir ayrıştırıcının verinin nerede bittiğini
  taraması ya da tahmin etmesi gerekmez — ilk 8 baytı okursanız paketin
  tamamını bilirsiniz.
- **Basit başlık yapısı.** Her başlık tek satırlık bir `name=value` çiftidir.
  Kaçış dizisi yok, satır devamı yok, çok aşamalı ayrıştırma kuralları yok.
- **Deterministik paketler.** Aynı girdi her zaman aynı çıktıyı üretir;
  ayrıştırıcı hiçbir durum tutmaz.
- **Yerel ikili (binary) veri desteği.** `str` olarak verilen gövde UTF-8 ile
  kodlanır; `bytes` olarak verilirse olduğu gibi taşınır.

## Proje yapısı

```
shitp/
├── base.py      # ShitProtocol — create() and decode()
├── packet.py    # ShitpPacket — headers, body, and encode()
├── methods.py   # ShitpMethod enum
├── mimes.py     # MimeType enum
├── status.py    # ShitpStatus enum
└── web/         # Request and response builders
```

## Tel formatı

Her SHITP paketi dört parçadan oluşur:

```
+------------------+------------------+------------------+-------------+
|  Header Size     |   Body Size      |     Headers      |    Body     |
|    4 bytes       |    4 bytes       |   variable       |  variable   |
+------------------+------------------+------------------+-------------+
```

- **Header Size** ve **Body Size**, ağ bayt sırasında (big-endian)
  kodlanmış işaretsiz 32 bitlik tam sayılardır.
- **Headers**, aşağıda açıklanan `name=value` çiftlerinden oluşur.
- **Body**, isteğin veya yanıtın yüküdür.

## Başlık formatı

Her başlık, UTF-8 ile kodlanmış ve `\n` ile sonlandırılmış tek satırlık bir
`name=value` çiftidir:

```
version=0.1.1
method=FETCH
mime_type=shitp/markdown
host=example.com
custom_header=value
```

İlk `=` karakteri adı değerden ayırır; değerler ek `=` karakterleri
içerebilir. Oluşturucular standart başlıkları otomatik olarak ayarlar;
`set_header()` ile istediğiniz kadar özel başlık ekleyebilirsiniz.

## Kurulum

Paket henüz PyPI'da yayınlanmadı. Depoyu klonlayın ve `shitp/` dizinini
projenize ekleyin — harici bağımlılık yoktur.

Gereksinimler:

- Python 3.10+ (tip belirteçleri `str | bytes` sözdizimini kullanır)
- Yalnızca standart kütüphane (`struct`, `enum`)

## Kullanım

Bir istemci isteği ve bir sunucu yanıtı oluşturma:

```python
from shitp.web.request import ShitpWebRequest
from shitp.web.response import ShitpWebResponse
from shitp.methods import ShitpMethod
from shitp.mimes import MimeType
from shitp.status import ShitpStatus

request = ShitpWebRequest(
    ShitpMethod.FETCH,
    "example.com",
    "# Hello, SHITP!",
).create("0.1.1", MimeType.MARKDOWN)

response = ShitpWebResponse(
    ShitpStatus.OK,
    "# Hello!",
).create("0.1.1", MimeType.MARKDOWN)
```

Her ikisi de bir `ShitpPacket` döndürür. `set_header()` ile özel başlıklar
ekleyin ve `encode()` ile serileştirin:

```python
request.set_header("user_agent", "shitp-client/0.1.1")

raw_request = request.encode()  # bytes, ready for the wire
```

Bir paketi çözme — `ShitProtocol.decode()` 8 baytlık boyut önekini otomatik
olarak işler:

```python
from shitp.base import ShitProtocol

protocol = ShitProtocol()
packet = protocol.decode(raw_request)

print(packet.get_header("method"))  # 'FETCH'
print(packet.get_header("host"))    # 'example.com'
print(packet.get_body())            # b'# Hello, SHITP!'
```

Web oluşturucuları olmadan doğrudan paket oluşturma:

```python
from shitp.packet import ShitpPacket

packet = ShitpPacket()
packet.set_header("version", "0.1.1")
packet.set_header("mime_type", "shitp/markdown")
packet.set_body("# Hello, SHITP!")

raw = packet.encode()
```

## v0.1.1'de neler var?

### Metotlar — `shitp.methods`

| Metot   | Açıklama                               |
|---------|----------------------------------------|
| `FETCH` | Sunucudan bir kaynak getirir           |
| `PUSH`  | Sunucuya bir kaynak gönderir           |

### MIME türleri — `shitp.mimes`

| MIME türü        | Açıklama                          |
|------------------|-----------------------------------|
| `shitp/markdown` | Markdown içerik (şimdilik tek tür) |

### Durumlar — `shitp.status`

| Durum                   | Açıklama               |
|-------------------------|------------------------|
| `OK`                    | İstek başarılı         |
| `NOT_FOUND`             | Kaynak bulunamadı      |
| `INTERNAL_SERVER_ERROR` | Sunucu tarafı hatası   |
| `TIMEOUT`               | Zaman aşımı            |
| `RESPONSE_ERROR`        | Yanıt hatası           |

### Sınıflar — `shitp.base` ve `shitp.packet`

| Üye                                         | Amaç                                                     |
|---------------------------------------------|----------------------------------------------------------|
| `ShitProtocol.create(version, mime_type)`   | Standart başlıklarla bir paket oluşturur                 |
| `ShitProtocol.decode(raw_packet)`           | Tel baytlarını (boyut öneki dahil) bir pakete ayrıştırır |
| `ShitpPacket.set_header(name, value)`       | Bir başlık ekler veya değiştirir                         |
| `ShitpPacket.get_header(name)`              | Başlık değerini döndürür; yoksa `None`                   |
| `ShitpPacket.set_body(str \| bytes)`        | Paket gövdesini ayarlar (`str` UTF-8 ile kodlanır)       |
| `ShitpPacket.get_body()`                    | Ham gövde baytlarını döndürür                            |
| `ShitpPacket.encode()`                      | Paketi tel baytlarına serileştirir                       |

### Oluşturucular — `shitp.web`

| Sınıf                                      | Amaç                                              |
|--------------------------------------------|---------------------------------------------------|
| `ShitpWebRequest(method, host, body=None)` | `method`/`host` başlıklı istemci isteği           |
| `ShitpWebResponse(status, body=None)`      | `status` başlıklı sunucu yanıtı                   |

## Yol haritası

- [x] Sorumluluklar ayrılmalı. Protokol genel amaçlı kullanım için yeniden tasarlanmalı.
- [x] Nesne yönelimli programlama için yeniden düzenleme
- [ ] Daha fazla metot, MIME türü ve durum kodu
- [ ] Başlıklar için tipli veri yapıları
- [ ] Resmi protokol şartname belgesi

Taşıma katmanı (TCP soketleri) ve üzerine kurulan sunucu çerçeveleri bu
deponun kapsamı dışındadır; ayrı depolarda yer alacaklar.

## Katkıda bulunma

PR'lar ve issue'lar her zaman memnuniyetle karşılanır. Daha büyük değişiklikler
için önce tartışmak üzere bir issue açın; küçük düzeltmeler için doğrudan PR
gönderebilirsiniz.
