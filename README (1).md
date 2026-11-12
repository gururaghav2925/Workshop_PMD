# 🌍 Country & Currency CRUD App (Android + SQLite)

An Android application that performs **CRUD operations** (Create, Read, Update, Delete) on **country and currency** data using a local **SQLite database**, with records displayed in a **ListView**.

---

## 📌 Objective

- Understand the SQLite database in Android for persistent local storage.
- Create an Android application capable of adding, reading, updating, and deleting records.
- Use a `ListView` to display records dynamically.
- Enhance knowledge of Android activities, intents, and adapters.

---

## ✨ Features

- 📋 View all countries and their currencies in a `ListView`
- ➕ Add a new country–currency record
- ✏️ Update an existing record
- 🗑️ Delete a record
- 💾 Data stored locally with SQLite (no server needed)

---

## 📚 Theory

**SQLite** is a lightweight relational database embedded within Android. It stores structured data in tables and supports SQL queries. It is local and needs no server, which makes it ideal for mobile apps.

| CRUD Operation | Description |
|---|---|
| Create (Insert) | Add new records |
| Read (Select) | Retrieve records for display |
| Update | Modify existing records |
| Delete | Remove records from the database |

- **ListView:** A scrollable list widget that binds data dynamically through adapters.
- **Activities and Intents:** Each screen is an Activity; an Intent navigates between activities and passes data.

---

## 🛠️ Tools Required

- Android Studio (2022+)
- Java SDK
- Android Emulator or Physical Device
- SQLite Database (embedded in Android)

---

## 🧱 Project Structure

| File | Purpose |
|---|---|
| `DatabaseHelper.java` | Creates the database and table, handles version upgrades |
| `DBManager.java` | Performs the CRUD operations |
| `Country.java` | Model class (id, name, currency) |
| `CountryListActivity.java` | Launcher screen showing the `ListView` |
| `AddCountryActivity.java` | Adds a new country |
| `ModifyCountryActivity.java` | Updates or deletes a country |

```
app/src/main/
├── java/com/example/countryapp/
│   ├── DatabaseHelper.java
│   ├── DBManager.java
│   ├── Country.java
│   ├── CountryListActivity.java
│   ├── AddCountryActivity.java
│   └── ModifyCountryActivity.java
├── res/layout/
│   ├── activity_country_list.xml
│   ├── activity_add_country.xml
│   └── activity_modify_country.xml
└── AndroidManifest.xml
```

### 🗄️ Database

- **Database:** `CountryDB`
- **Table:** `countries`

| Column | Type |
|---|---|
| `id` | INTEGER PRIMARY KEY AUTOINCREMENT |
| `name` | TEXT |
| `currency` | TEXT |

---

## 🚀 Implementation

### Step 1: Create a New Android Project

1. Open Android Studio → **File → New → New Project**.
2. Select **Empty Activity**.
3. Name the project (e.g., `CountryCRUDApp`) and choose **Java** as the language.
4. Finish the setup.

This creates a project with `MainActivity` and the necessary Gradle files. `MainActivity` is later replaced by `CountryListActivity` as the launcher.

### Step 2: `DatabaseHelper.java`

`SQLiteOpenHelper` creates and manages the database and handles version upgrades.

```java
package com.example.countryapp;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "CountryDB";
    private static final int DATABASE_VERSION = 1;

    public static final String TABLE_COUNTRIES = "countries";
    public static final String COLUMN_ID = "id";
    public static final String COLUMN_NAME = "name";
    public static final String COLUMN_CURRENCY = "currency";

    private static final String TABLE_CREATE =
            "CREATE TABLE " + TABLE_COUNTRIES + " (" +
            COLUMN_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
            COLUMN_NAME + " TEXT, " +
            COLUMN_CURRENCY + " TEXT);";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(TABLE_CREATE);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_COUNTRIES);
        onCreate(db);
    }
}
```

