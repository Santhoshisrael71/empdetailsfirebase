
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: Santhosh P
Registeration Number :212224220088
```
## Expense.java
```

package com.example.firebase01;

public class Expense {
    public String description;
    public String amount;
    public String category;

    public Expense() {
        // Required for Firebase
    }

    public Expense(String description, String amount,
                   String category) {
        this.description = description;
        this.amount = amount;
        this.category = category;
    }
}
```
## MAINACTIVITY:
```

package com.example.firebase01;

import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.database.*;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etDescription, etAmount, etCategory;
    Button btnAdd, btnUpdate, btnDelete;
    ListView listView;

    DatabaseReference dbRef;
    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();
    ArrayAdapter<String> adapter;
    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etDescription = findViewById(R.id.etDescription);
        etAmount = findViewById(R.id.etAmount);
        etCategory = findViewById(R.id.etCategory);
        btnAdd = findViewById(R.id.btnAdd);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance()
                .getReference("Expenses");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );
        listView.setAdapter(adapter);

        dbRef.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(DataSnapshot snapshot) {
                items.clear();
                keys.clear();

                for (DataSnapshot d : snapshot.getChildren()) {
                    Expense expense = d.getValue(Expense.class);

                    if (expense != null) {
                        items.add(expense.description + " | ₹"
                                + expense.amount + " | "
                                + expense.category);
                        keys.add(d.getKey());
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(DatabaseError error) {
                Toast.makeText(
                        MainActivity.this,
                        error.getMessage(),
                        Toast.LENGTH_SHORT
                ).show();
            }
        });

        btnAdd.setOnClickListener(v -> {
            if (!validateFields()) return;

            String key = dbRef.push().getKey();

            Expense expense = new Expense(
                    etDescription.getText().toString().trim(),
                    etAmount.getText().toString().trim(),
                    etCategory.getText().toString().trim()
            );

            if (key != null) {
                dbRef.child(key).setValue(expense)
                        .addOnSuccessListener(unused -> {
                            clearFields();
                            Toast.makeText(this, "Expense added",
                                    Toast.LENGTH_SHORT).show();
                        })
                        .addOnFailureListener(e ->
                                Toast.makeText(this, e.getMessage(),
                                        Toast.LENGTH_LONG).show());
            }
        });

        btnUpdate.setOnClickListener(v -> {
            if (selectedKey == null) {
                showMessage("Select an expense first");
                return;
            }

            if (!validateFields()) return;

            Expense expense = new Expense(
                    etDescription.getText().toString().trim(),
                    etAmount.getText().toString().trim(),
                    etCategory.getText().toString().trim()
            );

            dbRef.child(selectedKey).setValue(expense)
                    .addOnSuccessListener(unused -> {
                        clearFields();
                        showMessage("Expense updated");
                    })
                    .addOnFailureListener(e -> showMessage(e.getMessage()));
        });

        btnDelete.setOnClickListener(v -> {
            if (selectedKey == null) {
                showMessage("Select an expense first");
                return;
            }

            dbRef.child(selectedKey).removeValue()
                    .addOnSuccessListener(unused -> {
                        clearFields();
                        showMessage("Expense deleted");
                    })
                    .addOnFailureListener(e -> showMessage(e.getMessage()));
        });

        listView.setOnItemClickListener((parent, view, position, id) -> {
            selectedKey = keys.get(position);

            dbRef.child(selectedKey).get()
                    .addOnSuccessListener(snapshot -> {
                        Expense expense = snapshot.getValue(Expense.class);

                        if (expense != null) {
                            etDescription.setText(expense.description);
                            etAmount.setText(expense.amount);
                            etCategory.setText(expense.category);
                        }
                    });
        });
    }

    private boolean validateFields() {
        if (etDescription.getText().toString().trim().isEmpty()) {
            etDescription.setError("Enter description");
            return false;
        }

        if (etAmount.getText().toString().trim().isEmpty()) {
            etAmount.setError("Enter amount");
            return false;
        }

        if (etCategory.getText().toString().trim().isEmpty()) {
            etCategory.setError("Enter category");
            return false;
        }

        try {
            if (Double.parseDouble(
                    etAmount.getText().toString().trim()) <= 0) {
                etAmount.setError("Enter an amount greater than zero");
                return false;
            }
        } catch (NumberFormatException e) {
            etAmount.setError("Enter a valid amount");
            return false;
        }

        return true;
    }

    private void clearFields() {
        etDescription.setText("");
        etAmount.setText("");
        etCategory.setText("");
        selectedKey = null;
        listView.clearChoices();
    }

    private void showMessage(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
}
```
## ACTIVITY_MAIN.XML:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Expense Tracker"
        android:textSize="26sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginBottom="16dp"/>

    <EditText
        android:id="@+id/etDescription"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Expense description"
        android:inputType="textCapSentences"/>

    <EditText
        android:id="@+id/etAmount"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Amount (₹)"
        android:inputType="numberDecimal"/>

    <EditText
        android:id="@+id/etCategory"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Category (Food, Travel...)"
        android:inputType="textCapSentences"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Add"/>

        <Button
            android:id="@+id/btnUpdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Update"/>

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Delete"/>
    </LinearLayout>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Saved Expenses"
        android:textSize="18sp"
        android:textStyle="bold"
        android:layout_marginTop="16dp"/>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>

</LinearLayout>
```


## OUTPUT

<img width="1920" height="1080" alt="Screenshot (7)" src="https://github.com/user-attachments/assets/5a14632b-dd10-4902-abf3-8a59b2960990" />


<img width="1920" height="1080" alt="Screenshot (6)" src="https://github.com/user-attachments/assets/8d505fe8-905f-49bc-8163-8847f4feefe4" />






## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.

