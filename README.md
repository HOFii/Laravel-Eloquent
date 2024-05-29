# LARAVEL LOGGING

## POINT UTAMA

### 1. Instalasi

-   Minimal PHP versi 8 atau lebih,

-   Composer versi 2 atau lebih,

-   Lalu pada cmd ketikan `composer create-project laravel/laravel=v10.2.4 belajar-laravel-eloquent`.

---

### 2. Pengenalan & Query Builder

- Laravel Eloquent adalah ORM (Object-Relational Mapping) bawaan dari framework Laravel yang memudahkan interaksi dengan database. Eloquent memungkinkan pengembang untuk bekerja dengan database menggunakan model PHP, sehingga tidak perlu menulis query SQL secara langsung.

-   Query Builder di Laravel adalah salah satu fitur yang sangat berguna untuk membangun dan menjalankan query database secara lebih efisien dan aman. Query Builder memungkinkan Anda untuk membangun query SQL dengan menggunakan sintaks PHP, sehingga lebih mudah dibaca dan ditulis.

-   Dokumentasi Query Bulder: https://laravel.com/docs/11.x/queries#main-content

---

### 3. Model

-   Kita bisa membuat Model menggunakan php artisan, gunakan perintah `php artisan make:model NamaModel` pada terminal.

-   Atau bisa juga dengan pendukungnya, misalnya `php artisan make:model NamaModel --migration --seed`.

-   Saat membuat Model biasanya menambahkan pendukung seperti database Migration atau database Seeder.

-   Buat Model dengan nama `php artisan make:model Category --migration --seed`

-   Kode Model Category

    ```PHP
    class Category extends Model
    {
        protected $table = "categories";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = false;
    }
    ```

-   Kode Model Category Migration

    ```PHP
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->string("id", 100)->nullable(false)->primary();
            $table->string("name", 100)->nullable(false);
            $table->text("description")->nullable();
            $table->timestamp("created_at")->nullable(false)->useCurrent();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }

    ```

-   Gunakan perintah `php artisan migrate` untuk menjalankan database Migration.
    ```

    ```

---

### 4. Insert

-   Gunaka method `save()` setelah membuat object dari kelas Model,

-   Kode insert

    ```PHP
    public function testInsert()
    {
        $category = new Category();
        $category->id = "GADGET";
        $category->name = "Gadget";
        $result = $category->save();

        self::assertTrue($result);
    }
    ```

-   Setelah membuat Model, untuk melakukan operasi CRUD terhadap Model, kita bisa menggunakan Query Builder.

-   Kode Insert Mary

    ```PHP
    public function testInsertMany()
    {
        $categories = [];
        for ($i = 0; $i < 10; $i++) {
            $categories[] = [
                "id" => "ID $i",
                "name" => "Name $i",
                'is_active' => true
            ];
        }

        // $result = Category::query()->insert($categories);
        $result = Category::insert($categories);

        self::assertTrue($result);

        // $total = Category::query()->count();
        $total = Category::count();

        self::assertEquals(10, $total);
    }
    ```

---

### 5. Find

-   Method dengan _prefix_ `find()` di Query Builder mempermudah untuk mendapatkan data menggunakan Primary Key.

-   Kode Caategory Seeder

    ```PHP
    public function run(): void
    {
        $category = new Category();
        $category->id = "FOOD";
        $category->name = "Food";
        $category->description = "Food Category";
        $category->is_active = true;
        $category->save();
    }
    ```

-   Kode Find

    ```PHP
       public function testFind()
    {
        $this->seed(CategorySeeder::class);

        // $category = Category::query()->find("FOOD");
        $category = Category::find("FOOD");

        self::assertNotNull($category);
        self::assertEquals("FOOD", $category->id);
        self::assertEquals("Food", $category->name);
        self::assertEquals("Food Category", $category->description);
    }
    ```

---

### 6. Update

-   Gunakan method `update()` atau `save()` untuk melakukan update terhadap Model,

