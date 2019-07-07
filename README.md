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
### 암시적 인텐트로 signIn 결과값을 가져올 때 startActivityForResult 를 써준다
```kotlin
private fun signIn() {
        // 암시적 인텐트로 signIn 결과값을 가져올 때 startActivityForResult
        val signInIntent = googleSignInClient.signInIntent
        startActivityForResult(signInIntent, RC_SIGN_IN)
}
```
### AccountFragment
#### AccountFragment.kt 까지 googleSignInClient 객체를 가져와야 함 -> 
```kotlin
val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
            .requestIdToken(getString(R.string.default_web_client_id))
            .requestEmail()
            .build()

        googleSignInClient = GoogleSignIn.getClient(requireContext(), gso)

```
#### -> googleSignInClient.signOut() 도 같이 해줘야 하기 때문 ( FirebaseAuth.getInstance().signOut() 둘 다 해줘야 함 )
```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.action_sign_out) {
            FirebaseAuth.getInstance().signOut()
            googleSignInClient.signOut()

            // 로그인 화면으로 이동
            startActivity(Intent(requireContext(), LoginActivity::class.java))
            requireActivity().finish()
        }
        return super.onOptionsItemSelected(item)
    }
}
```   
#### Fragment binding 하는 모습
```kotlin
class HomeFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        val view = inflater.inflate(R.layout.fragment_home, container, false)

        val binding = FragmentHomeBinding.bind(view)

        // FirebaseAuth.getInstance().currentUser 가 null 이 아니면 user 객체 를 binding.user 에 넣어라
        FirebaseAuth.getInstance().currentUser?.let {user->
            binding.user = user
        }
        return binding.root
    }


}
```
#### SquareImageView ( Java.class )
```kotlin   
public class SquareImageView extends AppCompatImageView {
    public SquareImageView(Context context) {
        super(context);
    }

    public SquareImageView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public SquareImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, widthMeasureSpec);
    }
}
```
#### (options: FirestoreRecyclerOptions<Post>) = PostRecyclerAdapter 의 생성자
```kotlin   

class PostRecyclerAdapter(options: FirestoreRecyclerOptions<Post>) :
        FirestoreRecyclerAdapter<Post, PostRecyclerAdapter.PostViewHolder>(options) {
        class PostViewHolder(val binding: ItemPostBinding) : RecyclerView.ViewHolder(binding.root)

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_post, parent, false)

            return PostViewHolder(ItemPostBinding.bind(view))
        }

        override fun onBindViewHolder(holder: PostViewHolder, postion: Int, model: Post) {
            holder.binding.post = model
        }
    }
```

## Firebase 용 RecyclerAdapter 참고하는곳
https://github.com/firebase/FirebaseUI-Android/tree/master/firestore

## MainActivity 에 BackButton 추가시켜야 하는 것
```kotlin
navController = findNavController(R.id.nav_host_fragment)
        setupActionBarWithNavController(navController)
```

```xml
<androidx.appcompat.widget.Toolbar
        android:background="@android:color/transparent"
/>
```

### Common Intents ( : 그림 가져오는 것 )
https://developer.android.com/guide/components/intents-common#Storage

```text
imageUri.value = intent.data 
// imageUri.setValue(intent.getData()) ==> intent해서 가져온 getData 값을 imageUri 에 MutableLiveData 의 메소드 setValue 를 호출해서 set시킴     
```

### Firebase ImageUpload 하는 방법 (File 말고 Uri 를 가지고 업로드 시킴 )
#### 1. 아까 imageUri.getValue 값을 stream 형태로 넘기고 -> 그것을 다시 backgroundThread 에서 돌리기 위해 lifecycleScope.launch(Dispatchers.IO) 사용

- contentResolver : uri -> stream 형태로 변환

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.action_send) {
            // 이미지 업로드 & DB 에 작성해야 함

            imageUri.value?.let { uri ->
                val stream = requireActivity().contentResolver.openInputStream(uri)

                stream?.let {

                    // 백그라운드 처리 ( imageUri )
                    lifecycleScope.launch(Dispatchers.IO) {

                        // viewModel로 보냄
                        val downloadUri = viewModel.uploadImage(stream)
                        Log.d("log", "$downloadUri")

                        launch(Dispatchers.Main) {
                            // Ui 갱신
                        }
                    }
                }
            }
            // 업로드 결과 db에 작성
            return true
        }
        return super.onOptionsItemSelected(item)
    }
```

#### viewModel 비동기 한줄로 쓰는거 : Tasks.await().continueWithTask() {task ->}
- firebase 를 콜백인 것을 콜백이 아닌것 처럼 바꾸는 것 (순차적) 
- Tasks.await() 을 써야 함 ( 비동기인데 , 비동기 아닌 것 처럼 해주는 것 )
- 하지만 Tasks.await() 는 자바에서 못 쓰는 것 ==> coroutine 이랑 같이 써야 하는 것

#### fun uploadImage 를 viewModel 로 빼는 이유?

```kotlin
class CreatePostViewModel : ViewModel() {

    // 이 처리(fun uploadImage)를 viewModel 로 빼는 이유?
    fun uploadImage(stream: InputStream): Uri {

        // CreatePostViewModel 에 liveData 객체 가져오는지?

        val ref =
            FirebaseStorage.getInstance().reference.child("images/${System.currentTimeMillis()}.jpg")

        // ref.putStream(stream)
        return Tasks.await(ref.putStream(stream).continueWithTask(Continuation<UploadTask.TaskSnapshot, Task<Uri>> { task ->
            if (!task.isSuccessful) {
                task.exception?.let {
                    throw it
                }
            }
            return@Continuation ref.downloadUrl
        }))


    }

}
```
### CoroutineScope 대신에 AC 랑 같이 쓰는 ViewModelScope 이나 LifeCycleScope 를 쓰자

- uploadImage 가 끝나는 시점에서 UI 쪽에서 

### DB 에 넣는 거 => add data 
https://firebase.google.com/docs/firestore/manage-data/add-data


### progressbar 넣기 해야함...
https://www.youtube.com/watch?v=2xkb0qgZXS8&list=PLxTmPHxRH3VUHBs-zl8qHbkdAcCwsGtyn&index=131