- `DATABASE_NAME` and `DATABASE_VERSION` define the database.
- `onCreate()` executes SQL to create the table.
- `onUpgrade()` handles database schema changes.

### Step 3: `DBManager.java`

Handles CRUD operations using `SQLiteDatabase` methods (`insert`, `query`, `update`, `delete`).

```java
package com.example.countryapp;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import java.util.ArrayList;
import java.util.List;

public class DBManager {

    private DatabaseHelper dbHelper;
    private SQLiteDatabase database;

    public DBManager(Context context) {
        dbHelper = new DatabaseHelper(context);
        database = dbHelper.getWritableDatabase();
    }

    // Insert new country
    public long addCountry(String name, String currency) {
        ContentValues values = new ContentValues();
        values.put(DatabaseHelper.COLUMN_NAME, name);
        values.put(DatabaseHelper.COLUMN_CURRENCY, currency);
        return database.insert(DatabaseHelper.TABLE_COUNTRIES, null, values);
    }

    // Get all countries
    public List<Country> getAllCountries() {
        List<Country> countryList = new ArrayList<>();
        Cursor cursor = database.query(DatabaseHelper.TABLE_COUNTRIES,
                null, null, null, null, null, null);
        if (cursor.moveToFirst()) {
            do {
                int id = cursor.getInt(
                    cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_ID));
                String name = cursor.getString(
                    cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_NAME));
                String currency = cursor.getString(
                    cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_CURRENCY));
                countryList.add(new Country(id, name, currency));
            } while (cursor.moveToNext());
        }
        cursor.close();
        return countryList;
    }

    // Update a country
    public int updateCountry(int id, String name, String currency) {
        ContentValues values = new ContentValues();
        values.put(DatabaseHelper.COLUMN_NAME, name);
        values.put(DatabaseHelper.COLUMN_CURRENCY, currency);
        return database.update(DatabaseHelper.TABLE_COUNTRIES, values,
                DatabaseHelper.COLUMN_ID + "=?",
                new String[]{String.valueOf(id)});
    }

    // Delete a country
    public int deleteCountry(int id) {
        return database.delete(DatabaseHelper.TABLE_COUNTRIES,
                DatabaseHelper.COLUMN_ID + "=?",
                new String[]{String.valueOf(id)});
    }
}
```

- `addCountry()` inserts a new record.
- `getAllCountries()` reads all records into a `List`.
- `updateCountry()` modifies a record by ID.
- `deleteCountry()` removes a record by ID.

### Step 4: `Country.java`

Model class that defines the data structure used across the app.

```java
package com.example.countryapp;

public class Country {

    private int id;
    private String name;
    private String currency;

    public Country(int id, String name, String currency) {
        this.id = id;
        this.name = name;
        this.currency = currency;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCurrency() { return currency; }

    @Override
    public String toString() {
        return name + " - " + currency;
    }
}
```

- Stores `id`, `name`, and `currency` for each country.
- `toString()` is used to display the country in the `ListView`.

### Step 5: `CountryListActivity.java`

Displays all records in a `ListView` and handles navigation to the Add/Modify screens.

```java
package com.example.countryapp;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import androidx.appcompat.app.AppCompatActivity;
import java.util.List;

public class CountryListActivity extends AppCompatActivity {

    private ListView listView;
    private DBManager dbManager;
    private List<Country> countryList;
    private ArrayAdapter<Country> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        dbManager = new DBManager(this);
        loadCountries();

        listView.setOnItemClickListener((parent, view, position, id) -> {
            Country selectedCountry = countryList.get(position);
            Intent intent = new Intent(CountryListActivity.this,
                    ModifyCountryActivity.class);
            intent.putExtra("countryId", selectedCountry.getId());
            startActivity(intent);
        });
    }

    private void loadCountries() {
        countryList = dbManager.getAllCountries();
        adapter = new ArrayAdapter<>(this,
                android.R.layout.simple_list_item_1, countryList);
        listView.setAdapter(adapter);
    }

    public void addCountry(View view) {
        startActivity(new Intent(this, AddCountryActivity.class));
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadCountries(); // Refresh list when returning
    }
}
```