-   Kode Update

    ```PHP
     public function testUpdate()
    {
        $this->seed(CategorySeeder::class);

        $category = Category::find("FOOD");
        $category->name = "Food Updated";

        $result = $category->update();
        self::assertTrue($result);
    }
    ```

-   Jika ingin melakukan update yang berdampak ke libih satu data, bisa menggunakan Query Builder.

-   Kode Update Many

    ```PHP
    public function testUpdateMany()
    {
        $categories = [];
        for ($i = 0; $i < 10; $i++) {
            $categories[] = [
                "id" => "ID $i",
                "name" => "Name $i",
                'is_active' => true
            ];
        }

        $result = Category::insert($categories);
        self::assertTrue($result);

        Category::whereNull("description")->update([
            "description" => "Updated"
        ]);
        $total = Category::where("description", "=", "Updated")->count();
        self::assertEquals(10, $total);

    }
    ```

---

### 7. Select

-   Gunakan Query Builder saat melakukan Select data yang datanya lebih dari satu.

-   Kode Select

    ```PHP
     public function testSelect()
    {
        for ($i = 0; $i < 5; $i++) {
            $category = new Category();
            $category->id = "ID $i";
            $category->name = "Name $i";
            $category->is_active = true;
            $category->save();
        }

        $categories = Category::whereNull("description")->get();
        self::assertEquals(5, $categories->count());
        $categories->each(function ($category) {
            self::assertNull($category->description);

             $category->description = "Updated";
            $category->update(); // select result
        });
    }
    ```

---

### 8. Delete

-   Gunakan method `delete()` untuk melakukan delete data, saat melakukan delete dengan Model dengan kata kunci `new` kita harus mengubah _attribute_ `$exits` dari _false_ ke _true_.

-   Kode Delete

    ```PHP
    public function testDelete()
    {
        $this->seed(CategorySeeder::class);

        $category = Category::find("FOOD");
        $result = $category->delete();
        self::assertTrue($result);

        $total = Category::count();
        self::assertEquals(0, $total);
    }
    ```

-   Sama seperti update saat ingin melakukan delete yang berdampak ke banyak data harus menggunakan Query Builder.

-   Kode Delete Many

    ```PHP
    public function testDeleteMany()
    {
        $categories = [];
        for ($i = 0; $i < 10; $i++) {
            $categories[] = [
                "id" => "ID $i",
                "name" => "Name $i",
                'is_active' => true
            ];
        }

        $result = Category::insert($categories);
        self::assertTrue($result);

        $total = Category::count();
        assertEquals(10, $total);

        Category::whereNull("description")->delete();

        $total = Category::count();
        assertEquals(0, $total);
    }
    ```

---

### 9. UUID

-   UUID (Universally Unique Identifier) di Laravel biasanya digunakan untuk mengidentifikasi entitas secara unik. UUID berguna karena mereka sangat tidak mungkin untuk menghasilkan duplikat, bahkan di antara database yang berbeda.

-   Dokumentasi UUID: https://laravel.com/docs/11.x/eloquent#uuid-and-ulid-keys

-   Buat Model baru `php artisan make:model Voucher --migration --seed`

-   Kode Voucher Migration

    ```PHP
     public function up(): void
    {
        Schema::create('vouchers', function (Blueprint $table) {
            $table->uuid("id")->nullable(false)->primary();
            $table->string("name", 100)->nullable(false);
            $table->string("voucher_code", 200)->nullable(false);
            $table->timestamp("created_at")->nullable(false)->useCurrent();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('vouchers');
    }
    ```

-   Kode Voucher Model

    ```PHP
        class Voucher extends Model
    {
        use HasUuids, SoftDeletes;

        protected $table = "vouchers";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = false;
    }
    ```

-   Kode Unit Voucher Model

    ```PHP
    public function testCreateVoucher()
    {
        $voucher = new Voucher();
        $voucher->name = "Sample Voucher";
        $voucher->voucher_code = "23414124214";
        $voucher->save();

        self::assertNotNull($voucher->id);
    }
    ```

-   UUID bisa juga diganti dengan non Primary key

