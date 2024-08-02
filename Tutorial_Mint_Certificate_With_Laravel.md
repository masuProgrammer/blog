Berikut adalah versi terbaru dari tutorial lengkap untuk menjalankan fungsi `mintCertificate` di smart contract dari aplikasi Laravel PHP menggunakan library `web3p/web3.php`.

### Tutorial Lengkap: Menjalankan Fungsi `mintCertificate` di Smart Contract dari Laravel PHP

Tutorial ini akan memandu Anda melalui langkah-langkah untuk menghubungkan aplikasi Laravel Anda ke jaringan Ethereum testnet `TestEdge-2` dari HAQQ dan mengeksekusi fungsi `mintCertificate` di smart contract. Kita akan menggunakan library `web3p/web3.php`.

### Langkah 1: Instalasi Laravel dan Library `web3p/web3.php`

1. **Instal `web3p/web3.php`**:
    Instal library `web3p/web3.php` untuk menghubungkan Laravel ke Ethereum node:

    ```bash
    composer require web3p/web3.php
    ```

### Langkah 2: Konfigurasi Ethereum Node

1. **Tambah Konfigurasi ke `.env`**:
    Tambahkan URL RPC dari testnet `TestEdge-2` dan informasi kontrak ke file `.env`:

    ```
    ETHEREUM_NODE_URL=https://rpc.eth.testedge2.haqq.network
    CONTRACT_ADDRESS=0xYourContractAddress
    PRIVATE_KEY=YourPrivateKey
    ```

2. **Membuat ABI File**:
    Simpan ABI dari smart contract Anda dalam file `CertificateNFTABI.json` di folder `storage/app`.

### Langkah 3: Buat Service untuk Interaksi dengan Smart Contract

Buat service class untuk mengelola interaksi dengan smart contract.

1. **Buat File `EthereumService.php`**:
    Buat file `EthereumService.php` di dalam folder `app/Services` dan tambahkan kode berikut:

    ```php
    <?php

    namespace App\Services;
    
    use Web3\Web3;
    use Web3\Contract;
    use Illuminate\Support\Facades\Log;
    
    class EthereumService
    {
        protected $web3;
        protected $contract;
        protected $eth;
    
        public function __construct()
        {
            $provider = new \Web3\Providers\HttpProvider(env('ETHEREUM_NODE_URL'));
            $this->web3 = new Web3($provider);
            $this->contract = new Contract($provider, file_get_contents(storage_path('app/CertificateNFTABI.json')));
            $this->eth = $this->web3->eth;
        }
    
        public function mintCertificate($to, $certificateType, $certificateName, $holderName, $issueDate, $imageUrl)
        {
            $contractAddress = env('CONTRACT_ADDRESS');
    
            // Call contract function
            $this->contract->at($contractAddress)->send('mintCertificate', [$to, $certificateType, $certificateName, $holderName, $issueDate, $imageUrl], function ($err, $transactionHash) {
                if ($err !== null) {
                    Log::error('Error sending transaction: ' . $err->getMessage());
                    return;
                }
    
                Log::info('Transaction hash: ' . $transactionHash);
            });
        }
    }

    ```

### Langkah 4: Buat Controller untuk Menjalankan Fungsi Mint

1. **Buat File Controller**:
    Buat file `CertificateController.php` di dalam folder `app/Http/Controllers` dan tambahkan kode berikut:

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Services\EthereumService;
    use Illuminate\Http\Request;

    class CertificateController extends Controller
    {
        protected $ethereumService;

        public function __construct(EthereumService $ethereumService)
        {
            $this->ethereumService = $ethereumService;
        }

        public function mint(Request $request)
        {
            $request->validate([
                'to' => 'required|ethereum_address',
                'certificateType' => 'required|string',
                'certificateName' => 'required|string',
                'holderName' => 'required|string',
                'issueDate' => 'required|date',
                'imageUrl' => 'required|url',
            ]);

            $this->ethereumService->mintCertificate(
                $request->input('to'),
                $request->input('certificateType'),
                $request->input('certificateName'),
                $request->input('holderName'),
                $request->input('issueDate'),
                $request->input('imageUrl')
            );

            return response()->json(['message' => 'Minting request sent.']);
        }
    }
    ```

### Langkah 5: Tambahkan Route

1. **Tambahkan Route**:
    Tambahkan route ke file `routes/web.php`:

    ```php
    use App\Http\Controllers\CertificateController;

    Route::post('/mint', [CertificateController::class, 'mint']);
    ```

### Langkah 6: Menjalankan Aplikasi

1. **Jalankan Server Laravel**:
    Jalankan server Laravel:

    ```bash
    php artisan serve
    ```

2. **Kirim Permintaan POST**:
    Anda dapat mengirim permintaan POST ke endpoint `/mint` dengan data yang diperlukan menggunakan alat seperti Postman atau curl.

    Contoh permintaan dengan curl:

    ```sh
    curl -X POST http://127.0.0.1:8000/mint \
    -H "Content-Type: application/json" \
    -d '{
        "to": "0xRecipientAddress",
        "certificateType": "Certificate Type",
        "certificateName": "Certificate Name",
        "holderName": "Holder Name",
        "issueDate": "2024-08-01",
        "imageUrl": "https://example.com/image.png"
    }'
    ```

### Kesimpulan

Dengan mengikuti tutorial ini, Anda dapat menghubungkan aplikasi Laravel Anda ke jaringan Ethereum testnet `TestEdge-2` dari HAQQ dan menjalankan fungsi `mintCertificate` di smart contract. Pastikan semua konfigurasi sudah benar dan ABI smart contract telah disimpan dengan benar di folder `storage/app`.
