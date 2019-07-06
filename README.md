# 공부하면서 헷갈리는거 메모

```xml
<TextView
        android:text="sign in with Google"
        android:id="@+id/imageView6"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView"
        android:background="@drawable/btn_google_signin_light_normal_xxhdpi" />
```

## bottomNav 만들기
> 1. 각 연결할 fragment 를 만든다.(3개의 bottom tab 이므로 3개 만듦)
> 2. fragment_tab 을 만들어서 constraint 에다가 bottomNaviagation 을 둔다
> 3. bottom_nav_menu 를 만들어서 menu_icon 을 만든다(title, icon도 넣고)
> 4. bottomNaviagation 을 둔 곳에 
```xml
        app:menu="@menu/bottom_nav_menu" />

```
추가시킨다.

5.bottom_nav_menu 
nav_graph 의 각 fragment id 와 bottom_nav_menu 의 id 를 각각 연결시킨다.

## nav_host_fragment & BottomNavigationView 연결
> BottomNavFragment 만들기
1. **nav_host_fragment** 와 **BottomNavigationView** 가 있어야 한다
2. nav_host_fragment 를 박아두기 위해서는 옵션으로 
```xml
app:navGraph="@navigation/nav_graph" />
```
가 있어야 한다. ( pallete 에서 NavHostFragment 검색해서 긁어온다. )

> **BottomNavigationView**
> 1. menu를 추가 시켜준다.
```xml
        app:menu="@menu/menu_bottom_nav"
```

## BottomNavFragment 에서 해야할 것
> nav_host_fragment(= nav_graph) 와 BottomNavigation 을 navController 를 통해서 연결해준다.

## 여기서 중요한 점
> nav_host_fragment(= nav_graph)

### 처음 화면 이동 ( 로그인 인증 안되었을시 -> 로그인 화면으로 가기 )
MainActivity.kt

```kotlin
// 처음 화면 이동 ( 로그인 인증 안되었을시 -> 로그인 화면으로 가기 )
        if (FirebaseAuth.getInstance().currentUser == null) {
            startActivity(Intent(this, LoginActivity::class.java))
            finish()
        }
```

### 여기서 GoogleSignInClient 가 빨간 불 나는데 객체를 얻어와야 함 https://developers.google.com/identity/sign-in/android/sign-in
```kotlin
class LoginActivity : AppCompatActivity() {

    private lateinit var auth: FirebaseAuth
    private lateinit var googleSignInClient: GoogleSignInClient

    companion object {
        const val RC_SIGN_IN = 1000
        const val TAG = "LoginActivity"
    }
// ...
// Initialize Firebase Auth

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        // Configure Google Sign In
        val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
            .requestIdToken(getString(R.string.default_web_client_id))
            .requestEmail()
            .build()
        auth = FirebaseAuth.getInstance()

        googleSignInClient = GoogleSignIn.getClient(this, gso)

        button_sign_in.setOnClickListener {
            signIn()
        }

    }
}
```

