# Implementasi Chatbot (BotMan)

## 1. Konsep Chatbot di Bengkelin

Chatbot Bengkelin menggunakan **BotMan** dengan pendekatan **rule-based (berbasis aturan)**. Berbeda dengan chatbot berbasis NLP/AI yang memahami bahasa alami, chatbot rule-based bekerja dengan cara mencocokkan input pengguna dengan aturan yang sudah ditentukan.

### Perbandingan Pendekatan

| Aspek | Rule-based (BotMan) | AI/NLP (ChatGPT, Dialogflow) |
|-------|---------------------|------------------------------|
| Implementasi | Sederhana, kode PHP | Kompleks, perlu training model |
| Akurasi | Tinggi untuk skenario terbatas | Lebih fleksibel, memahami variasi kata |
| Biaya | Gratis | Berbayar (API calls) |
| Cocok untuk | Menu terstruktur, FAQ | Percakapan bebas |
| Keterbatasan | Input harus tepat sesuai aturan | Bisa "salah mengerti" konteks |

Untuk aplikasi bengkel dengan skenario terbatas (info layanan, info pengiriman, cara daftar), pendekatan rule-based sudah sangat memadai dan lebih mudah dipelihara.

---

## 2. Arsitektur BotMan

```
Mobile App
    │
    │  POST /api/chatbot
    │  Body: { "message": "1" }
    ▼
┌─────────────────────────────────────────────────────────────┐
│  ChatbotController::handle()                                │
│                                                             │
│  $botman = app('botman')   ← instance dari Service Provider │
│  $botman->hears('{message}', callback)                      │
│  $botman->listen()                                          │
└─────────────────────────────────────────────────────────────┘
         │
         ├── Input '1' → showPengirimanProdukMenu()
         │                    │
         │               $botman->ask(pertanyaan, callback)
         │               ← percakapan 1 langkah
         │
         ├── Input '2' → $botman->startConversation(new LayananBengkelConversation())
         │               ← percakapan multi-langkah
         │
         ├── Input '3' → infoRegister()
         │
         └── Input lain → pesan error
```

---

## 3. Setup BotMan

### 3.1 Instalasi

```bash
composer require botman/botman
composer require botman/driver-web
```

### 3.2 Service Provider

BotMan didaftarkan sebagai singleton di `app/Providers/BotManServiceProvider.php`:

```php
namespace App\Providers;

use BotMan\BotMan\BotManFactory;
use BotMan\BotMan\Cache\LaravelCache;
use BotMan\Drivers\Web\WebDriver;
use Illuminate\Support\ServiceProvider;

class BotManServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton('botman', function ($app) {
            $config = [
                'web' => [
                    'matchingData' => [
                        'driver' => 'web',
                    ],
                ],
            ];

            return BotManFactory::create($config, new LaravelCache());
        });
    }
}
```

Daftarkan di `config/app.php`:
```php
'providers' => [
    // ...
    App\Providers\BotManServiceProvider::class,
],
```

---

## 4. Implementasi ChatbotController

```php
namespace App\Http\Controllers\API;

use App\Conversations\LayananBengkelConversation;
use App\Conversations\PengirimProdukConversation;
use BotMan\BotMan\BotMan;
use Illuminate\Http\Request;

class ChatbotController extends Controller
{
    /**
     * Entry point: menerima semua pesan chatbot
     * Route: POST /api/chatbot
     */
    public function handle()
    {
        $botman = app('botman');

        // Tangkap semua pesan ({message} = wildcard)
        $botman->hears('{message}', function (BotMan $bot, $message) {
            switch ($message) {
                case '1':
                    $this->showPengirimanProdukMenu($bot);
                    break;
                case '2':
                    $bot->startConversation(new LayananBengkelConversation());
                    break;
                case '3':
                    $this->infoRegister($bot);
                    break;
                case '0':
                    $this->showMainMenu($bot);
                    break;
                default:
                    $bot->reply('Harap masukkan angka pilihan di atas atau ketik 0 untuk kembali ke menu utama.');
            }
        });

        $botman->listen();
    }

    /**
     * Tampilkan menu utama
     */
    public function showMainMenu(BotMan $bot)
    {
        $bot->reply(
            "Selamat datang di menu utama. Pilih opsi berikut:\n" .
            "1. Pengiriman Produk\n" .
            "2. Layanan Bengkel\n" .
            "3. Cara Mendaftar Akun\n" .
            "0. Kembali ke Menu Utama"
        );
    }

    /**
     * Sub-menu pengiriman produk
     * Menggunakan $bot->ask() untuk percakapan satu langkah
     */
    public function showPengirimanProdukMenu(BotMan $bot)
    {
        $message = "Layanan Pengiriman Produk:\n" .
                   "1. Informasi pickup\n" .
                   "2. Informasi jasa kurir\n";

        $bot->reply($message);

        // ask() mengirim pertanyaan dan menunggu jawaban (percakapan 1 langkah)
        $bot->ask('Ketik pilihan Anda:', function ($answer, $bot) {
            $selected = strtolower($answer->getText());

            if ($selected === '1') {
                $bot->say(
                    "❗INFO PICKUP DI TEMPAT❗\n\n" .
                    "1. Konsumen dapat langsung mengambil barang.\n" .
                    "2. Tidak ada biaya tambahan.\n" .
                    "3. Bisa cek kondisi barang.\n\n" .
                    "Persyaratan:\n" .
                    "- Bawa bukti pesanan\n" .
                    "- Datang tepat waktu"
                );
            } elseif ($selected === '2') {
                $bot->say(
                    "❗INFO JASA KURIR❗\n\n" .
                    "1. Barang diantar ke alamat.\n" .
                    "2. Bisa lacak status pengiriman.\n\n" .
                    "Pilihan Layanan:\n" .
                    "- Reguler, express, same-day"
                );
            } else {
                $bot->say("Pilihan tidak valid. Kembali ke menu utama.");
                $this->showMainMenu($bot);
            }
        });
    }

    /**
     * Info cara registrasi akun
     */
    public function infoRegister(BotMan $bot)
    {
        $url = url('/userregister');
        $bot->reply(
            "❗INFO REGISTRASI AKUN❗\n\n" .
            "1. Kunjungi: {$url}\n" .
            "2. Isi formulir dengan benar\n" .
            "3. Klik 'Daftar' untuk menyelesaikan"
        );
    }
}
```