- Retrieves all countries using `DBManager`.
- Populates the `ListView` with an `ArrayAdapter`.
- Clicking an item navigates to `ModifyCountryActivity`.

### Step 6: `AddCountryActivity.java`

Adds new countries to the database.

```java
package com.example.countryapp;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class AddCountryActivity extends AppCompatActivity {

    private EditText etName, etCurrency;
    private DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_country);

        etName = findViewById(R.id.etName);
        etCurrency = findViewById(R.id.etCurrency);
        dbManager = new DBManager(this);
    }

    public void saveCountry(View view) {
        String name = etName.getText().toString();
        String currency = etCurrency.getText().toString();

        if (!name.isEmpty() && !currency.isEmpty()) {
            dbManager.addCountry(name, currency);
            Toast.makeText(this, "Country added", Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(this, "Please enter all fields",
                    Toast.LENGTH_SHORT).show();
        }
    }
}
```

- Reads user input.
- Validates fields.
- Adds the country to the database via `DBManager`.

### Step 7: `ModifyCountryActivity.java`

Updates or deletes a selected country.

```java
package com.example.countryapp;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class ModifyCountryActivity extends AppCompatActivity {

    private EditText etName, etCurrency;
    private DBManager dbManager;
    private int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_country);

        etName = findViewById(R.id.etName);
        etCurrency = findViewById(R.id.etCurrency);
        dbManager = new DBManager(this);

        countryId = getIntent().getIntExtra("countryId", -1);

        Country country = dbManager.getAllCountries().stream()
                .filter(c -> c.getId() == countryId)
                .findFirst().orElse(null);

        if (country != null) {
            etName.setText(country.getName());
            etCurrency.setText(country.getCurrency());
        }
    }

    public void updateCountry(View view) {
        dbManager.updateCountry(countryId,
                etName.getText().toString(),
                etCurrency.getText().toString());
        Toast.makeText(this, "Country updated", Toast.LENGTH_SHORT).show();
        finish();
    }

    public void deleteCountry(View view) {
        dbManager.deleteCountry(countryId);
        Toast.makeText(this, "Country deleted", Toast.LENGTH_SHORT).show();
        finish();
    }
}
```

- Retrieves the country ID via intent.
- Populates fields with existing data.
- Updates or deletes the record on button click.

### Step 8: Layout Files

- `activity_country_list.xml`: `ListView` and Add button.
- `activity_add_country.xml`: Inputs for name and currency.
- `activity_modify_country.xml`: Inputs plus Update/Delete buttons.

Example, `activity_country_list.xml`:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <Button
        android:text="Add Country"
        android:onClick="addCountry"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

### Step 9: `AndroidManifest.xml`

```xml
<application
    android:theme="@style/Theme.CRUD">

    <activity android:name=".CountryListActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>

    <activity android:name=".AddCountryActivity"/>
    <activity android:name=".ModifyCountryActivity"/>
</application>
```

---

## 📸 Screenshots

| Country List | Add Country |
|---|---|
| ![Country List](screenshots/country_list.png) | ![Add Country](screenshots/add_country.png) |

| Modify / Delete Country | Country List (after changes) |
|---|---|
| ![Modify Delete](screenshots/modify_delete_country.png) | ![Country List After](screenshots/country_list_after.png) |

---

## ▶️ How to Run

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
```

1. Open the project in **Android Studio** (**File → Open**).
2. Wait for Gradle sync to finish.
3. Connect a device or start an emulator.
4. Click **Run ▶**.

---

## ✅ Conclusion

- Developed a **fully functional CRUD app** using SQLite.
- Learned about **database creation, insertion, retrieval, update, and deletion**.
- Practiced using **ListView, ArrayAdapter, Activities, Intents, and UI components**.
- Gained **hands-on experience in Android Studio and Java programming**.
