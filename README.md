# artisan-generate-model-from-database
Custom Laravel Artisan Command untuk generate Model dari Database yang sudah ada pada Laravel 11.

## Persiapan
Pastikan anda telah melakukan konfigurasi Database pada Laravel 11 dengan benar, sebagai contoh disini saya menggunakan server MariaDB dengan nama database ```laravel```
```bash
DB_CONNECTION=mariadb
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
DB_COLLATION=utf8mb4_unicode_ci
```

## Buat Perintah Artisan
Untuk membuat perintah baru, gunakan perintah ```make:command```
diikuti nama perintah yang akan dibuat, sebagai contoh ```GenerateModel```.

```bash
  php artisan make:command GenerateModel
```

Perintah ini akan membuat kelas baru, yaitu ```app/Console/Commands/GenerateModel.php```.
Selanjutnya buka file ```app/Console/Commands/GenerateModel.php``` lalu sesuaikan konfigurasi seperti contoh berikut:
```bash
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Filesystem\Filesystem;
use Illuminate\Support\Facades\Schema;
use Illuminate\Support\Str;

class GenerateModel extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    //perintah yang akan dijalankan, 
    protected $signature = 'make:generate-model';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Genereta model dari Database';

    protected $files;

    public function __construct(Filesystem $files)
    {
        parent::__construct();
        $this->files = $files;
    }

    //fungsi untuk mengambil kolom yang dijadikan primary key 
    public function getPK($table)
    {
        $pk = '';
        $indexes = Schema::getIndexes($table);
        for ($i = 0; $i < count($indexes); $i++) {
            if ($indexes[$i]['name'] == 'primary') {
                $pk = $indexes[$i]['columns'][0];
                break;
            }
        }

        return $pk;
    }

    /**
     * Execute the console command.
     */
    public function handle()
    {
        $tables = Schema::getTables();
        for ($i = 0; $i < count($tables); $i++) {
            $tableName = $tables[$i]['name'];
            $name = Str::studly($tableName);
            $pk = $this->getPK($tableName);
            $columns = Schema::getColumns($tableName);
            $columnsList = '';

            for ($j = 0; $j < count($columns); $j++) {
                //kondisi jika nama kolom sebagai primary key, dan nama kolom adalah created_at dan update maka tidak akan dimasukan kedalam $columnsList
                if ($columns[$j]['name'] != $pk && $columns[$j]['name'] != 'created_at' && $columns[$j]['name'] != 'updated_at') {
                    $columnsList .= "'" . $columns[$j]['name'] . "',";
                }
            }

            $path = app_path('Models/' . $name . '.php');
            if ($this->files->exists($path)) {
                $this->error('Model sudah ada!');
                return;
            }

            $stub = $this->files->get(__DIR__ . '/stubs/generatemodel.stub');
            $stub = str_replace('{{ namespace }}', 'App\Models', $stub);
            $stub = str_replace('{{ class }}', $name, $stub);
            $stub = str_replace('{{ nama_tabel }}', $tableName, $stub);
            $stub = str_replace('{{ primary_key }}', $pk, $stub);
            $stub = str_replace('{{ list_kolom }}', $columnsList, $stub);
            $this->files->put($path, $stub);
            $this->info('Model berhasil dibuat!.');
        };
    }
}

```

Buat file baru dengan nama ```generatemodel.stub``` pada folder ```app/Console/Commands/stubs``` lalu sesuaikan seperti contoh berikut:

```bash
<?php

namespace {{ namespace }};

use Illuminate\Database\Eloquent\Model;

class {{ class }} extends Model
{
    protected $table = '{{ nama_tabel }}';
    protected $primaryKey = '{{ primary_key }}';
    protected $fillable = [
        {{ list_kolom }}
    ];
}
```

## Jalankan Perintah Artisan
Terlebih dahulu jalankan perintah 
```bash
php artisan list
```
pastikan perintah ```make:generate-model``` sudah ada didalam ```list```.
Selanjutkan jalankan perintah
```bash
php artisan make:generate-model
```
Output ```Model berhasil dibuat!.``` akan ditampilkan sebanyak jumlah tabel yang ada pada Database anda, dan file model akan otomatis dibuat pada folder ```app/Models``` sesuai dengan nama tabel.
Berikut contoh file hasil generate:
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class DataSiswa extends Model
{
    protected $table = 'data_siswa';
    protected $primaryKey = 'id_data_siswa';
    protected $fillable = [
        'nama_siswa','alamat_siswa','tempat_lahir_siswa','tgl_lahir_siswa',
    ];
}

```