---

## 5. Implementasi Conversation (Multi-langkah)

Untuk percakapan yang memerlukan beberapa langkah bolak-balik, BotMan menyediakan kelas `Conversation`. Berikut contoh implementasi:

```php
namespace App\Conversations;

use BotMan\BotMan\Conversation;
use BotMan\BotMan\Messages\Incoming\Answer;

class LayananBengkelConversation extends Conversation
{
    protected $pilihanLayanan;

    /**
     * Titik masuk percakapan
     */
    public function run()
    {
        $this->askJenisLayanan();
    }

    /**
     * Langkah 1: Tanya jenis layanan
     */
    protected function askJenisLayanan()
    {
        $this->ask(
            "Informasi layanan bengkel apa yang Anda butuhkan?\n" .
            "1. Ganti Oli\n" .
            "2. Tune Up\n" .
            "3. Perbaikan Rem\n" .
            "0. Kembali",
            function (Answer $answer) {
                $this->pilihanLayanan = $answer->getText();
                $this->replyLayananInfo();
            }
        );
    }

    /**
     * Langkah 2: Berikan informasi sesuai pilihan
     */
    protected function replyLayananInfo()
    {
        switch ($this->pilihanLayanan) {
            case '1':
                $this->say(
                    "Layanan Ganti Oli:\n" .
                    "- Direkomendasikan setiap 5.000 km\n" .
                    "- Tersedia oli mineral, semi-sintetis, sintetis\n" .
                    "- Harga mulai Rp 50.000"
                );
                break;
            case '2':
                $this->say(
                    "Layanan Tune Up:\n" .
                    "- Pembersihan karburator/injektor\n" .
                    "- Pengecekan busi & filter udara\n" .
                    "- Harga mulai Rp 150.000"
                );
                break;
            case '3':
                $this->say(
                    "Layanan Perbaikan Rem:\n" .
                    "- Penggantian kampas rem\n" .
                    "- Pengecekan cakram/tromol\n" .
                    "- Harga mulai Rp 100.000"
                );
                break;
            case '0':
                $this->say("Kembali ke menu utama. Ketik 0 untuk melihat menu.");
                break;
            default:
                $this->say("Pilihan tidak valid.");
        }
    }
}
```

---

## 6. Cara Integrasi di Aplikasi Mobile (Flutter)

Di sisi Flutter, chatbot dikonsumsi melalui HTTP POST biasa:

```dart
class ChatbotService {
  static const String _baseUrl = 'https://api.bengkelin.com/api';

  Future<String> sendMessage(String message) async {
    final response = await http.post(
      Uri.parse('$_baseUrl/chatbot'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({'message': message}),
    );

    if (response.statusCode == 200) {
      // BotMan merespons dalam format teks langsung
      final data = json.decode(response.body);
      return data['messages'][0]['text'] ?? '';
    }

    throw Exception('Chatbot error');
  }
}
```

---

## 7. Keterbatasan dan Pengembangan

### Keterbatasan Saat Ini

1. Chatbot bersifat **stateless** — tidak mengingat percakapan sebelumnya antar sesi.
2. Hanya mendukung **input numerik** (berbasis menu angka).
3. Tidak bisa menjawab pertanyaan di luar skrip yang sudah ditentukan.

### Ide Pengembangan

1. **Integrasi NLP** — menggunakan Dialogflow atau OpenAI API untuk memahami pertanyaan bebas.
2. **Chatbot berbasis database** — jawaban disimpan di database agar bisa dikelola admin tanpa ubah kode.
3. **Riwayat percakapan** — menyimpan log chat di database untuk analitik.
4. **Rekomendasi bengkel** — chatbot bisa menyarankan bengkel berdasarkan lokasi dan masalah kendaraan.
