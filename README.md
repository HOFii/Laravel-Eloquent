# LARAVEL ELOQUENT

## POINT UTAMA

### 1. Instalasi

-   Minimal PHP versi 8 atau lebih,

-   Composer versi 2 atau lebih,

-   Lalu pada cmd ketikan `composer create-project laravel/laravel=v10.2.4 belajar-laravel-eloquent`.

---

### 2. Pengenalan & Query Builder

- Laravel Eloquent adalah ORM (Object-Relational Mapping) bawaan dari framework Laravel yang memudahkan interaksi dengan database. Eloquent memungkinkan pengembang untuk bekerja dengan database menggunakan model PHP, sehingga tidak perlu menulis query SQL secara langsung.

-   Query Builder di Laravel adalah salah satu fitur yang sangat berguna untuk membangun dan menjalankan query database secara lebih efisien dan aman. Query Builder memungkinkan Anda untuk membangun query SQL dengan menggunakan sintaks PHP, sehingga lebih mudah dibaca dan ditulis.

-   Dokumentasi [Query Builder](https://laravel.com/docs/11.x/queries#main-content)

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

-   Dokumentasi [UUID](https://laravel.com/docs/11.x/eloquent#uuid-and-ulid-keys)

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

### 10. Timestamps

-   `$timestamps` yang jika menggunakan value _true_, maka secara otomatis `Eloquent` akan menambahkan _attribute_ `create_at` dan `update_at` pada Model.

-   yang artinya kita harus membuat kolom tersebut, dan di _Migrations_ itu bisa dilakukan secara otomatis.

-   Buat file baru `php artisan make:model Comment --migration --seed`.

-   Kode Comment Migration

    ```PHP
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            // $table->bigInteger("id")->autoIncrement();
            // $table->id();
            $table->integer("id")->autoIncrement();
            $table->string("email", 100)->nullable(false);
            $table->string("title", 200)->nullable(false);
            $table->text("comment")->nullable();
            $table->timestamps(); // created_at, updated_at
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
    ```

-   Kode Model Comment

    ```PHP
    class Comment extends Model
    {
        protected $table = "comments";
        protected $primaryKey = "id";
        protected $keyType = "int";
        public $incrementing = true;
        public $timestamps = true;

    }
    ```

-   Kode Comment Test

    ```PHP
     public function testCreateComment()
    {
        $comment = new Comment();
        $comment->email = "gusti@hori.com";
        $comment->title = "Sample Title";
        $comment->comment = "Sample Comment";
        $comment->commentable_id = '1';
        $comment->commentable_type = 'product';

        $comment->save();

        self::assertNotNull($comment->id);
    }
    ```

---

### 11. Default Attribut Values

-   Saat membuat tabel, bisa juga membuat default value, namun kadang itu kurang flexible, karena kita tidak bisa mengubah-ubah secara mudah,

-   Laravel Model memiliki fitur default attribute values, dimana kita bisa membuat default value,

-   Untuk menentukan default values, kita bisa menggunakan attribute `$attributes` yang berisi _associative array_ kolom => default value.

-   Kode Model Comment

    ````PHP
    protected $attributes = [
        "title" => "Sample Title",
        "comment" => "Sample Comment"
    ];
        ```
    ````

-   Kode Coment Default Attribute Value

    ```PHP
     public function testDefaultAttributeValues()
    {
        $comment = new Comment();
        $comment->email = "gusti@hofi.com";
        $comment->commentable_id = '1';
        $comment->commentable_type = 'product';

        $comment->save();

        self::assertNotNull($comment->id);
        self::assertNotNull($comment->title);
        self::assertNotNull($comment->comment);
    }
    ```

---

### 12. Fillable Attributes

-   Saat kita membuat object model yang datanya misalnya dikirim dari web form atau juga mungkin body http request,

-   Akan sangat merepotkan jika harus menambahkan satu persatu _attribute_ nya ke object model,

-   Method `create(attributes) pada` di laravel memudahkan membuat model secara otomatis menggunakan query builder.

-   Kode Membuat Object

    ```PHP
     public function testCreate()
    {
        $request = [
            "id" => "FOOD",
            "name" => "Food",
            "description" => "Food Category"
        ];

        $category = new Category($request);
        $category->save();

        self::assertNotNull($category->id);
    }
    ```

-   Secara default, semua _attribute_ di Model tidak bisa di set secara masal menggunakan method `create()`,

-   Hal ini untuk menjaga agar tidak ada data salah yang akhirnya tidak sengaja mengubah data di database, misal jika ada Model User, lalu terdapat _attribute_ `is_admin`, jika sampai ada request yang mengirim _attribute_ `is_admin: true`, maka secara otomatis data di database akan diubah,

-   Oleh karena itu, kita harus beri tahu ke Laravel, _attribute_ mana saja yang bisa diubah secara masal Kita bisa gunakan _attribute_ `$fillable` di Model nya.

---

    ```PHP
    class Category extends Model
    {
        protected $table = "categories";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = false;

        protected $fillable = [ // tambahkan variable $fillable pada category model
            "id",
            "name",
            "description"
        ];
    }
    ```

---

### 13. Relationship

-   Relasi antar tabel tersebut bisa kita lakukan secara manual di Laravel, namun artinya kita harus melakukan join tabel secara manual,

-   Di Laravel Eloquent mendukung Model Relationship, sehingga proses join tabel tidak perlu kita lakukan secara manual,

-   Kita akan bahas secara bertahap relasi-relasi antar tabel di Laravel Eloquent di materi-materi berikutnya

---

### 14. One to One

-   Relasi _One to One_ didukung di Laravel Eloquent, dengan method `hasOne()` pada model,

-   Pada model kebalikannya, bisa menggunakan method `belongsTo()` pada model.

-   Contoh kasus, ada dua model, _Customer_ dan _Wallet_ dimana satu _Customer_ memiliki satu _Wallet_.

-   Kode membuat model

-   Customer `php artisan make:model Customer --migration --seed`,

-   Wallet `php artisan make:model Wallet --migration --seed`

-   Kode Customer Migration

    ```PHP
    public function up(): void
    {
        Schema::create('customers', function (Blueprint $table) {
            $table->string("id", 100)->nullable(false)->primary();
            $table->string("name", 100)->nullable(false);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('customers');
    }
    ```

-   Kode Wallet Migation

    ```PHP
    public function up(): void
    {
        Schema::create('wallets', function (Blueprint $table) {
            $table->integerIncrements("id");
            $table->string("customer_id", 100)->nullable(false);
            $table->bigInteger("amount")->nullable(false)->default(0);
            $table->foreign("customer_id")->on("customers")->references("id");
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('wallets');
    }
    ```

-   Jalankan migration `php artisan migrate`,

-   Kode Customer Model

    ```PHP
    class Customer extends Model
    {
        protected $table = "customers";
        protected $primaryKey = "id";
        protected $keyType = "string";
        public $incrementing = false;
        public $timestamps = false;

        public function wallet(): HasOne //method hasOne untuk relasi one to one
        {
            return $this->hasOne(Wallet::class, "customer_id", "id");
        }
    }
    ```

-   Kode Wallet Model

    ```PHP
    class Wallet extends Model
    {
        protected $table = "wallets";
        protected $primaryKey = "id";
        protected $keyType = "int";
        public $incrementing = true;
        public $timestamps = false;

        public function customer(): BelongsTo //method belongsTo untuk relasi kebalikannya
        {
            return $this->belongsTo(Customer::class, "customer_id", "id");
        }
    }
    ```

-   Kode Seeder Customer & Wallet

    ```PHP

    // customer seeder
    class CustomerSeeder extends Seeder
    {
        /**
        * Run the database seeds.
        */
        public function run(): void
        {
            $customer = new Customer();
            $customer->id = "GUSTI";
            $customer->name = "Gusti";
            $customer->email = "gusti@hofi.com";
            $customer->save();
        }
    }

    // wallet seeder
    class WalletSeeder extends Seeder
    {
        public function run(): void
        {
            $wallet = new Wallet();
            $wallet->amount = 1000000;
            $wallet->customer_id = "GUSTI";
            $wallet->save();
        }
    }
    ```

-   Kode Test Query One to One

    ```PHP
    public function testOneToOne()
    {
        $this->seed([CustomerSeeder::class, WalletSeeder::class]);

        $customer = Customer::find("GSUTI");
        self::assertNotNull($customer);

        // $wallet = Wallet::where("customer_id", $customer->id)->first();
        $wallet = $customer->wallet;
        self::assertNotNull($wallet);

        self::assertEquals(1000000, $wallet->amount);
    }
    ```

---

### 15. Factory

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