-   Kode Voucher Non Primary Key

    ```PHP
    class Voucher extends Model
    {
        use HasUuids, SoftDeletes;

        protected $table = "vouchers";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = false;

        public function uniqueIds(): array
        {
            return [$this->primaryKey, "voucher_code"];
        }
    }
    ```

-   Kode test Voucher Non Primary Key

    ```PHP
    public function testCreateVoucherUUID()
    {
        $voucher = new Voucher();
        $voucher->name = "Sample Voucher";
        $voucher->save();

        self::assertNotNull($voucher->id);
        self::assertNotNull($voucher->voucher_code);
    }
    ```

---

### 10. Factory

-   Saat kita membuat object Model, biasanya kita harus ubah tiap atribut satu satu secara manual,

-   Laravel Eloquent memiliki fitur bernama Factory, ini sebenarnya adalah implementasi dari Design,

-   Patterns bernama Factory Patterns, dimana, kita membuat class Factory yang digunakan untuk membuat object
    Dengan begitu, jika kita membuat object yang hampir sama, kita bisa menggunakan Factory.

-   Dokumentasi: https://refactoring.guru/

#### Contoh Kasus

-   Misal kita akan membuat model Employee, dimana Employee memiliki title dan salary yang selalu sama untuk title yang sama,

-   Untuk mempermudah, kita bisa menggunakan Factory ketika membuat object Employee

-   Kode Employee Migration

    ```PHP
    public function up(): void
    {
        Schema::create('employees', function (Blueprint $table) {
            $table->string("id", 100)->nullable(false)->primary();
            $table->string("name", 100)->nullable(false);
            $table->string("title", 100)->nullable(false);
            $table->bigInteger("salary")->nullable(false);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('employees');
    }
    ```

-   Kode Employee Model

    ```PHP
    class Employee extends Model
    {
        use HasFactory;

        protected $table = "employees";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = true;
    }
    ```

-   Nama Factory secara default adalah nama Model + Factory,

-   Jika tidak menggunakan format yang sesuai, secara default Factory tidak bisa ditemukan,

-   Selain itu, di Model harus menggunakan trait HasFactory, untuk memberitahu bahwa Model ini memiliki Factory,

-   Untuk membuat class Factory, kita tidak perlu melakukannya secara manual, cukup gunakan perintah `php artisan make:factory NamaFactory`.

-   Kode Employee Factory

    ```PHP
    public function definition(): array
    {
        return [
            'id' => '',
            'name' => '',
            'title' => '',
            'salary' => 0
        ];
    }
    ```

-   Secara default, saat membuat Factory, kita wajib meng-override method `definition()`, yang digunakan sebagai _state_ awal data ketika dibuat menggunakan Factory,

-   Selanjutnya, kita bisa membuat _state_ lainnya, dimana _state_ awal akan menggunakan data dari method `definition()`.

-   kode Factory State

    ```PHP
    public function programmer(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'title' => 'Programmer',
                'salary' => 5000000
            ];
        });
    }

    public function seniorProgrammer(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'title' => 'Senior Programmer',
                'salary' => 10000000
            ];
        });
    }
    ```

-   kode Unit Factory

    ```PHP
     public function testFactory()
    {
        $employee1 = Employee::factory()->programmer()->make();
        $employee1->id = '1';
        $employee1->name = 'Employee 1';
        $employee1->save();

        self::assertNotNull(Employee::where('id', '1')->first());

        $employee2 = Employee::factory()->seniorProgrammer()->create([
            'id' => '2',
            'name' => 'Employee 2'
        ]);
        self::assertNotNull($employee2);
        self::assertNotNull(Employee::where('id', '2')->first());
    }
    ```

---

## PERTANYAAN & CATATAN TAMBAHAN

-   Tidak ada.

### KESIMPULAN

-   Laravel Eloquent adalah alat yang sangat powerful untuk interaksi database dalam aplikasi Laravel. Dengan fitur-fitur seperti CRUD sederhana, relationship handling, dan kemampuan query yang kuat, Eloquent tidak hanya meningkatkan produktivitas developer tetapi juga membuat kode lebih bersih dan mudah dipelihara
