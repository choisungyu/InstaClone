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

bottomNav 만들기
1. 각 연결할 fragment 를 만든다.(3개의 bottom tab 이므로 3개 만듦)
2. fragment_tab 을 만들어서 constraint 에다가 bottomNaviagation 을 둔다
3. bottom_nav_menu 를 만들어서 menu_icon 을 만든다(title, icon도 넣고)
4. bottomNaviagation 을 둔 곳에 
```xml
        app:menu="@menu/bottom_nav_menu" />

```
추가시킨다.

5.bottom_nav_menu 
nav_graph 의 각 fragment id 와 bottom_nav_menu 의 id 를 각각 연결시킨다.
