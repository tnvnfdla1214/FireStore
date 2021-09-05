# FireStore와 storage
파이어베이스는 2가지 클라우드 데이터베이스 솔루션인 RealtimeDataBase 와 FireStore 를 제공하고 있습니다. 

다만 이전에 나온것이 RealtimeDataBase이고 차후 FireStore 나오며 구글은 FireStore을 권장하고 있습니다.

Firestore는 구글에서 서비스 중인 NoSQL 데이터베이스입니다.

Firestore는 컬렉션(Collection)과 문서(Documents)가 있고, 문서내엔 키-값 쌍으로 이루어진 데이터가 있다. 각 문서는 하위 컬렉션을 가질 수도 있습니다.

또한 Cloud Stoage는 사진,동영상, 등의 사용자 제작 콘텐츠를 저장하고 제공하는 기능을 합니다.

***
 ### :wrench: 프로젝트 사용 사례
 
 <img src = "https://user-images.githubusercontent.com/48902047/132117888-0bd2687b-610c-45a1-addc-3dc23f82ee16.JPG">  <img src = "https://user-images.githubusercontent.com/48902047/132119246-665515ed-a96e-467c-b759-c9b108f32b22.JPG">
+ [조치원 수호대](https://github.com/tnvnfdla1214/homemade_guardian) 
  + 유저
  + 게시물
  + 커뮤니티
  + 채팅
  + 리뷰
  + 댓글


 + [Malang](https://github.com/tnvnfdla1214/Malang)
   + 유저
   + 캘린더
   + 커플

***

### :lollipop: 설명 (조치원 수호대 유저)

#### :라이브러리 추가

```Java
/*build.gradle*/

dependencies {
    .
    .
    
    implementation 'com.google.firebase:firebase-database:19.3.1'
    implementation 'com.google.firebase:firebase-firestore:21.5.0'
    implementation 'com.google.firebase:firebase-storage:19.1.1'
    // FirebaseUI for Cloud Storage
    implementation 'com.firebaseui:firebase-ui-storage:4.1.0'
    .
    .
}

```
#### UserModel 설계

먼저 저장할 유저의 정보를 담을 수 있는 model을 만든다.

알아야 할 것은 파이어스토어를 사용하기 위해서는 디폴트 생성자를 만들어줘야 한다.

또한 해당 정보를 파이어스토어와 연결을 시켜주기 위해 매핑하는 함수또한 있어야 한다.

```Java
/*UserModel.java*/

public class UserModel implements Serializable {
    private String UserModel_ID; 
    private String UserModel_Uid;
    private String UserModel_NickName;
    private String UserModel_BirthDay;
    private int UserModel_University;
    private String UserModel_ProfileImage;
    private Date UserModel_DateOfManufacture;
    private String UserModel_Token;//토큰값

    private ArrayList<HashMap<String, String>> UserModel_Unreview = new ArrayList<HashMap<String, String>>();

    private ArrayList<String> UserModel_UnReViewUserList;
    private ArrayList<String> UserModel_UnReViewPostList;

    private ArrayList<String> UserModel_kindReviewList;
    private ArrayList<String> UserModel_correctReviewList;
    private ArrayList<String> UserModel_completeReviewList;
    private ArrayList<String> UserModel_badReviewList;
    private ArrayList<String> UserModel_WritenReviewList;
    private ArrayList<String> UserModel_Market_reservationList;
    private ArrayList<String> UserModel_Market_dealList;

    public UserModel(){
    }

    public Map<String, Object> getUserInfo(){
        Map<String, Object> docData = new HashMap<>();
        docData.put("UserModel_ID", UserModel_ID);
        docData.put("UserModel_Uid", UserModel_Uid);
        docData.put("UserModel_NickName", UserModel_NickName);
        docData.put("UserModel_BirthDay", UserModel_BirthDay);
        docData.put("UserModel_University", UserModel_University);
        docData.put("UserModel_ProfileImage", UserModel_ProfileImage);
        docData.put("UserModel_DateOfManufacture", UserModel_DateOfManufacture);
        docData.put("UserModel_kindReviewList", UserModel_kindReviewList);
        docData.put("UserModel_correctReviewList", UserModel_correctReviewList);
        docData.put("UserModel_completeReviewList", UserModel_completeReviewList);
        docData.put("UserModel_badReviewList", UserModel_badReviewList);
        docData.put("UserModel_WritenReviewList", UserModel_WritenReviewList);
        docData.put("UserModel_Market_reservationList", UserModel_Market_reservationList);
        docData.put("UserModel_Market_dealList", UserModel_Market_dealList);
        docData.put("UserModel_Token", UserModel_Token);
        docData.put("UserModel_Unreview", UserModel_Unreview);
        return  docData;
    }
}

```
#### 정보를 저장할 액티비티 생성

해당 정보를 User 객체에 세팅 후 스토리지와 파이어 스토어에 저장한다.

스토리지는 "USERS/" + CurrentUser.getUid() + "/USERSImage.jpg" 와 같이 유저 / 해당 유저의 uid / 유저이미지 순으로 저장한다.

파이어스토어는 User 객체를 매팅해 놓았던 형식대로 파이어스토어에 저장한다.

파이어 스토어에는 ProfileImage 라는 String형의 자료가 있는데 이는 스토리지의 디렉토리 주소형식과 동일하다. 정보를 가져올때 해당 디렉토리 주소로 찾아올 수 있다.

```Java
/*MemberInitActivity.java*/

public class MemberInitActivity extends BasicActivity { 

    private FirebaseUser CurrentUser;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user_init);
        .
        .
        CurrentUser=FirebaseAuth.getInstance().getCurrentUser();
        MemberInit_Storage_Uploader();
        .
        .
    }

    //스토리지에 사진을 먼저 담는 함수
    private void MemberInit_Storage_Uploader() {

        // 입력받은 닉네임을 받아오고 20자가 넘으면 안됨
        final String UserModel_Nickname = Nickname_EditText.getText().toString();
        if (UserModel_Nickname.length() < 20) {

            // 등록이 시작되면 다른 이벤트를 방지하기 위해서 Dialog를 활성화한다.
            dialog.callDialog();

            // 파이어베이스 스토리지에 대한 Reference와 현재 유저의 정보를 받아온다.
            FirebaseStorage Firebasestorage = FirebaseStorage.getInstance();
            StorageReference Storagereference = Firebasestorage.getReference();
            CurrentUser = FirebaseAuth.getInstance().getCurrentUser();

            //스토리지의 USER/유저의 UID/이미지 들어가는곳  에다가 넣는다.
            final StorageReference ImageRef_USERS_Uid = Storagereference.child("USERS/" + CurrentUser.getUid() + "/USERSImage.jpg");
            .
            .
            FirebaseInstanceId.getInstance().getInstanceId().addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                @Override
                public void onComplete(@NonNull Task<InstanceIdResult> task) {
                    if (!task.isSuccessful()) {
                        return;
                    }
                        .
                        .
                        MemberInit_Store_Uploader(userModel);
                        .
                        .
                    }else{
                      .
                      .
                      MemberInit_Store_Uploader(userModel);
                    }
        }
    }

    //Usermodel에다 담은 회원정보를 파이어스토어 USERS/CurrentUser의 Uid에다가 넣는 함수
    private void MemberInit_Store_Uploader(final UserModel Usermodel) {
        FirebaseFirestore docSet_USERS_Uid = FirebaseFirestore.getInstance();
        docSet_USERS_Uid.collection("USERS").document(CurrentUser.getUid()).set(Usermodel.getUserInfo())
                .addOnSuccessListener(new OnSuccessListener<Void>() {
                    @Override
                    public void onSuccess(Void aVoid) {
                      .
                      .
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                      .
                      .
                    }
                });
    }
}

```
<img src = "https://user-images.githubusercontent.com/48902047/132119863-23834ff9-b1df-489a-bef9-a1972301c050.JPG">

#### 정보를 가져올 액티비티 생성

저는 프레그먼트에서 직접 정보를 가져오기 때문에 프레그먼트 코드가 있습니다.

위와 같이 가져올 User또한 해당 Uid로 USER 디렉토리에서 정보를 찾아 FireStore 에서 모든 자료와 사진의 디렉토리 주소 정보를 가져옵나다.

정보를 가져올때도 User에 저장해놓은 규칙에 따라 정보를 매핑에 정보를 가져옵니다

이미지 또한 ProfileImage의 주소 String으로 정보를 가져 옵니다.

```Java
/*MyInfo_BottombarFragment.java*/

public class MyInfo_BottombarFragment extends Fragment {

    private String CurrentUid;
    private FirebaseUser CurrentUser;
    UserModel userModel = new UserModel();


    @Nullable
    @Override
    public View onCreateView(@Nullable LayoutInflater inflater,@Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_myinfo, container, false);
        
        CurrentUser = FirebaseAuth.getInstance().getCurrentUser();
        CurrentUid =CurrentUser.getUid();
        .
        .
        getUserModel(CurrentUid);
        .
        .
        return view;
    }

    public void getUserModel(String CurrentUid){
        final DocumentReference documentReference = FirebaseFirestore.getInstance().collection("USERS").document(CurrentUid);
        documentReference.get().addOnCompleteListener(new OnCompleteListener<DocumentSnapshot>() {
            @Override
            public void onComplete(@NonNull Task<DocumentSnapshot> task) {
                if (task.isSuccessful()) {
                    DocumentSnapshot document = task.getResult();
                    if (document != null) {
                        if (document.exists()) {  //데이터의 존재여부
                            userModel = document.toObject(UserModel.class);
                            Profile_Info(userModel);
                        }
                    }
                }
            }
        });
        return;
    }

    public void Profile_Info(UserModel Usermodel){
            .
            .
            Glide.with(this).load(Usermodel.getUserModel_ProfileImage()).centerCrop().into(Myinfo_profileImage);
            .
            .
    }
}

```
