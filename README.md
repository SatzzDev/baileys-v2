# <div align='center'>Baileys - WhatsApp API</div>

<div align='center'>

![WhatsApp API](https://i.supa.codes/kyWCSZ)

</div>

> [!TIP]  
> Store ini sangat berguna untuk keperluan seperti:
> - Menyimpan polling
> - Retry pesan
> - Melacak status kontak dan grup
> - Menyediakan command `.listchat`, `.listgroup`, dll. dengan data real-time

Jika kamu menggunakan custom `getMessage()`, store ini juga dapat dijadikan referensi lokal untuk mendekripsi polling dan mengirim ulang pesan.

## Penjelasan Tentang WhatsApp ID

- `id` atau biasa disebut juga `jid` adalah **identitas WhatsApp** dari seseorang atau grup yang menjadi tujuan pengiriman pesan.  
- Format ID harus sesuai dengan jenis akun tujuan:

### Jenis Format ID WhatsApp

#### 1. Pengguna Pribadi (User)
**Format:**
```
[kode negara][nomor telepon]@s.whatsapp.net
```
**Contoh:**
```
628123456789@s.whatsapp.net
```

#### 2. Grup WhatsApp
**Format:**
```
[timestamp grup dibuat]-[random id]@g.us
```
**Contoh:**
```
1234567890-987654321@g.us
```

#### 3. Broadcast (Daftar Siaran)
**Format:**
```
[timestamp]@broadcast
```
**Contoh:**
```
1685539347@broadcast
```

#### 4. Status (Story)
**Format:**
```
status@broadcast
```

#### 5. Newsletter (Channel WhatsApp)
**Format:**
```
[numeric id]@newsletter
```
**Contoh:**
```
120363025487665599@newsletter
```

> **TIP:**  
> Kamu bisa mendapatkan `jid` dari:
> - `m.key.remoteJid`
> - `groupParticipantsUpdate`
> - `messages.upsert`, dll

> **CAUTION:**  
> Jangan pernah mengubah format `jid` secara manual tanpa validasi.  
> Salah format bisa menyebabkan error `bad jid` atau pesan tidak terkirim.

## Fungsi Utilitas (Utility Functions)

Baileys menyediakan beberapa fungsi utilitas penting yang sangat membantu saat mengembangkan bot:

- **`getContentType(message)`**  
  Mengembalikan jenis konten dari pesan (misalnya: `imageMessage`, `conversation`, `buttonsMessage`, dll).

- **`getDevice(jid)`**  
  Mengembalikan jenis perangkat yang digunakan pengirim (jika tersedia), contoh: Android, iPhone, Web.

- **`makeCacheableSignalKeyStore(authState)`**  
  Membungkus SignalKeyStore menjadi versi yang lebih efisien dan bisa di-cache, untuk performa autentikasi yang lebih cepat.

- **`downloadContentFromMessage(message, type)`**  
  Mengunduh media dari pesan (seperti gambar, video, dokumen).  
  `type` bisa berupa `'image'`, `'video'`, `'audio'`, `'document'`, dll.

  Contoh penggunaan:
  ```javascript
  const stream = await downloadContentFromMessage(msg.imageMessage, 'image')
  const buffer = Buffer.concat([])
  for await (const chunk of stream) buffer.push(chunk)
  ```

> [!NOTE]  
> Sebagian besar fungsi utilitas tidak dipanggil otomatis — Kamu harus menggunakannya sesuai kebutuhan, terutama saat menangani pesan media, format m.chat, atau decrypt konten.

## Mengirim Pesan

- Semua jenis pesan dapat dikirim menggunakan **satu fungsi saja**, yaitu `sendMessage()`.  
- Lihat daftar jenis pesan yang didukung [di sini](https://baileys.whiskeysockets.io/types/AnyMessageContent.html)  
- Dan semua opsi pengiriman pesan [di sini](https://baileys.whiskeysockets.io/types/MiscMessageGenerationOptions.html)

Contoh:

```javascript
const jid = '628XXXXXXXXX@s.whatsapp.net' // tujuan
const content = { text: 'Halo, ini pesan dari bot!' } // isi pesan
const options = { quoted: null } // opsi tambahan (misalnya: balasan)

await miya.sendMessage(m.chat, content, options)
```

### Pesan Non-Media

#### Pesan Teks
```javascript
await miya.sendMessage(m.chat, { text: 'Halo dunia' })
```

#### Pesan Balasan (Quote)
```javascript
await miya.sendMessage(m.chat, { text: 'Ini balasan pesan kamu' }, { quoted: message })
```

#### Mention Pengguna (Tag)
Gunakan `@nomor` dalam teks dan sertakan `mentions` di payload.
```javascript
await miya.sendMessage(
  m.chat,
  {
    text: '@628XXXXXXXXX Hai Satzz!',
    mentions: ['628XXXXXXXXX@s.whatsapp.net']
  }
)
```

#### Meneruskan Pesan (Forward)
Butuh objek pesan (`WAMessage`). Bisa didapat dari store atau pesan sebelumnya.
```javascript
const msg = getMessageFromStore() // Kamu buat sendiri sesuai struktur
await miya.sendMessage(m.chat, { forward: msg, force: true })
```

### Pesan Interaktif

#### Tombol Teks (Buttons)
```javascript
await miya.sendMessage(m.chat, {
  text: 'Pilih salah satu:',
  buttons: [
    { buttonId: 'btn_1', buttonText: { displayText: 'Tombol 1' }, type: 1 },
    { buttonId: 'btn_2', buttonText: { displayText: 'Tombol 2' }, type: 1 }
  ],
  footer: 'Contoh footer'
})
```

#### Daftar (List Message)
```javascript
await miya.sendMessage(m.chat, {
  text: 'Pilih dari daftar berikut:',
  footer: 'Contoh footer',
  title: 'Judul Daftar',
  buttonText: 'Buka List',
  sections: [
    {
      title: 'Menu 1',
      rows: [
        { title: 'Opsi A', rowId: 'pilih_a' },
        { title: 'Opsi B', rowId: 'pilih_b' }
      ]
    },
    {
      title: 'Menu 2',
      rows: [
        { title: 'Opsi C', rowId: 'pilih_c' }
      ]
    }
  ]
})
```

> [!TIP]  
> Kamu bisa menggabungkan semua jenis pesan dengan opsi tambahan seperti `quoted`, `mentions`, `ephemeralExpiration`, dan lainnya untuk membuat interaksi bot yang lebih kaya dan interaktif.

#### Lokasi Biasa
```javascript
await miya.sendMessage(
  m.chat,
  {
    location: {
      degreesLatitude: -6.200000,
      degreesLongitude: 106.816666
    }
  }
)
```

#### Lokasi Langsung (Live Location)
```javascript
await miya.sendMessage(
  m.chat,
  {
    location: {
      degreesLatitude: -6.200000,
      degreesLongitude: 106.816666
    },
    live: true
  }
)
```

#### Kirim Kontak (vCard)
```javascript
const vcard =
  'BEGIN:VCARD\n' +
  'VERSION:3.0\n' +
  'FN:Satzz Izumi\n' +
  'ORG:ZERO DEV;\n' +
  'TEL;type=CELL;type=VOICE;waid=628XXXXXXXXX:+62 831-4366-3697\n' +
  'END:VCARD'

await miya.sendMessage(
  m.chat,
  {
    contacts: {
      displayName: 'Satzz Izumi',
      contacts: [{ vcard }]
    }
  }
)
```

#### Pesan Reaksi (Reaction Message)

- Kamu perlu mengirimkan `key` dari pesan yang ingin diberikan reaksi.  
  `key` bisa diambil dari [store](#mengimplementasikan-data-store) atau menggunakan [WAMessageKey](https://baileys.whiskeysockets.io/types/WAMessageKey.html).

```javascript
await miya.sendMessage(
  m.chat,
  {
    react: {
      text: '🔥', // gunakan string kosong '' untuk menghapus reaksi
      key: message.key
    }
  }
)
```

#### Pin Pesan (Pin Message)

- Kamu juga perlu memberikan `key` dari pesan yang ingin dipin.  
  Kamu dapat mengatur durasi pin berdasarkan waktu dalam detik.

| Durasi | Detik        |
|--------|--------------|
| 24 jam | 86.400       |
| 7 hari | 604.800      |
| 30 hari| 2.592.000    |

```javascript
await miya.sendMessage(
  m.chat,
  {
    pin: {
      type: 1, // 1 untuk pin, 2 untuk unpin
      time: 86400,
      key: message.key
    }
  }
)
```

### Menandai Pesan (Keep Message)

- Untuk menyimpan pesan tertentu agar tidak terhapus otomatis.

```javascript
await miya.sendMessage(
  m.chat,
  {
    keep: {
      key: message.key,
      type: 1 // 1 = simpan, 2 = batalkan simpan
    }
  }
)
```

#### Pesan Polling (Poll Message)

- Kirim polling ke grup atau kontak pribadi. Dapat menentukan apakah polling bersifat publik (announcement group).

```javascript
await miya.sendMessage(
  m.chat,
  {
    poll: {
      name: 'Polling Hari Ini',
      values: ['Opsi A', 'Opsi B', 'Opsi C'],
      selectableCount: 1,
      toAnnouncementGroup: false
    }
  }
)
```

#### Pesan Hasil Polling (Poll Result)

- Kirim hasil polling secara manual jika dibutuhkan. Cocok untuk sistem polling terintegrasi.

```javascript
await miya.sendMessage(
  m.chat,
  {
    pollResult: {
      name: 'Hasil Polling',
      values: [
        ['Opsi A', 120],
        ['Opsi B', 350],
        ['Opsi C', 75]
      ]
    }
  }
)
```

### Pesan Panggilan (Call Message)

- Digunakan untuk mengirim notifikasi panggilan, bisa suara atau video.

```javascript
await miya.sendMessage(
  m.chat,
  {
    call: {
      name: 'Hay',
      type: 1 // 1 = suara, 2 = video
    }
  }
)
```

### Pesan Event (Event Message)

- Cocok untuk mengumumkan acara atau undangan dengan detail lokasi dan waktu.

```javascript
await miya.sendMessage(
  m.chat,
  {
    event: {
      isCanceled: false, // true jika dibatalkan
      name: 'Liburan Bareng!',
      description: 'Siapa yang mau ikut?', 
      location: {
        degreesLatitude: 24.121231,
        degreesLongitude: 55.1121221,
        name: 'Pantai Sanur'
      },
      startTime: 1715000000, 
      endTime: 1715086400, 
      extraGuestsAllowed: true // apakah boleh bawa tamu
    }
  }
)
```

### Pesan Pemesanan (Order Message)

- Digunakan untuk menampilkan detail pemesanan dari katalog bisnis WhatsApp.

```javascript
await miya.sendMessage(
  m.chat,
  {
    order: {
      orderId: '574XXX',
      thumbnail: 'your_thumbnail', 
      itemCount: 3,
      status: 'INQUIRY', // atau ACCEPTED / DECLINED
      surface: 'CATALOG',
      message: 'Deskripsi pesanan',
      orderTitle: 'Judul Pesanan',
      sellerJid: '628xxx@s.whatsapp.net',
      token: 'your_token',
      totalAmount1000: '150000',
      totalCurrencyCode: 'IDR'
    }
  }
)
```

### Pesan Produk (Product Message)

- Menampilkan detail produk dari katalog bisnis.

```javascript
await miya.sendMessage(
  m.chat,
  {
    product: {
      productImage: { 
        url: 'https://your-image.url/image.jpg'
      },
      productId: 'PRD-001', 
      title: 'Produk Spesial',
      description: 'Deskripsi lengkap produk kamu di sini', 
      currencyCode: 'IDR', 
      priceAmount1000: '50000', 
      retailerId: 'store-izumi', // opsional
      url: 'https://linkproduk.com', // opsional
      productImageCount: 1, 
      firstImageId: 'img-001', // opsional
      salePriceAmount1000: '45000', 
      signedUrl: 'https://your.signed.url' // opsional
    },
    businessOwnerJid: '628xxx@s.whatsapp.net'
  }
)
```

### Pesan Pembayaran (Payment Message)

- Digunakan untuk mengirimkan informasi pembayaran, cocok untuk chatbot belanja.

```javascript
await miya.sendMessage(
  m.chat,
  {
    payment: {
      note: 'Hi!',
      currency: 'IDR',
      offset: 0,
      amount: '10000',
      expiry: 0,
      from: '628xxxx@s.whatsapp.net',
      image: {
        placeholderArgb: '#222222', 
        textArgb: '#FFFFFF',  
        subtextArgb: '#AAAAAA'
      }
    }
  }
)
```

#### Pesan Undangan Pembayaran (Payment Invite Message)

- Digunakan untuk mengundang pengguna lain melakukan pembayaran.

```javascript
await miya.sendMessage(
  m.chat, 
  { 
    paymentInvite: {
      type: 1, // 1 = request, 2 = accept, 3 = decline (sesuaikan sesuai konteks)
      expiry: 0
    }
  }
)
```

### Pesan Undangan Admin Channel (Admin Invite Message)

- Meminta pengguna untuk menjadi admin di saluran (newsletter) kamu.

```javascript
await miya.sendMessage(
  m.chat,
  {
    adminInvite: {
      jid: '123xxx@newsletter',
      name: 'Channel Satzz',
      caption: 'Tolong jadi admin channel saya ya!',
      expiration: 86400, // dalam detik (24 jam)
      jpegThumbnail: Buffer // opsional, bisa berupa buffer gambar
    }
  }
)
```

### Undangan Grup WhatsApp (Group Invite Message)

- Mengirim undangan ke grup tertentu menggunakan kode undangan.

```javascript
await miya.sendMessage(
  m.chat,
  {
    groupInvite: {
      jid: '123xxx@g.us',
      name: 'Grup Dev Satzz',
      caption: 'Ayo gabung ke grup WhatsApp kami!',
      code: 'ABCD1234', // kode undangan grup
      expiration: 86400,
      jpegThumbnail: Buffer // opsional
    }
  }
)
```

### Pesan Bagikan Nomor Telepon (Share Phone Number)

- Mengirim permintaan eksplisit untuk membagikan nomor telepon pengguna.

```javascript
await miya.sendMessage(
  m.chat,
  {
    sharePhoneNumber: {}
  }
)
```

### Pesan Permintaan Nomor Telepon (Request Phone Number)

- Meminta pengguna untuk membagikan nomor telepon mereka secara langsung.

```javascript
await miya.sendMessage(
  m.chat,
  {
    requestPhoneNumber: {}
  }
)
```

### Pesan Balasan Tombol (Button Reply Message)

- Digunakan untuk merespons interaksi tombol yang diklik pengguna. Tipe pesan dibedakan berdasarkan jenis tombol yang digunakan.

#### Tombol Tipe List
```javascript
await miya.sendMessage(
  m.chat,
  {
    buttonReply: {
      name: 'Hai', 
      description: 'Deskripsi pilihan', 
      rowId: 'pilihan_1'
    }, 
    type: 'list'
  }
)
```

#### Tombol Tipe Plain
```javascript
await miya.sendMessage(
  m.chat,
  {
    buttonReply: {
      displayText: 'Halo', 
      id: 'plain_id'
    }, 
    type: 'plain'
  }
)
```

#### Tombol Tipe Template
```javascript
await miya.sendMessage(
  m.chat,
  {
    buttonReply: {
      displayText: 'Pilih Saya', 
      id: 'template_id', 
      index: 1
    }, 
    type: 'template'
  }
)
```

#### Tombol Tipe Interactive (Native Flow)
```javascript
await miya.sendMessage(
  m.chat,
  {
    buttonReply: {
      body: 'Mau pilih yang mana?', 
      nativeFlows: {
        name: 'menu_options', 
        paramsJson: JSON.stringify({ id: 'menu_1', description: 'Deskripsi interaktif' }),
        version: 1 // bisa juga 2 atau 3
      }
    }, 
    type: 'interactive'
  }
)
```

### Pesan dengan Tombol (Buttons Message)

- Pesan biasa yang disertai hingga **3 tombol** untuk respon cepat.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Ini adalah pesan tombol!',
    caption: 'Gunakan jika memakai gambar/video',
    footer: 'Salam dari Satzz Izumi!',
    buttons: [
      { 
        buttonId: 'btn1', 
        buttonText: { displayText: 'Tombol 1' }
      },
      { 
        buttonId: 'btn2', 
        buttonText: { displayText: 'Tombol 2' }
      },
      { 
        buttonId: 'btn3', 
        buttonText: { displayText: 'Tombol 3' }
      }
    ]
  }
)
```

### Pesan List Tombol (Buttons List Message)

- Hanya bisa digunakan di **chat pribadi**, bukan grup.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Ini adalah daftar pilihan!',
    footer: 'Dipersembahkan oleh Satzz Izumi',
    title: 'Judul Daftar Pilihan',
    buttonText: 'Klik untuk melihat opsi',
    sections: [
      {
        title: 'Bagian 1',
        rows: [
          { title: 'Opsi 1', rowId: 'opsi1' },
          { title: 'Opsi 2', rowId: 'opsi2', description: 'Deskripsi Opsi 2' }
        ]
      },
      {
        title: 'Bagian 2',
        rows: [
          { title: 'Opsi 3', rowId: 'opsi3' },
          { title: 'Opsi 4', rowId: 'opsi4', description: 'Deskripsi Opsi 4' }
        ]
      }
    ]
  }
)
```

### Pesan Daftar Produk dengan Tombol (Buttons Product List Message)

- Hanya dapat digunakan di **chat pribadi**, bukan grup.  
- Menampilkan daftar produk dari katalog bisnis WhatsApp kamu.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Ini adalah daftar produk!',
    footer: 'Dikirim oleh Satzz Izumi',
    title: 'Pilih Produk Unggulan',
    buttonText: 'Lihat Daftar Produk',
    productList: [
      {
        title: 'Kategori Produk Utama',
        products: [
          { productId: '1234' },
          { productId: '5678' }
        ]
      }
    ],
    businessOwnerJid: '628xxx@s.whatsapp.net',
    thumbnail: 'https://example.jpg' // atau buffer gambar
  }
)
```

### Pesan Kartu dengan Tombol (Buttons Cards Message)

- Menampilkan beberapa kartu (card) interaktif dengan gambar atau video + tombol.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Isi Utama Pesan',
    title: 'Judul Utama',
    subtile: 'Subjudul Opsional',
    footer: 'Footer Pesan',

    cards: [
      {
        image: { url: 'https://example.jpg' }, // bisa juga Buffer
        title: 'Judul Kartu',
        body: 'Isi Konten Kartu',
        footer: 'Footer Kartu',
        buttons: [
          {
            name: 'quick_reply',
            buttonParamsJson: JSON.stringify({
              display_text: 'Tombol Cepat',
              id: 'ID_TOMBOL_1'
            })
          },
          {
            name: 'cta_url',
            buttonParamsJson: JSON.stringify({
              display_text: 'Kunjungi Website',
              url: 'https://www.example.com'
            })
          }
        ]
      },
      {
        video: { url: 'https://example.mp4' }, // bisa juga Buffer video
        title: 'Judul Kartu Video',
        body: 'Deskripsi Konten',
        footer: 'Footer Kartu',
        buttons: [
          {
            name: 'quick_reply',
            buttonParamsJson: JSON.stringify({
              display_text: 'Respon Cepat',
              id: 'ID_TOMBOL_2'
            })
          },
          {
            name: 'cta_url',
            buttonParamsJson: JSON.stringify({
              display_text: 'Lihat Selengkapnya',
              url: 'https://www.example.com'
            })
          }
        ]
      }
    ]
  }
)
```

