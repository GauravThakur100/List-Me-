# ListMe

ListMe is an Android application that allows users to create, view, and manage their to-do lists. The app leverages Firebase for authentication and real-time database operations, ensuring a seamless user experience.

## Features

- **User Authentication**: Secure sign-in using email authentication via Firebase Authentication.
- **To-Do List Management**: Add, view, and manage to-do items.
- **Real-time Sync**: Data is synced in real-time with Firebase Realtime Database.
- **Floating Action Button**: Easily add new to-do items with a floating action button.

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/gauravthakur/listme.git
    ```

2. Open the project in Android Studio.

3. Make sure you have configured Firebase for your project and added the `google-services.json` file to your project.

4. Build and run the project on an Android device or emulator.

## Usage

1. Open the ListMe app.
2. Sign in using your email.
3. Use the floating action button to add new to-do items.
4. Tap on any to-do item to view or edit details.
5. Use the options menu to sign out.

## Code Overview

The main functionality of the app is implemented in the `MainActivity` class.

### MainActivity.java

```java
public class MainActivity extends AppCompatActivity implements ToDoListAdapter.TodoListAdapterOnClickHandler {
    // Fields for RecyclerView, Adapter, FirebaseAuth, and others

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize RecyclerView, FirebaseAuth, and set up listeners
        // ...

        // Authentication state listener
        mAuthStateListener = new FirebaseAuth.AuthStateListener() {
            @Override
            public void onAuthStateChanged(FirebaseAuth firebaseAuth) {
                FirebaseUser user = firebaseAuth.getCurrentUser();
                if (user != null) {
                    mUserId = user.getUid();
                } else {
                    Intent signInIntent = AuthUI.getInstance()
                            .createSignInIntentBuilder()
                            .setIsSmartLockEnabled(false)
                            .setTheme(R.style.Theme_ToDoNotify)
                            .setAvailableProviders(Collections.singletonList(new AuthUI.IdpConfig.EmailBuilder().build()))
                            .build();
                    signInLauncher.launch(signInIntent);
                }
            }
        };

        // FloatingActionButton for adding new to-do items
        FloatingActionButton fab = findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(MainActivity.this, DetailActivity.class);
                intent.putExtra("uid", mUserId);
                startActivity(intent);
            }
        });
    }

    @Override
    public void onClick(int position) {
        Intent detailIntent = new Intent(this, DetailActivity.class);
        detailIntent.putExtra("uid", mUserId);
        DatabaseReference ref = mAdapter.getRef(position);
        String id = ref.getKey();
        detailIntent.putExtra("ref", id);
        startActivity(detailIntent);
    }

    @Override
    public void onResume() {
        super.onResume();
        mFirebaseAuth.addAuthStateListener(mAuthStateListener);
    }

    @Override
    protected void onPause() {
        super.onPause();
        mFirebaseAuth.removeAuthStateListener(mAuthStateListener);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.options_menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.sign_out) {
            AuthUI.getInstance().signOut(this);
            return true;
        } else {
            return super.onOptionsItemSelected(item);
        }
    }

    private void setAdapter() {
        FirebaseDatabase firebaseDatabase = FirebaseDatabase.getInstance();
        Query query = firebaseDatabase.getReference().child("to_do").orderByChild("uid").equalTo(mUserId);

        FirebaseRecyclerOptions<ToDo> options =
                new FirebaseRecyclerOptions.Builder<ToDo>()
                        .setQuery(query, ToDo.class)
                        .build();
        mAdapter = new ToDoListAdapter(options, this);
        mRecyclerView.setAdapter(mAdapter);
        mAdapter.startListening();
    }
}
```

### ToDoListAdapter.java

The `ToDoListAdapter` class handles binding the to-do items to the RecyclerView.

### DetailActivity.java

The `DetailActivity` class handles viewing and editing details of a selected to-do item.

## Contributing

1. Fork the repository.
2. Create a new branch: `git checkout -b feature-name`
3. Make your changes and commit them: `git commit -m 'Add new feature'`
4. Push to the branch: `git push origin feature-name`
5. Submit a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgements

- Thanks to all the contributors and users for their support and feedback.
- Special thanks to the open-source community for providing the resources and inspiration to build this app.