### Pesan Tombol Template (Buttons Template Message)

- Menampilkan tombol dengan tipe URL, panggilan, atau tombol balasan cepat.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Ini adalah pesan template tombol!',
    footer: 'Dikirim oleh Satzz Izumi',
    templateButtons: [
      {
        index: 1,
        urlButton: {
          displayText: 'Ikuti Channel',
          url: 'https://whatsapp.com/channel/0029Vag9VSI2ZjCocqa2lB1y'
        }
      },
      {
        index: 2,
        callButton: {
          displayText: 'Hubungi Saya!',
          phoneNumber: '628xxxx'
        }
      },
      {
        index: 3,
        quickReplyButton: {
          displayText: 'Balas Cepat',
          id: 'id-button-reply'
        }
      }
    ]
  }
)
```

### Pesan Tombol Interaktif (Interactive Buttons)

- Mendukung berbagai jenis tombol dan dapat digunakan dengan media.

```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Ini pesan interaktif!',
    title: 'Hai!',
    subtitle: 'Subjudul di sini',
    footer: 'Dikirim oleh Satzz Izumi',
    interactiveButtons: [
      {
        name: 'quick_reply',
        buttonParamsJson: JSON.stringify({
          display_text: 'Klik Aku!',
          id: 'id_kamu'
        })
      },
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Kunjungi Channel',
          url: 'https://whatsapp.com/channel/0029Vag9VSI2ZjCocqa2lB1y',
          merchant_url: 'https://whatsapp.com/channel/0029Vag9VSI2ZjCocqa2lB1y'
        })
      },
      {
        name: 'cta_copy',
        buttonParamsJson: JSON.stringify({
          display_text: 'Salin Link',
          copy_code: 'https://whatsapp.com/channel/0029Vag9VSI2ZjCocqa2lB1y'
        })
      },
      {
        name: 'cta_call',
        buttonParamsJson: JSON.stringify({
          display_text: 'Telepon Saya',
          phone_number: '628xxxx'
        })
      },
      {
        name: 'single_select',
        buttonParamsJson: JSON.stringify({
          title: 'Pilih Opsi',
          sections: [
            {
              title: 'Pilihan Utama',
              highlight_label: 'Rekomendasi',
              rows: [
                {
                  header: 'Header 1',
                  title: 'Opsi 1',
                  description: 'Deskripsi 1',
                  id: 'id1'
                },
                {
                  header: 'Header 2',
                  title: 'Opsi 2',
                  description: 'Deskripsi 2',
                  id: 'id2'
                }
              ]
            }
          ]
        })
      }
    ]
  }
)
```

#### Versi dengan Media

##### Gambar
```javascript
await miya.sendMessage(
  m.chat,
  {
    image: { url: 'https://example.jpg' },
    caption: 'Isi Pesan',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Footer',
    interactiveButtons: [ /* tombol seperti di atas */ ],
    hasMediaAttachment: false
  }
)
```

##### Video
```javascript
await miya.sendMessage(
  m.chat,
  {
    video: { url: 'https://example.mp4' },
    caption: 'Isi Video',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Footer',
    interactiveButtons: [ /* tombol seperti di atas */ ],
    hasMediaAttachment: false
  }
)
```

##### Dokumen
```javascript
await miya.sendMessage(
  m.chat,
  {
    document: { url: 'https://example.jpg' },
    mimetype: 'image/jpeg',
    jpegThumbnail: await miya.resize('https://example.jpg', 320, 320),
    caption: 'Isi Dokumen',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Footer',
    interactiveButtons: [ /* tombol seperti di atas */ ],
    hasMediaAttachment: false
  }
)
```

##### Lokasi
```javascript
await miya.sendMessage(
  m.chat,
  {
    location: {
      degreesLatitude: -6.2,
      degreesLongitude: 106.8,
      name: 'Satzz HQ'
    },
    caption: 'Ayo ke sini!',
    title: 'Lokasi Tujuan',
    subtitle: 'Subjudul Lokasi',
    footer: 'Peta lokasi',
    interactiveButtons: [ /* tombol seperti di atas */ ],
    hasMediaAttachment: false
  }
)
```

##### Produk
```javascript
await miya.sendMessage(
  m.chat,
  {
    product: {
      productImage: { url: 'https://example.jpg' },
      productId: '836xxx',
      title: 'Produk Pilihan',
      description: 'Deskripsi produk terbaik',
      currencyCode: 'IDR',
      priceAmount1000: '283000',
      retailerId: 'SatzzStore',
      url: 'https://example.com',
      productImageCount: 1
    },
    businessOwnerJid: '628xxx@s.whatsapp.net',
    caption: 'Produk baru tersedia!',
    title: 'Nama Produk',
    subtitle: 'Subjudul Produk',
    footer: 'Info Produk',
    interactiveButtons: [ /* tombol seperti di atas */ ],
    hasMediaAttachment: false
  }
)
```

### Mention Status (Status Mentions Message)

- Digunakan untuk membuat status WhatsApp yang menyebut seseorang secara langsung.

```javascript
await miya.sendStatusMentions(
  m.chat, 
  {
    image: {
      url: 'https://example.com.jpg'
    }, 
    caption: 'Halo dari Satzz!'
  }
)
```

### Pesan Album (Send Album Message)

- Mengirim beberapa gambar atau video sebagai album (sekuens media). Bisa pakai `Buffer` atau URL.

```javascript
await miya.sendAlbumMessage(
  m.chat,
  [
    {
      image: { url: 'https://example.jpg' }, 
      caption: 'Gambar 1'
    },
    {
      image: Buffer, 
      caption: 'Gambar 2'
    },
    {
      video: { url: 'https://example.mp4' }, 
      caption: 'Video 1'
    }, 
    {
      video: Buffer, 
      caption: 'Video 2'
    }
  ],
  { 
    quoted: message, // opsional, untuk membalas pesan
    delay: 2000 // jeda antar media (ms)
  }
)
```

### Pesan Toko (Shop Message)

- Digunakan untuk mengarahkan pengguna ke katalog atau produk dalam fitur bisnis WhatsApp.

#### Teks Saja
```javascript
await miya.sendMessage(
  m.chat, 
  {      
    text: 'Body pesan',
    title: 'Judul Toko', 
    subtitle: 'Subjudul', 
    footer: 'Powered by Satzz',
    shop: {
      surface: 1,
      id: 'https://example.com'
    }, 
    viewOnce: true
  }
)
```

#### Gambar
```javascript
await miya.sendMessage(
  m.chat, 
  { 
    image: { url: 'https://example.jpg' },
    caption: 'Deskripsi produk',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Footer',
    shop: {
      surface: 1,
      id: 'https://example.com'
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Video
```javascript
await miya.sendMessage(
  m.chat, 
  { 
    video: { url: 'https://example.mp4' },
    caption: 'Tonton videonya!',
    title: 'Judul Video',
    subtitle: 'Subjudul',
    footer: 'Footer',
    shop: {
      surface: 1,
      id: 'https://example.com'
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Dokumen
```javascript
await miya.sendMessage(
  m.chat, 
  {
    document: { url: 'https://example.jpg' },
    mimetype: 'image/jpeg',
    jpegThumbnail: await miya.resize('https://example.jpg', 320, 320),
    caption: 'Lampiran dokumen',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Footer',
    shop: {
      surface: 1,
      id: 'https://example.com'
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Lokasi
```javascript
await miya.sendMessage(
  m.chat, 
  { 
    location: {
      degreesLatitude: -6.2000, 
      degreesLongitude: 106.8166,
      name: 'Lokasi Toko'
    },
    caption: 'Lihat lokasi kami!',
    title: 'Judul Lokasi',
    subtitle: 'Subjudul',
    footer: 'Peta lokasi',
    shop: {
      surface: 1,
      id: 'https://example.com'
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Produk
```javascript
await miya.sendMessage(
  m.chat,
  {
    product: {
      productImage: { url: 'https://example.jpg' },
      productId: '836xxx',
      title: 'Nama Produk',
      description: 'Deskripsi produk menarik',
      currencyCode: 'IDR',
      priceAmount1000: '283000',
      retailerId: 'SatzzStore',
      url: 'https://example.com',
      productImageCount: 1
    },
    businessOwnerJid: '628xxx@s.whatsapp.net',
    caption: 'Lihat produk unggulan kami!',
    title: 'Judul Produk',
    subtitle: 'Subjudul Produk',
    footer: 'Info produk',
    shop: {
      surface: 1,
      id: 'https://example.com'
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

### Pesan Koleksi (Collection Message)

- Fitur ini digunakan untuk menampilkan koleksi katalog dari bisnis tertentu di WhatsApp.

#### Teks Saja
```javascript
await miya.sendMessage(
  m.chat, 
  {
    text: 'Isi pesan',
    title: 'Judul Koleksi',
    subtitle: 'Subjudul',
    footer: 'Dari Satzz Izumi',
    collection: {
      bizJid: '628xxx@s.whatsapp.net', 
      id: 'https://example.com', 
      version: 1
    },
    viewOnce: true
  }
)
```

#### Gambar
```javascript
await miya.sendMessage(
  m.chat, 
  { 
    image: { url: 'https://example.jpg' },
    caption: 'Koleksi Gambar',
    title: 'Judul Koleksi',
    subtitle: 'Subjudul',
    footer: 'Katalog Satzz',
    collection: {
      bizJid: '628xxx@s.whatsapp.net', 
      id: 'https://example.com',
      version: 1
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Video
```javascript
await miya.sendMessage(
  m.chat, 
  {
    video: { url: 'https://example.mp4' },
    caption: 'Koleksi Video',
    title: 'Judul Video',
    subtitle: 'Subjudul',
    footer: 'Video Katalog',
    collection: {
      bizJid: '628xxx@s.whatsapp.net', 
      id: 'https://example.com',
      version: 1
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Dokumen
```javascript
await miya.sendMessage(
  m.chat, 
  {
    document: { url: 'https://example.jpg' },
    mimetype: 'image/jpeg',
    jpegThumbnail: await miya.resize('https://example.jpg', 320, 320),
    caption: 'Dokumen Katalog',
    title: 'Judul Dokumen',
    subtitle: 'Subjudul',
    footer: 'Lampiran Koleksi',
    collection: {
      bizJid: '628xxx@s.whatsapp.net',
      id: 'https://example.com',
      version: 1
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Lokasi
```javascript
await miya.sendMessage(
  m.chat, 
  {
    location: {
      degreesLatitude: -6.2, 
      degreesLongitude: 106.8,
      name: 'Lokasi Bisnis'
    },
    caption: 'Lihat lokasi koleksi',
    title: 'Judul Lokasi',
    subtitle: 'Subjudul',
    footer: 'Lokasi Katalog',
    collection: {
      bizJid: '628xxx@s.whatsapp.net',
      id: 'https://example.com',
      version: 1
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

#### Produk
```javascript
await miya.sendMessage(
  m.chat,
  {
    product: {
      productImage: { url: 'https://example.jpg' },
      productId: '836xxx',
      title: 'Nama Produk',
      description: 'Deskripsi produk',
      currencyCode: 'IDR',
      priceAmount1000: '283000',
      retailerId: 'SatzzStore',
      url: 'https://example.com',
      productImageCount: 1
    },
    businessOwnerJid: '628xxx@s.whatsapp.net',
    caption: 'Koleksi Produk',
    title: 'Judul',
    subtitle: 'Subjudul',
    footer: 'Produk Katalog',
    collection: {
      bizJid: '628xxx@s.whatsapp.net',
      id: 'https://example.com',
      version: 1
    },
    hasMediaAttachment: false,
    viewOnce: true
  }
)
```

### Mengirim Pesan dengan Pratinjau Link (Link Preview)

1. Secara default, WhatsApp Web tidak menampilkan pratinjau link.
2. Namun, Baileys menyediakan fungsi pembangkit preview link otomatis.
3. Untuk mengaktifkannya, install dulu dependensinya dengan:  
   ```bash
   yarn add link-preview-js
   ```
4. Contoh kirim pesan dengan pratinjau link:
```javascript
await miya.sendMessage(
  m.chat,
  {
    text: 'Hai! Ini dikirim dari https://github.com/whiskeysockets/baileys'
  }
)
```

### Pesan Media (Media Messages)

Mengirim media (gambar, video, audio, stiker) jauh lebih efisien dengan Baileys.

> [!NOTE]  
> Kamu bisa menggunakan `Buffer`, `{ stream }`, atau `{ url }`.  
> Lihat lebih lengkap di [dokumentasi media](https://baileys.whiskeysockets.io/types/WAMediaUpload.html)

> [!TIP]  
> Gunakan **stream** atau **url langsung** agar lebih hemat memori.

#### Pesan GIF (video pendek)

> WhatsApp tidak mendukung file `.gif`, maka harus dikirim dalam bentuk `.mp4` dengan flag `gifPlayback: true`

```javascript
await miya.sendMessage(
  m.chat,
  {
    video: fs.readFileSync('Media/ma_gif.mp4'),
    caption: 'Halo dari GIF!',
    gifPlayback: true
  }
)
```

#### Pesan Video
```javascript
await miya.sendMessage(
  m.chat,
  {
    video: { url: './Media/ma_video.mp4' },
    caption: 'Ini videonya'
  }
)
```

#### Pesan Video PTV (Picture to Video / video bulat WA)

```javascript
await miya.sendMessage(
  m.chat,
  {
    video: { url: './Media/ma_video.mp4' },
    ptv: true
  }
)
```

#### Pesan Audio

> Agar audio kompatibel di semua perangkat, sebaiknya gunakan `ffmpeg` dengan pengaturan berikut:

```bash
ffmpeg -i input.mp4 -avoid_negative_ts make_zero -ac 1 output.ogg
```

```javascript
await miya.sendMessage(
  m.chat,
  {
    audio: { url: './Media/audio.ogg' },
    mimetype: 'audio/ogg; codecs=opus'
  }
)
```

#### Pesan Gambar

```javascript
await miya.sendMessage(
  m.chat,
  {
    image: { url: './Media/ma_img.png' },
    caption: 'Halo dari gambar!'
  }
)
```

#### Pesan View Once

> Fitur **View Once** memungkinkan media hanya bisa dilihat satu kali.

```javascript
await miya.sendMessage(
  m.chat,
  {
    image: { url: './Media/ma_img.png' },
    viewOnce: true,
    caption: 'Media hanya bisa dilihat sekali'
  }
)
```

## Memodifikasi Pesan

### Menghapus Pesan (Untuk Semua Orang)

- Digunakan untuk menarik pesan yang sudah dikirim (delete for everyone).

```javascript
const msg = await miya.sendMessage(m.chat, { text: 'Halo dunia' })
await miya.sendMessage(m.chat, { delete: msg.key })
```

> **Catatan:**  
> Untuk menghapus pesan **hanya untuk diri sendiri**, gunakan `chatModify` (lihat bagian [Modifikasi Chat](#modifying-chats)).

### Mengedit Pesan

- Kamu dapat mengedit isi pesan yang telah dikirim sebelumnya, selama masih berada dalam konteks yang diizinkan oleh WhatsApp.

```javascript
await miya.sendMessage(m.chat, {
  text: 'Teks yang sudah diperbarui di sini',
  edit: response.key
})
```

## Memanipulasi Pesan Media

### Menambahkan Thumbnail pada Media

- Thumbnail (gambar pratinjau) untuk **gambar** dan **stiker** bisa dihasilkan secara otomatis jika kamu menambahkan salah satu dari dependency berikut:

```bash
yarn add jimp
# atau
yarn add sharp
```

- Untuk **video**, kamu juga bisa menghasilkan thumbnail otomatis, tapi pastikan kamu sudah install `ffmpeg` di sistem kamu.

> Contoh penggunaan otomatis biasanya tidak perlu kamu atur manual — Baileys akan meng-generate thumbnail bila dependensi sudah tersedia.

### Mengunduh Media dari Pesan (Downloading Media Messages)

Jika kamu ingin menyimpan media yang diterima dari pengguna:

```javascript
import { createWriteStream } from 'fs'
import { downloadMediaMessage, getContentType } from 'Satzzizumi'

miya.ev.on('messages.upsert', async ({ messages }) => {
let m = messages[0]
if (!m.message) return // jika tidak ada media atau isi pesan

let messageType = getContentType(m.message) // deteksi tipe pesan (image, video, audio, dll)

if (messageType === 'imageMessage') {
let stream = await downloadMediaMessage(
m,
'stream', // bisa juga 'buffer' kalau ingin langsung di-handle tanpa file
{},
{
logger,
reuploadRequest: miya.updateMediaMessage // agar bisa reupload jika file sudah tidak ada
}
)

let file = createWriteStream('./downloaded-image.jpeg')
stream.pipe(file)
}
})
```

### Re-upload Media ke WhatsApp

Jika media sudah dihapus dari server WhatsApp, kamu bisa minta perangkat pengirim untuk melakukan *reupload*:

```javascript
await miya.updateMediaMessage(msg)
```

> Fitur ini penting saat media gagal diunduh karena sudah tidak tersedia di server WhatsApp.

## Menolak Panggilan (Reject Call)

- Kamu bisa mendapatkan `callId` dan `callFrom` dari event `call`.

```javascript
await miya.rejectCall(callId, callFrom)
```

## Mengirim Status ke Chat (Send States in Chat)

### Menandai Pesan Dibaca (Reading Messages)

- Kamu harus menandai pesan satu per satu menggunakan key dari `WAMessage`.
- Tidak bisa menandai seluruh chat sebagai terbaca secara langsung seperti di WhatsApp Web.

```javascript
const key = {
remoteJid: '628xxx@s.whatsapp.net',
fromMe: false,
id: 'ABCDEF123456'
}

// bisa juga array untuk banyak pesan sekaligus
await miya.readMessages([key])
```

> Kamu bisa mendapatkan `messageID` dari:
```javascript
let messageID = message.key.id
```

### Memperbarui Status Kehadiran (Update Presence)

- Status `presence` bisa berupa:  
  `available`, `unavailable`, `composing`, `recording`, `paused`, dll.  
  [Lihat daftar lengkapnya di sini](https://baileys.whiskeysockets.io/types/WAPresence.html)

```javascript
await miya.sendPresenceUpdate('available', jid) // online
await miya.sendPresenceUpdate('composing', jid) // mengetik
await miya.sendPresenceUpdate('unavailable', jid) // offline
```

> **Catatan:**  
> Jika kamu menggunakan WhatsApp Desktop secara bersamaan, maka WA tidak akan mengirim notifikasi ke perangkat lain.  
> Kalau kamu ingin tetap terima notifikasi di HP, kamu bisa set status bot jadi offline:
```javascript
await miya.sendPresenceUpdate('unavailable')
```

## Memodifikasi Chat (Modifying Chats)

WhatsApp menggunakan komunikasi terenkripsi untuk memperbarui status chat atau aplikasi. Beberapa fitur modifikasi sudah didukung oleh Baileys, dan bisa kamu kirim seperti di bawah ini.

> **PERINGATAN:**  
> Jika kamu salah menggunakan modifikasi ini (misal kirim data invalid), WhatsApp bisa **logout semua perangkat** dan kamu harus scan ulang QR.

### Mengarsipkan Chat (Archive)

```javascript
let lastMsgInChat = await getLastMessageInChat(jid) // kamu buat fungsi ini sendiri
await miya.chatModify({ archive: true, lastMessages: [lastMsgInChat] }, jid)
```

### Membisukan / Mengaktifkan Notifikasi (Mute / Unmute)

| Durasi    | Milidetik       |
|-----------|------------------|
| Hapus     | `null`           |
| 8 Jam     | `86400000`       |
| 7 Hari    | `604800000`      |

```javascript
await miya.chatModify({ mute: 8 * 60 * 60 * 1000 }, jid) // bisukan 8 jam
await miya.chatModify({ mute: null }, jid) // aktifkan kembali notifikasi
```

### Tandai Sebagai Terbaca / Belum Dibaca

```javascript
let lastMsgInChat = await getLastMessageInChat(jid)
await miya.chatModify({ markRead: false, lastMessages: [lastMsgInChat] }, jid)
```

### Hapus Pesan Hanya untuk Saya

```javascript
await miya.chatModify(
  {
    clear: {
      messages: [
        {
          id: 'ATWYHDNNWU81732J',
          fromMe: true,
          timestamp: '1654823909'
        }
      ]
    }
  },
  jid
)
```

### Hapus Chat Secara Keseluruhan

```javascript
let lastMsgInChat = await getLastMessageInChat(jid)
await miya.chatModify({
  delete: true,
  lastMessages: [
    {
      key: lastMsgInChat.key,
      messageTimestamp: lastMsgInChat.messageTimestamp
    }
  ]
}, jid)
```

### Pin / Unpin Chat

```javascript
await miya.chatModify({
  pin: true // false untuk unpin
}, jid)
```

### Tandai / Hapus Bintang dari Pesan

```javascript
await miya.chatModify({
  star: {
    messages: [
      {
        id: 'messageID',
        fromMe: true
      }
    ],
    star: true // true: beri bintang, false: hapus bintang
  }
}, jid)
```

### Pesan Menghilang Otomatis (Disappearing Messages)

| Durasi    | Detik (Seconds) |
|-----------|------------------|
| Nonaktif  | `0`              |
| 24 Jam    | `86400`          |
| 7 Hari    | `604800`         |
| 90 Hari   | `7776000`        |

#### Aktifkan

```javascript
await miya.sendMessage(m.chat, {
  disappearingMessagesInChat: 604800 // 7 hari
})
```

#### Kirim Pesan dengan Mode Menghilang

```javascript
await miya.sendMessage(
  m.chat,
  { text: 'halo' },
  { ephemeralExpiration: 604800 }
)
```

#### Nonaktifkan

```javascript
await miya.sendMessage(m.chat, {
  disappearingMessagesInChat: false
})
```

### Menghapus Pesan Tertentu (Clear Messages)
```javascript
await miya.clearMessage(m.chat, key, timestamps)
```

## Query Pengguna (User Queries)

### Cek Apakah Nomor Terdaftar di WhatsApp
```javascript
let [result] = await miya.onWhatsApp(jid)
if (result.exists) console.log(`${jid} terdaftar di WhatsApp sebagai ${result.jid}`)
```

### Ambil Riwayat Chat (termasuk grup)

> Kamu perlu mengambil pesan paling lama dari chat tersebut

```javascript
let msg = await getOldestMessageInChat(jid)
await miya.fetchMessageHistory(
  50, // maksimal 50 per query
  msg.key,
  msg.messageTimestamp
)
```

- Hasilnya akan dikirimkan melalui event `messaging.history-set`

### Ambil Status WhatsApp (Bio)

```javascript
let status = await miya.fetchStatus(jid)
console.log('Status: ' + status)
```

### Ambil Foto Profil (Profil, Grup, Channel)

```javascript
let ppUrl = await miya.profilePictureUrl(jid)
console.log('Foto profil: ' + ppUrl)
```

### Ambil Profil Bisnis (Business Profile)

> Cocok untuk akun bisnis WhatsApp, seperti deskripsi & kategori bisnis

```javascript
let profile = await miya.getBusinessProfile(jid)
console.log('Deskripsi bisnis: ' + profile.description + ', Kategori: ' + profile.category)
```

### Cek Kehadiran Seseorang (Presence: Online / Typing)

```javascript
miya.ev.on('presence.update', console.log)
await miya.presenceSubscribe(jid)
```

## Ubah Profil

### Ubah Status Profil (Bio)

```javascript
await miya.updateProfileStatus('Halo Dunia!')
```

### Ubah Nama Profil

```javascript
await miya.updateProfileName('Satzz Izumi')
```

### Ubah Foto Profil (termasuk grup)

> Sama seperti pesan media, kamu bisa pakai:  
> `{ url }`, `Buffer`, atau `{ stream }`

```javascript
await miya.updateProfilePicture(m.chat, { url: './foto-baru.jpeg' })
```

### Hapus Foto Profil (termasuk grup)

```javascript
await miya.removeProfilePicture(jid)
```

## Grup WhatsApp (Groups)

> Untuk mengubah pengaturan grup, kamu harus menjadi admin grup tersebut.

### Membuat Grup
```javascript
let group = await miya.groupCreate('Grup Hebat Satzz', ['1234@s.whatsapp.net', '4564@s.whatsapp.net'])
console.log('Grup berhasil dibuat dengan ID: ' + group.gid)

await miya.sendMessage(group.id, { text: 'Halo semuanya!' })
```

### Tambah / Hapus / Jadikan Admin / Turunkan Admin

```javascript
await miya.groupParticipantsUpdate(
  m.chat,
  ['abcd@s.whatsapp.net', 'efgh@s.whatsapp.net'],
  'add' // bisa diganti: 'remove', 'promote', 'demote'
)
```

### Ubah Nama Grup

```javascript
await miya.groupUpdateSubject(m.chat, 'Nama Baru Grup!')
```

### Ubah Deskripsi Grup

```javascript
await miya.groupUpdateDescription(m.chat, 'Deskripsi baru untuk grup ini')
```

### Ubah Pengaturan Grup

```javascript
// hanya admin yang bisa kirim pesan
await miya.groupSettingUpdate(m.chat, 'announcement')

// semua anggota bisa kirim pesan
await miya.groupSettingUpdate(m.chat, 'not_announcement')

// semua anggota bisa ubah info grup (foto, nama, dll.)
await miya.groupSettingUpdate(m.chat, 'unlocked')

// hanya admin yang bisa ubah info grup
await miya.groupSettingUpdate(m.chat, 'locked')
```

### Keluar dari Grup

```javascript
await miya.groupLeave(jid)
```

### Dapatkan Kode Undangan Grup

```javascript
let code = await miya.groupInviteCode(jid)
console.log('Kode undangan grup: ' + code)
// gabung pakai: https://chat.whatsapp.com/ + code
```

### Reset / Ganti Kode Undangan Grup

```javascript
let newCode = await miya.groupRevokeInvite(jid)
console.log('Kode undangan baru: ' + newCode)
```

### Gabung Grup dengan Kode Undangan

```javascript
let response = await miya.groupAcceptInvite('ABC123DEF456')
console.log('Berhasil gabung ke grup: ' + response)
```

### Lihat Info Grup dari Kode Undangan

```javascript
let response = await miya.groupGetInviteInfo('ABC123DEF456')
console.log('Info grup: ', response)
```

### Lihat Metadata Grup (peserta, nama, deskripsi, dll.)

```javascript
let metadata = await miya.groupMetadata(jid)
console.log(metadata.id + ', Nama: ' + metadata.subject + ', Deskripsi: ' + metadata.desc)
```

### Gabung Grup dari `groupInviteMessage`

```javascript
let response = await miya.groupAcceptInviteV4(m.chat, groupInviteMessage)
console.log('Gabung ke grup: ' + response)
```

### Lihat Daftar Pengguna yang Minta Gabung

```javascript
let response = await miya.groupRequestParticipantsList(jid)
console.log(response)
```

### Setujui / Tolak Permintaan Gabung

```javascript
let response = await miya.groupRequestParticipantsUpdate(
  m.chat,
  ['abcd@s.whatsapp.net', 'efgh@s.whatsapp.net'],
  'approve' // atau 'reject'
)
console.log(response)
```

### Dapatkan Metadata Semua Grup yang Kamu Ikuti

```javascript
let allGroups = await miya.groupFetchAllParticipating()
console.log(allGroups)
```

### Aktifkan Pesan Sementara di Grup (Ephemeral Message)

| Durasi    | Detik (Seconds) |
|-----------|------------------|
| Nonaktif  | 0                |
| 24 Jam    | 86400            |
| 7 Hari    | 604800           |
| 90 Hari   | 7776000          |

```javascript
await miya.groupToggleEphemeral(m.chat, 86400) // contoh: aktif 1 hari
```

### Ubah Mode Penambahan Anggota Grup

```javascript
await miya.groupMemberAddMode(
  m.chat,
  'all_member_add' // atau 'admin_add'
)
```

## Privasi (Privacy)

### Blokir / Buka Blokir Pengguna

```javascript
await miya.updateBlockStatus(m.chat, 'block') // Blokir pengguna
await miya.updateBlockStatus(m.chat, 'unblock') // Buka blokir pengguna
```

### Ambil Pengaturan Privasi

```javascript
let privacySettings = await miya.fetchPrivacySettings(true)
console.log('Pengaturan privasi:', privacySettings)
```

### Lihat Daftar Blokir

```javascript
let blocklist = await miya.fetchBlocklist()
console.log(blocklist)
```

### Ubah Privasi Terakhir Dilihat (Last Seen)

```javascript
let value = 'all' // bisa juga: 'contacts', 'contact_blacklist', 'none'
await miya.updateLastSeenPrivacy(value)
```

### Ubah Privasi Status Online

```javascript
let value = 'all' // atau 'match_last_seen'
await miya.updateOnlinePrivacy(value)
```

### Ubah Privasi Foto Profil

```javascript
let value = 'all' // bisa juga: 'contacts', 'contact_blacklist', 'none'
await miya.updateProfilePicturePrivacy(value)
```

### Ubah Privasi Status WhatsApp

```javascript
let value = 'all' // bisa juga: 'contacts', 'contact_blacklist', 'none'
await miya.updateStatusPrivacy(value)
```

### Ubah Privasi Centang Biru (Read Receipts)

```javascript
let value = 'all' // atau 'none'
await miya.updateReadReceiptsPrivacy(value)
```

### Ubah Privasi Siapa yang Bisa Menambahkan ke Grup

```javascript
let value = 'all' // bisa juga: 'contacts', 'contact_blacklist'
await miya.updateGroupsAddPrivacy(value)
```

### Ubah Mode Default Pesan Sementara

Durasi dalam detik:

| Durasi    | Detik (Seconds) |
|-----------|------------------|
| Nonaktif  | 0                |
| 24 Jam    | 86400            |
| 7 Hari    | 604800           |
| 90 Hari   | 7776000          |

```javascript
let ephemeral = 86400
await miya.updateDefaultDisappearingMode(ephemeral)
```

### NEWSLETTER

- **Mendapatkan informasi newsletter**
```javascript
const metadata = await miya.newsletterMetadata("invite", "xxxxx")
// atau
const metadata = await miya.newsletterMetadata("jid", "abcd@newsletter")
console.log(metadata)
```

- **Mengubah deskripsi newsletter**
```javascript
await miya.newsletterUpdateDescription("abcd@newsletter", "Deskripsi Baru")
```

- **Mengubah nama newsletter**
```javascript
await miya.newsletterUpdateName("abcd@newsletter", "Nama Baru")
```

- **Mengubah foto profil newsletter**
```javascript
await miya.newsletterUpdatePicture("abcd@newsletter", buffer)
```

- **Menghapus foto profil newsletter**
```javascript
await miya.newsletterRemovePicture("abcd@newsletter")
```

- **Mematikan notifikasi newsletter**
```javascript
await miya.newsletterMute("abcd@newsletter")
```

- **Mengaktifkan kembali notifikasi newsletter**
```javascript
await miya.newsletterUnmute("abcd@newsletter")
```

- **Membuat newsletter baru**
```javascript
const metadata = await miya.newsletterCreate("Nama Newsletter", "Deskripsi Newsletter")
console.log(metadata)
```

- **Menghapus newsletter**
```javascript
await miya.newsletterDelete("abcd@newsletter")
```

- **Mengikuti newsletter**
```javascript
await miya.newsletterFollow("abcd@newsletter")
```

- **Berhenti mengikuti newsletter**
```javascript
await miya.newsletterUnfollow("abcd@newsletter")
```

- **Mengirim reaksi ke pesan di newsletter**
```javascript
const id = "175"
await miya.newsletterReactMessage("abcd@newsletter", id, "🥳")
```

### Ikon AI

```javascript
// cukup tambahkan "ai: true" pada sendMessage
await miya.sendMessage(id, { text: "Hello World", ai: true })
```

## Broadcast & Status WhatsApp

### Kirim Broadcast dan Status (Stories)

- Kamu bisa kirim pesan ke broadcast & story WhatsApp menggunakan `sendMessage()` seperti biasa, tapi dengan tambahan properti khusus:

```javascript
await miya.sendMessage(
  m.chat,
  {
    image: {
      url: url
    },
    caption: 'Halo dari broadcast!'
  },
  {
    backgroundColor: '#ffffff', // opsional
    font: 'default', // opsional
    statusJidList: ['628xxx@s.whatsapp.net'], // daftar kontak yang akan terima status
    broadcast: true // aktifkan mode broadcast
  }
)
```

- Konten pesan bisa berupa `extendedTextMessage`, `imageMessage`, `videoMessage`, atau `voiceMessage`.  
  [Lihat semua tipe konten pesan di sini](https://baileys.whiskeysockets.io/types/AnyRegularMessageContent.html)

- Kamu juga bisa menggunakan `backgroundColor`, `font`, dan pengaturan lainnya pada opsi pengiriman.  
  [Lihat semua opsi di sini](https://baileys.whiskeysockets.io/types/MiscMessageGenerationOptions.html)

- ID broadcast biasanya berbentuk: `12345678@broadcast`

### Ambil Info Daftar Broadcast

```javascript
let bList = await miya.getBroadcastListInfo('1234@broadcast')
console.log(`Nama list: ${bList.name}, Penerima: ${bList.recipients}`)
```

## Menulis Fungsionalitas Kustom (Custom Functionality)

Baileys dirancang untuk **ekstensi & kustomisasi**. Kamu tidak perlu fork repo untuk modifikasi — cukup tulis kode kamu sendiri dan panggil lewat API yang disediakan.

### Mengaktifkan Log Debug WhatsApp

- Untuk melihat semua pesan mentah dari WhatsApp, aktifkan logger debug saat inisialisasi soket:

```javascript
import P from 'pino'

const sock = makeWASocket({
  logger: P({ level: 'debug' })
})
```

> Ini sangat berguna kalau kamu ingin memahami **bagaimana WhatsApp bekerja di balik layar** atau mau buat fitur-fitur advance yang gak didokumentasikan.

## Bagaimana WhatsApp Berkomunikasi Dengan Kita

> **TIP:**  
> Kalau kamu ingin mempelajari protokol komunikasi WhatsApp, disarankan untuk memahami tentang **LibSignal Protocol** dan **Noise Protocol**.

### Contoh Kasus

Misalnya, kamu ingin melacak **persentase baterai** dari HP yang terhubung.  
Kalau kamu mengaktifkan log `debug`, maka akan muncul pesan seperti ini di terminal:

```
{
    "level": 10,
    "fromMe": false,
    "frame": {
        "tag": "ib",
        "attrs": {
            "from": "@s.whatsapp.net"
        },
        "content": [
            {
                "tag": "edge_routing",
                "attrs": {},
                "content": [
                    {
                        "tag": "routing_info",
                        "attrs": {},
                        "content": {
                            "type": "Buffer",
                            "data": [8,2,8,5]
                        }
                    }
                ]
            }
        ]
    },
    "msg": "communication"
}
```

### Penjelasan Struktur `frame`

Setiap pesan dari WhatsApp memiliki struktur `frame` dengan komponen utama berikut:

- `tag` — menandakan tipe pesan (contoh: `'message'`)
- `attrs` — objek berisi key-value untuk metadata (biasanya mengandung ID pesan)
- `content` — data utama dari isi pesan (contoh: isi teks dari pesan)

> Untuk dokumentasi lebih lanjut, lihat [struktur WABinary](/src/WABinary/readme.md)

### Daftarkan Callback Untuk Event WebSocket

> **TIP:**  
> Lihat fungsi `onMessageReceived` di file `socket.ts` untuk memahami cara event websocket diproses.

```javascript
// untuk semua pesan dengan tag 'edge_routing'
miya.ws.on('CB:edge_routing', (node) => { })

// untuk pesan dengan tag 'edge_routing' dan atribut id = abcd
miya.ws.on('CB:edge_routing,id:abcd', (node) => { })

// untuk pesan dengan tag 'edge_routing', id = abcd & isi pertama adalah 'routing_info'
miya.ws.on('CB:edge_routing,id:abcd,routing_info', (node) => { })
```
